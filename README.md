# Minotask Web App

Minotask is a Slovenian SaaS that turns PDF invoices into compliant Intrastat reports. It consists of a React/Tailwind frontend and a Flask backend that runs the AI-powered extraction, enrichment, and Excel generation pipeline. Authentication and user storage run through Supabase; job progress is streamed to the browser via Server-Sent Events (SSE).

## Monorepo Layout

- `frontend/` - React 18 + Vite app (shadcn/ui, Tailwind, React Query). Handles auth, PDF upload, job tracking, downloads, bug reporting, pricing/profile pages, and Stripe UI scaffolding.
- `backend/` - Flask API. Validates Supabase JWTs, accepts PDF uploads, runs the Intrastat engine (Gemini AI + tariff/weight enrichment), emits SSE progress, serves Excel output, and receives bug reports.

## How Frontend & Backend Talk

- Auth: frontend uses Supabase client; access token is sent as `Authorization: Bearer <jwt>` to backend.
- Jobs: `POST /api/jobs` uploads PDFs + report_type; backend queues processing and returns `jobId`. Frontend then:
  - listens to `GET /api/jobs/{id}/events?token=jwt` via SSE for live stage updates;
  - falls back to 4s polling `GET /api/jobs/{id}` if SSE disconnects;
  - downloads result via `GET /api/jobs/{id}/download` when status is `completed`.
- Test/health: `GET /api/test` requires auth; `GET /health` is public.
- Bug reports: `POST /api/bug-report` (auth) with optional screenshot.


## High-Level Data Flow

1. User signs in via Supabase (email/password). Supabase session persists in browser.
2. User uploads one or more PDF invoices and selects report type (1=imports PREJEMI, 2=exports ODPREME).
3. Backend validates JWT, rate limits, validates PDFs, computes an input hash for idempotency, and enqueues work.
4. Intrastat engine (Gemini + tariff DB + OpenPyXL) extracts, enriches, and builds Excel in memory.
5. Progress events stream over SSE; on completion frontend triggers download.
6. Optional bug report flow sends text + screenshot to backend, which relays via Resend.
