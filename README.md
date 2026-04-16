# Minotask Web App

Minotask is a web application for turning invoice PDFs into Intrastat-ready reports with a guided, user-friendly workflow.

Simple explanation: users upload invoice documents, the app processes the data behind the scenes, and the finished report can be reviewed and downloaded from the browser.

## Overview

This repository contains the main Minotask product and a separate Prompt Lab workspace used internally for prompt and pipeline iteration.

- `frontend/` contains the customer-facing web app.
- `backend/` contains the API and report-processing services.
- `prompt-lab/` contains a local development workspace for Prompt Lab.

## Product Areas

- PDF upload and report generation
- Account access and user flows
- Pricing and profile management
- Background job processing with progress updates
- Internal Prompt Lab for experimenting with pipeline stages

## Tech Stack

- Frontend: React, TypeScript, Vite, Tailwind CSS
- Backend: Python, Flask
- Data and auth: Supabase
- Document/report processing: PDF parsing and Excel generation
- AI-assisted pipeline: model-driven extraction and enrichment

## Repository Layout

```text
minotask_webapp/
  backend/
  frontend/
  prompt-lab/
```

## Notes

- The top-level README stays intentionally high-level because this repository is public.
- More detailed setup and workflow notes live inside the individual app folders.
