# Package Manifest — FOLIO Backend Blueprint V3.0.0

**Prepared:** 20 November 2026  
**Package type:** Backend, database, integration and deployment engineering specification aligned to the supplied frontend screen export.

## Inputs to provide to the coding agent

1. `stitch_folio_digital_bookstore_ux ...zip` — the original frontend export from Google Stitch (83 HTML screen exports plus the design-system file).
2. `FOLIO-Backend-Blueprint-v3.0.zip` — this package.
3. The actual application source repository and its current branch/history.
4. Securely configured environment credentials only through the agent's approved secrets mechanism, never inside prompts, source files or commits.

## Package contents

- `README.md` — purpose, authority hierarchy, status and production-readiness definition.
- `AGENT-HANDOFF-PROMPT.md` — short pointer to the master prompt.
- `agent/MASTER-IMPLEMENTATION-PROMPT.md` — complete ready-to-paste implementation prompt.
- `frontend-reference/folio_design_system.md` — copied design-system reference from the supplied frontend export.
- `frontend-reference/FRONTEND-EXPORT-TECHNICAL-OBSERVATIONS.md` — verified inspection findings and migration cautions.
- `frontend-reference/SCREEN-INVENTORY.csv` — all 83 supplied screens, screen IDs, titles, viewport, and suggested route.
- `backend/15-FRONTEND-SCREEN-TO-BACKEND-MAPPING.md` — every screen mapped to domains, data, permissions and expected service contract.
- `backend/16-PRODUCTION-COMMERCE-AND-PAYMENT-INTEGRATION.md` — provider adapter, payment verification, UPI/card/netbanking UX, webhook, refunds and reconciliation requirements.
- `backend/17-DEPLOYMENT-RUNBOOK-AND-GO-LIVE-GATES.md` — environment, security, release, backup, restore, rollback and go-live.
- `backend/18-DATABASE-IMPLEMENTATION-DELIVERABLES.md` — requirements for turning the V2 logical data model into tested, repository-specific migrations.
- `backend/19-IMPLEMENTATION-EXECUTION-AND-ACCEPTANCE.md` — phased execution, evidence and acceptance gates.
- All V2 backend, product, decision, ADR, standards and template documents retained and included.

## Verified frontend archive facts

- 83 individual `code.html` screen files.
- 51 Customer screens.
- 18 Owner/Admin screens.
- 14 Support/Operations screens.
- 1 `folio_design_system.md`.
- Screen exports are standalone HTML design references; they are not a complete routed application source project.
- 83 HTML files reference the Tailwind CDN.
- The supplied export references Google Fonts and Material Symbols.
- 15 remote `lh3.googleusercontent.com` image URLs were found.
- 23 form elements were found across the HTML exports.
- No package manifest, app router, local JS/CSS component tree, database migrations or API client is present in the frontend export.

## Important status limitations

This package does not itself implement code, choose a payment provider, provision infrastructure, create merchant credentials, or approve tax/refund/privacy/legal policy. It directs the implementation agent to create these implementation artefacts after auditing the repository and obtaining required owner approvals.

Do not tell customers that live payments are available until the selected provider's live account, credentials, webhook, settlement and reconciliation tests have passed.
