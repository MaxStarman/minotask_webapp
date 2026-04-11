# minotask_webapp

Minotask is a small web app that turns uploaded invoice PDFs into an Intrastat-style Excel report.
This repo is a simple "frontend + backend" setup: the browser UI calls an API, the API does the work, and the UI shows progress + lets you download the result.

## What's inside

- `frontend/`: React (Vite + Tailwind) UI - the part you open in the browser.
- `backend/`: Flask API - the part that accepts uploads, runs processing, and returns results.

## High-level architecture (how everything is connected)

1. User opens the frontend and signs in (Supabase is used for auth/session).
2. Frontend calls the backend API with the user token (so the backend knows who is calling).
3. Backend starts a "job" when PDFs are uploaded (this is the long-running work).
4. Frontend listens for progress (SSE is used so the UI can update while the job runs).
5. Backend produces an Excel file, and the frontend offers a download when the job is done.

That's basically it: UI -> API -> background processing -> progress updates -> download.

## Running locally (very roughly)

- Backend: run the Flask app in `backend/` (you'll need environment variables for auth + any AI/services you use).
- Frontend: run the Vite dev server in `frontend/` and point it at the backend URL.

## Notes

- The backend exposes a small REST API plus an SSE endpoint for progress.
- The frontend uses a small API client and a progress hook to tie the UI to backend jobs.
