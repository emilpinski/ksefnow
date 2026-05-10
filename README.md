# KSeF Now

![Status](https://img.shields.io/badge/Status-Live-green)

> KSeF automation SaaS for Polish accounting firms — AI invoice analysis, Ministry of Finance API sync, Stripe billing.

## What is it

KSeF Now is a SaaS platform for accounting firms and bookkeepers that automates handling of mandatory KSeF (required from 2026). The application uses AI to analyze invoices, detect anomalies, and validate documents before submission to the Ministry of Finance system — with bidirectional sync to the National e-Invoice System API.

## Features

- **AI categorization** — automatic recognition of cost type, VAT rate, and accounting code via Anthropic Claude Sonnet via OpenRouter
- **KSeF Sync** — bidirectional synchronization with the Ministry of Finance API, queues, retries, event log
- **Anomaly alerts** — duplicate detection, unusual amounts, contractors outside the VAT whitelist
- **KSeF Sandbox** — test integration in the MF sandbox environment without production impact
- **Analytics dashboard** — invoice history, submission statuses, monthly charts
- **Multi-client management** — manage multiple accounting clients from a single panel (Supabase RLS)
- **Stripe billing** — SaaS subscriptions with 7-day trial and webhook handling
- **DPA compliance** — data processing agreement template for GDPR compliance
- **Playwright scraping** — automated invoice retrieval from external accounting systems

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js, React, TypeScript, Tailwind CSS |
| Backend | Next.js API Routes, Zod validation |
| AI | Anthropic Claude Sonnet via OpenRouter |
| Database | Supabase (PostgreSQL + Auth + RLS) |
| Payments | Stripe (subscriptions, webhooks) |
| Email | Nodemailer |
| Charts | Recharts |
| Animations | Framer Motion |
| Deploy | Vercel |

## Compliance and data

- **Data residency**: EU only (Supabase Frankfurt region)
- **GDPR**: compliant; data processing agreement available on request
- **Subprocessors**: Supabase (database, auth), Stripe (billing), Anthropic via OpenRouter (AI categorization inference only, no invoice data retained by model provider), Vercel (hosting)
- **Audit trail**: every invoice state change is recorded as an immutable event with actor ID and timestamp
- **Encryption**: all data encrypted at rest (AES-256) and in transit (TLS 1.3)

## Market context

KSeF (Krajowy System e-Faktur) becomes mandatory for all Polish VAT-registered businesses in February 2026, affecting approximately 2 million companies. Non-compliance carries financial penalties. KSeFNow provides accountants and accounting firms with a production-ready integration that handles the full invoice lifecycle: XML generation, MF API submission, status tracking, and audit logging.

Comparable enterprise solutions (SAP, Comarch) require months of integration work. KSeFNow is API-first and deployable in days.

## Status

Live — [ksefnow.pl](https://ksefnow.pl)

---
Built by [Emil Piński](https://emilpinski.pl)

> Source code is private. [Contact for collaboration](mailto:emilpinskidev@gmail.com)

## Screenshots

![Landing page](docs/screenshots/landing_page_ksefnow.jpg)
![Dashboard overview](docs/screenshots/Zrzut_ekranu_25-4-2026_121035_ksefnow.pl.jpeg)
![Invoice list](docs/screenshots/Zrzut_ekranu_25-4-2026_121119_ksefnow.pl.jpeg)
![KSeF sync panel](docs/screenshots/Zrzut_ekranu_25-4-2026_121140_ksefnow.pl.jpeg)
![Anomaly alerts](docs/screenshots/Zrzut_ekranu_25-4-2026_121150_ksefnow.pl.jpeg)
![Analytics dashboard](docs/screenshots/Zrzut_ekranu_25-4-2026_121218_ksefnow.pl.jpeg)
![Billing and subscription](docs/screenshots/Zrzut_ekranu_25-4-2026_121933_ksefnow.pl.jpeg)
