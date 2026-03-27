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

## Environment Variables

### Frontend (`frontend/.env.local`)

- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`
- `VITE_BACKEND_URL` - base URL of this Flask API.
- Optional Stripe placeholders already scaffolded: `VITE_STRIPE_ENABLED`, `VITE_API_URL` (for future billing endpoints).

### Backend (`backend/.env`)

- `SUPABASE_PROJECT_ID` - used to fetch JWKS and build issuer/audience.
- `JWT_SECRET` - Supabase legacy JWT secret (HS256) for verification fallback.
- `GEMINI_API_KEY` - Google Gemini for invoice extraction/classification.
- `OPENAI_API_KEY` - optional.
- `RESEND_API_KEY` - for bug report emails.
- `RATE_LIMIT_*` (optional), `MAX_CONTENT_LENGTH`, `MAX_UPLOAD_SIZE`.
- `FLASK_DEBUG`, `HOST`, `PORT` (defaults 0.0.0.0:5000).

## Running Locally

1. Clone with submodules:

```bash
git clone --recurse-submodules <repo-url>
cd minotask_webapp
```

2. Backend

```bash
cd backend
python -m venv .venv
. .venv/Scripts/activate  # or source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # then fill values
python app.py
```

API listens on `http://localhost:5000`.

3. Frontend

```bash
cd ../frontend
npm install
npm run dev
```

Set `VITE_BACKEND_URL=http://localhost:5000` and Supabase values. Vite serves on `http://localhost:5173` (or Replit port binding if used).

## Deployment Notes

- CORS allows `FRONTEND_URL_DEV` and `FRONTEND_URL_PROD` (configured in `backend/config.py`; defaults to Replit preview and https://minotask.si).
- Backend processes PDFs entirely in memory; ensure container memory limits are sufficient.
- SSE endpoints require the JWT passed as a `token` query param because `EventSource` cannot set headers.

## Stripe Integration Status

The frontend includes Stripe scaffolding (`src/stripe/*` and pricing/profile pages) with calls such as `createCheckoutSession`, `createCustomerPortalSession`, and credit tracking hooks. Backend endpoints are **not yet implemented**; see suggestion below to complete billing.

## High-Level Data Flow

1. User signs in via Supabase (email/password). Supabase session persists in browser.
2. User uploads one or more PDF invoices and selects report type (1=imports PREJEMI, 2=exports ODPREME).
3. Backend validates JWT, rate limits, validates PDFs, computes an input hash for idempotency, and enqueues work.
4. Intrastat engine (Gemini + tariff DB + OpenPyXL) extracts, enriches, and builds Excel in memory.
5. Progress events stream over SSE; on completion frontend triggers download.
6. Optional bug report flow sends text + screenshot to backend, which relays via Resend.

## Suggested Tests

- Frontend: `npm run lint`; manual flow: login ? upload sample PDFs ? watch progress ? download Excel.
- Backend: call `/health`, `/api/test` with a valid Supabase JWT; upload small PDFs to `/api/jobs` and verify Excel download; exercise SSE by keeping tab open during processing.

## Next Steps

- Implement Stripe endpoints on the backend (see plan below) and wire env vars; enable `VITE_STRIPE_ENABLED` in frontend once live.
- Add automated integration test with mocked Supabase JWT and a small sample PDF to verify end-to-end job lifecycle.
