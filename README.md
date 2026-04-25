# KSeF Now — Tax Automation SaaS

> Automate KSeF (National e-Invoice System) workflows for Polish accounting offices.

**Live:** [ksefnow.pl](https://ksefnow.pl) &nbsp;|&nbsp; **Status:** Live

---

## What it does

- Invoice ingestion directly from the Ministry of Finance KSeF API
- AI-powered cost classification into accounting codes with VAT rate detection
- Anomaly detection: duplicate amounts, off-whitelist vendors, atypical volumes
- Multi-tenant dashboard for accounting firms managing multiple client accounts
- JPK_V7, JPK_FA, JPK_KR XML export ready for MF submission
- Stripe subscription billing with tiered plans (199–999 PLN/month)

## How it works

- **KSeF API integration**: Bidirectional sync with Ministerstwo Finansów for real-time invoice ingestion, status polling, and correction workflows — handles queue retries, signature validation, audit logs
- **AI classification**: Claude API auto-categorizes invoices into accounting codes and routes to correct ledger accounts
- **Multi-tenant security**: Role-based access control with Supabase row-level security separating client data
- **Compliance exports**: Generates digitally-signed JPK XML reports ready for MF submission

## Tech Stack

![Next.js](https://img.shields.io/badge/Next.js_15-black?logo=next.js) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white) ![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?logo=supabase&logoColor=white) ![Stripe](https://img.shields.io/badge/Stripe-635BFF?logo=stripe&logoColor=white) ![Claude](https://img.shields.io/badge/Anthropic_Claude-D4A27F)
