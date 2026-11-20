# FOLIO — Master Application Build Package

This package combines the complete FOLIO Backend Blueprint V3.0 documentation with the master implementation prompt and Google AI Studio setup instructions.

**It intentionally excludes the Stitch frontend ZIP.** The owner adds that original ZIP separately in `FRONTEND_ZIP_GOES_HERE/`.

Start with `00-START-HERE-GEMINI-AI-STUDIO.md`, then copy the entire contents of `COPY-PASTE-MASTER-PROMPT-GEMINI-AI-STUDIO.md` into the coding agent.

## Package contents

- `BACKEND_BLUEPRINT_V3/` — 42 files of engineering/product reference documents, OpenAPI starter contract, ERD/schema documentation, frontend inventory/mapping, payment integration, security, deployment, QA and decision registers.
- `COPY-PASTE-MASTER-PROMPT-GEMINI-AI-STUDIO.md` — complete implementation prompt.
- `00-START-HERE-GEMINI-AI-STUDIO.md` — setup and usage guide.
- `FRONTEND_ZIP_GOES_HERE/` — placeholder for the owner-supplied frontend ZIP.

## Important status

This is a requirements and implementation handoff package, not source code for a running application. The agent must inspect the actual repository and implement the application. Production readiness requires owner decisions, merchant/provider credentials, code, migrations, tests and real deployment evidence.
