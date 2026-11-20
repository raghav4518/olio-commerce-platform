# FOLIO — Start Here (GitHub / Google AI Studio)

This repository is a **reference-and-handoff workspace** for building FOLIO. It is not yet the runnable application.

## Folder map

- `FRONTEND_DESIGN_EXPORT/` — all 83 extracted Google Stitch screen exports (51 Customer, 18 Owner/Admin, 14 Support/Operations) plus the design-system file.
- `BACKEND_BLUEPRINT_V3/` — the complete V3 product, database, API, payment, security, workflow, testing, deployment and traceability documents.
- `MASTER-IMPLEMENTATION-PROMPT-GEMINI-AI-STUDIO.md` — complete copy/paste implementation prompt adapted to these extracted folders.
- `docs/` — inventory, checksums and setup notes.

## Use in Google AI Studio

1. Create/use a private GitHub repository.
2. Push this entire folder structure to the repository without keeping it inside another nested copy of the repository.
3. Import the GitHub repository into Google AI Studio Build Mode.
4. Open `MASTER-IMPLEMENTATION-PROMPT-GEMINI-AI-STUDIO.md`, copy the entire contents and paste into the coding agent.
5. Require a read-only audit first. The agent must inspect all 83 screen exports and all V3 documents before changing source code.
6. Implement phase-by-phase, run tests, record evidence, and commit each coherent phase.

## Important

The HTML files are design references, not a working frontend application. V3 is an engineering specification, not compiled backend code. The coding agent must implement the actual routes, services, database migrations, API handlers, payment provider integration, webhook verification, ebook storage/access, tests and deployment configuration.

Do not commit API keys, payment credentials, production database passwords, private ebook originals, customer data or secret environment files. Use the host's secrets manager or environment configuration.
