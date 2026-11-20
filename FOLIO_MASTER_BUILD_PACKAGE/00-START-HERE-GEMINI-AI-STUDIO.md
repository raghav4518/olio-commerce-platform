# FOLIO Master Build Package — Start Here (Google AI Studio)

**Package version:** 1.0  
**Prepared:** 20 November 2026  
**Purpose:** One backend/engineering documentation package plus one master prompt. The owner adds the frontend ZIP separately.

## What this package contains

- `BACKEND_BLUEPRINT_V3/` — the complete FOLIO V3 backend, database, API, payment, security, workflow, integration, testing, deployment and product documentation.
- `COPY-PASTE-MASTER-PROMPT-GEMINI-AI-STUDIO.md` — the full implementation prompt for Google AI Studio Build Mode / Gemini coding agent.
- `FRONTEND_ZIP_GOES_HERE/` — a placeholder folder where you put your original Google Stitch frontend ZIP yourself.

**The frontend ZIP is intentionally not included in this master package.**

## Recommended setup

### 1. Extract this master ZIP

Extract the complete package into a folder. Do not remove or rename `BACKEND_BLUEPRINT_V3/` or the master prompt.

### 2. Add your frontend ZIP

Copy the original Stitch ZIP into:

`FRONTEND_ZIP_GOES_HERE/`

Keep the original archive unchanged. It should contain the 83 screen HTML exports and `folio_design_system.md`. You may instead extract its contents into a folder inside `FRONTEND_ZIP_GOES_HERE/`, preserving the full directory structure.

### 3. Put the package into the application repository

Recommended repository layout:

```text
folio-commerce-platform/
├── BACKEND_BLUEPRINT_V3/
├── FRONTEND_ZIP_GOES_HERE/
│   └── <your-original-stitch-frontend.zip>
├── COPY-PASTE-MASTER-PROMPT-GEMINI-AI-STUDIO.md
└── README.md
```

If you already have a FOLIO code repository, place the documentation package into it without overwriting source files. Create a dedicated branch first. If you do not have a repository, create a private GitHub repository and add these package files.

Do not commit production secrets, customer data, database dumps, private ebook files, or payment credentials. The Stitch archive is a design reference, not a licensed production asset bundle; the agent must review remote images/fonts and replace assets as needed.

### 4. Import the repository into Google AI Studio

1. Open [Google AI Studio](https://aistudio.google.com/).
2. Enter **Build** mode.
3. Use **Add files (+) → Import from GitHub** to import the repository, if you are using GitHub.
4. Confirm that `BACKEND_BLUEPRINT_V3/`, `FRONTEND_ZIP_GOES_HERE/` and the master prompt are visible in the project.
5. Select Gemini 4 Argon in the model selector only if it is available in your account/session. Otherwise use the strongest available coding/reasoning model. The prompt does not assume a model that the interface does not expose.

Google's Build Mode documentation describes a full-stack web app environment (React by default on the client and Node.js server runtime), GitHub import/sync, preview and Cloud Run deployment. Exact controls/features may change; follow the current UI shown in your account.

### 5. Paste the master prompt

Open `COPY-PASTE-MASTER-PROMPT-GEMINI-AI-STUDIO.md`, copy the entire file, and paste it into the Build Mode agent prompt.

The master prompt requires the agent to:
- inspect the real repository before changing it;
- locate and inspect the frontend ZIP you added;
- read the full V3 documentation;
- verify all 83 screens;
- map screens to routes, APIs, data and permissions;
- implement the full application in phases;
- implement real persistence, checkout, payment provider integration, verified webhooks, refunds and ebook entitlements;
- test authentication, access boundaries, payment flows, security and operations;
- prepare staging, deployment, backup/restore and rollback documentation;
- never claim production readiness without proof.

### 6. Expected first response from the agent

The agent should produce a repository and archive audit before making risky changes. It should verify that the frontend ZIP contains the expected 83 screens and that the backend files are readable. It should then create an implementation plan and continue with safe work. It must stop only where an actual owner decision or external credential is required.

### 7. Owner decisions and external setup

The blueprint deliberately keeps these items open until you approve them:
- launch country, legal seller entity and currency;
- payment gateway and merchant account;
- tax/GST/invoicing rules and professional/legal review;
- guest checkout and account-claim policy;
- supported ebook formats and reader engine;
- refund eligibility and effect of refund on access;
- hosting, database region, object storage and email providers;
- privacy/retention terms, staff MFA, support SLA, traffic targets and RPO/RTO.

The agent can build provider adapters and sandbox/test flows before all production credentials exist. It must not fabricate live success, use real credentials in source code, or turn an unapproved legal/business policy into production behavior.

## Important limitation

A ZIP of UI exports plus engineering Markdown does not itself create a deployed application. The actual AI coding agent must work in a real source repository/workspace, implement and run code, provision/configure approved services, and produce test evidence. Build preview, successful compilation, generated SQL, or sandbox payments alone do not equal production readiness.

## Official Google resources

- Google AI Studio: https://aistudio.google.com/
- Build mode documentation: https://ai.google.dev/gemini-api/docs/aistudio-build-mode
- Gemini 4 Argon developer announcement: https://blog.google/innovation-and-ai/technology/developers-tools/build-with-gemini-4-argon/
