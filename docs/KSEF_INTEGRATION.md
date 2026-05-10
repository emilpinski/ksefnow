# KSeF Integration

This document describes how KSeFNow integrates with the Polish National e-Invoice System (Krajowy System e-Faktur, KSeF) operated by the Ministry of Finance. KSeF becomes mandatory for Polish businesses in February 2026.

## API Endpoints and Environments

| Environment | Base URL |
|-------------|----------|
| Production | `https://rest.ksef.mf.gov.pl` |
| Sandbox (test) | `https://ksef-test.mf.gov.pl/api` |
| Demo | `https://ksef-demo.mf.gov.pl/api` |

Environment selection is controlled by a single configuration flag (`KSEF_ENV=prod|sandbox|demo`). All HTTP clients, signature endpoints, and credential stores are resolved from this flag so that no environment-specific URL is hardcoded outside the configuration layer.

## XML Schema

KSeFNow generates and validates invoices against the FA(2) schema (Faktura Anonimowa, version 2) published by the Ministry of Finance. The XSD is fetched from the official MF schema repository and pinned by version in the project.

Required top-level elements:

- `Naglowek` - header with schema version, system code, and issue timestamp.
- `Podmiot1` - seller (issuer) identification including NIP and address.
- `Podmiot2` - buyer identification.
- `Fa` - invoice body: line items, tax rates, totals, payment terms.
- `Stopka` - optional footer with software identification.

Outgoing XML is validated locally against the XSD before submission. Validation errors are surfaced to the user with the violating XPath and field name rather than the raw XSD message.

## Authentication Flow

KSeF uses a session-based authentication model with short-lived JWT tokens.

1. Session initialization. The client sends an `InitSessionSigned` request containing a challenge issued by KSeF and a digital signature of that challenge. The signature must be produced with either a qualified electronic signature (XAdES) or a qualified electronic seal (Pieczec Elektroniczna).
2. KSeF responds with a session JWT (`sessionToken`) and a reference number.
3. All subsequent API calls (send invoice, query status, fetch UPO) include the session token in the `SessionToken` header.
4. The session token has a limited lifetime. KSeFNow tracks token issue time and refreshes proactively before expiry.

```
client                          KSeF
  | --- GetChallenge ----------->|
  |<--- challenge + timestamp ---|
  | --- sign(challenge) -------- |  (local XAdES/seal)
  | --- InitSessionSigned ------>|
  |<--- sessionToken ------------|
  | --- SendInvoice (token) ---->|
  |<--- referenceNumber ---------|
```

Credential storage:

- Signing certificates (and their private keys) are not stored in the application database. KSeFNow integrates with an external signing service (HSM-backed) for production and supports software-key signing only in sandbox/demo for development convenience.

## Submission Flow

The full lifecycle of a single invoice:

1. Create. The invoice is composed from internal data and persisted in `DRAFT` state.
2. Generate XML. FA(2) XML is rendered from the internal model.
3. Validate. XML is validated against the pinned FA(2) XSD. Failures keep the invoice in `DRAFT` and return field-level errors.
4. Sign. The XML is sent to the signing service, which returns a XAdES-enveloped signature or an attached seal.
5. Submit. The signed payload is sent to KSeF. The state moves to `SENT`. KSeF returns an `elementReferenceNumber`.
6. Poll status. KSeFNow polls the status endpoint until KSeF reports a terminal state (`ACCEPTED` or `REJECTED`). Intermediate states such as `PROCESSING` are recorded but not exposed as user-actionable.
7. On acceptance. KSeF returns the KSeF number (32 characters) and the acceptance timestamp. Both are persisted on the invoice record. The state moves to `ACCEPTED`.
8. Fetch UPO. The Official Confirmation of Receipt (UPO, Urzedowe Poswiadczenie Odbioru) is downloaded and stored as an immutable artifact.

## State Machine

```
DRAFT -> SENT -> PROCESSING -> ACCEPTED
                            \-> REJECTED
```

