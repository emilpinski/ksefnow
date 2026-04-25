# KSeF Now

> Asystent AI dla biur rachunkowych do obsługi Krajowego Systemu e-Faktur.

![Screenshot](./screenshot.png)

## Co to jest

KSeF Now to SaaS dla biur rachunkowych i księgowych, który automatyzuje obsługę obowiązkowego KSeF (od 2026 roku). Aplikacja analizuje faktury AI, wykrywa anomalie, waliduje dokumenty przed wysłaniem do systemu MF i synchronizuje dane dwukierunkowo z API Krajowego Systemu e-Faktur.

Skierowana do biur rachunkowych obsługujących wielu klientów, które chcą zminimalizować ręczną weryfikację faktur i uniknąć kar za błędne przesyłanie.

## Funkcje

- **AI kategoryzacja** — automatyczne rozpoznawanie typu kosztu, stawki VAT i konta księgowego
- **KSeF Sync** — dwukierunkowa synchronizacja z API Ministerstwa Finansów, kolejki, ponowienia, dziennik zdarzeń
- **Alerty anomalii** — wykrywanie duplikatów, nietypowych kwot, kontrahentów spoza białej listy
- **Sandbox KSeF** — testowanie integracji w środowisku MF bez wpływu na produkcję
- **Dashboard analityczny** — wykresy Recharts, historia faktur, statusy przesyłania
- **Wielodostęp dla biur** — zarządzanie wieloma klientami z jednego panelu (Supabase RLS)
- **Stripe billing** — subskrypcje SaaS z 7-dniowym trial, webhooks
- **DPA compliance** — gotowy szablon umowy powierzenia danych (RODO/GDPR)
- **Playwright scraping** — automatyczne pobieranie faktur z systemów zewnętrznych

## Stack

| Warstwa | Technologia |
|---------|-------------|
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS v4 |
| Backend | Next.js API Routes, Zod validation |
| AI | Anthropic Claude (claude-sonnet) via SDK |
| Baza danych | Supabase (PostgreSQL + Auth + RLS) |
| Płatności | Stripe (subscriptions, webhooks) |
| Email | Nodemailer |
| Charts | Recharts |
| Animacje | Framer Motion |
| Deploy | Vercel |

## Uruchomienie

```bash
git clone https://github.com/emilpinski/ksefnow
cd ksefnow
npm install
cp .env.example .env.local
# Uzupelnij zmienne srodowiskowe
npm run dev
```

## Zmienne środowiskowe

| Zmienna | Opis | Wymagana |
|---------|------|----------|
| `NEXT_PUBLIC_SUPABASE_URL` | URL projektu Supabase | ✅ |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Klucz publiczny Supabase | ✅ |
| `SUPABASE_SERVICE_ROLE_KEY` | Klucz serwisowy (backend) | ✅ |
| `ANTHROPIC_API_KEY` | Klucz API Claude AI | ✅ |
| `STRIPE_SECRET_KEY` | Klucz Stripe (backend) | ✅ |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Klucz Stripe (frontend) | ✅ |
| `STRIPE_WEBHOOK_SECRET` | Secret do weryfikacji webhooków | ✅ |
| `SMTP_HOST` | Serwer SMTP dla emaili | ✅ |
| `SMTP_USER` | Login SMTP | ✅ |
| `SMTP_PASS` | Haslo SMTP | ✅ |
| `KSEF_SANDBOX_URL` | URL sandbox API MF | ✅ |

## Status

WIP — staging, wdrożenie 2026

---
Built by [Emil Piński](https://emilpinski.pl)