Transitions are persisted as append-only events (see Event Sourcing below). The current state is a projection of the event log.

## KSeF Reference Number

After acceptance KSeF assigns a 32-character KSeF number that is the canonical identifier of the invoice in the national system. KSeFNow:

- Stores it as a unique, indexed column on the invoice record.
- Treats it as immutable. Any subsequent change requires a corrective invoice (faktura korygujaca), never an in-place edit.
- Exposes it on PDF and email renderings alongside the buyer-facing invoice number.

## Retry Strategy

The MF API can be temporarily unavailable, especially around peak filing windows. KSeFNow uses exponential backoff with jitter for transient failures:

| Attempt | Delay |
|---------|-------|
| 1 | immediate |
| 2 | 2s + jitter |
| 3 | 8s + jitter |
| 4 | 30s + jitter |
| 5 | 120s + jitter |
| 6+ | 600s + jitter, capped, until max retry window (24h) |

Retries are triggered only for safe conditions: connection errors, HTTP 5xx, HTTP 429, and KSeF status codes that explicitly indicate a transient processing problem. HTTP 4xx (other than 429) is treated as a permanent failure and surfaces to the user as a rejection with the reported reason.

## Idempotency

Duplicate submission is prevented at two layers:

1. Application layer. Each invoice carries an internal idempotency key (UUID v4) generated at creation. The `submit` operation is keyed on this UUID; a repeated call with the same key short-circuits and returns the previous result instead of re-sending.
2. KSeF layer. The `elementReferenceNumber` returned by KSeF on the first successful submission is stored. Any subsequent submission attempt for the same invoice checks for an existing reference number before issuing a new request.

The combination guarantees that even a network failure mid-submission cannot cause a double registration in KSeF.

## Sandbox vs Production Switching

- A single environment variable (`KSEF_ENV`) selects the API base URL, the signing endpoint, and the certificate keystore.
- Production credentials are never available in non-production environments. The deployment pipeline refuses to inject prod credentials into sandbox builds.
- Sandbox invoices are marked with a clearly visible badge in the UI to avoid confusion with real fiscal documents.

## Error Handling

KSeF rejections come with a structured error code and human-readable message. KSeFNow maps the most common codes to actionable remediation:

| Code (example) | Meaning | Remediation |
|----------------|---------|-------------|
| 21100 | Invalid NIP | Verify buyer NIP and reissue |
| 21210 | Missing required field | Field name surfaced; edit invoice and resubmit |
| 21270 | Invalid date | Adjust issue or sale date within allowed range |
| 21310 | Signature invalid | Re-sign and resubmit; check certificate validity |
| 21500 | Schema validation failed | Review XSD error path; fix corresponding field |

Unknown codes are stored verbatim and shown to the user with a link to the MF error code reference. They never cause silent retry.

## Event Sourcing and Audit Log

Every state change of every invoice is appended to an immutable event log (`invoice_events` table). Each event records:

- `invoice_id`
- `event_type` (CREATED, VALIDATED, SIGNED, SUBMITTED, ACK_RECEIVED, ACCEPTED, REJECTED, UPO_FETCHED, CORRECTED)
- `payload` (the relevant request/response fragment, redacted of secrets)
- `actor` (user id or system process)
- `timestamp`
- `correlation_id` (links retries and follow-ups)

The current invoice state is always a deterministic projection of this log. This serves both audit requirements (KSeF and Polish tax law require traceable retention) and operational debugging.

## Implementation Status

The integration is under active development. Current status by area:

| Area | Status |
|------|--------|
| Sandbox session init and submission | Working |
| FA(2) XSD validation | Working |
| Status polling and UPO download | Working |
| Production signing via HSM | [PLANNED] |
| Corrective invoices (faktura korygujaca) | [IN PROGRESS] |
| Bulk submission (batch mode) | [PLANNED] |
| Webhook-driven status updates | [PLANNED] (currently polling only) |
| Self-billing (samofakturowanie) | [PLANNED] |

This document will be updated as features move from planned to working.
