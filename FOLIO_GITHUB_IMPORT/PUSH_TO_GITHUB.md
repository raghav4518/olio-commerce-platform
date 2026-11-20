# Push this FOLIO reference workspace to GitHub

The connected GitHub account selected for this task is `raghav4518`. The account currently has no repositories visible to the connected GitHub integration, and the integration does not expose a create-repository operation. Create the empty private repository once, then push this folder with GitHub Desktop or Git.

## 1. Create the destination repository

1. Sign in to GitHub as `raghav4518`.
2. Open https://github.com/new
3. Repository name: `folio-commerce-platform`
4. Visibility: **Private**.
5. Leave **Add a README**, `.gitignore`, and license unchecked so the repository is empty.
6. Click **Create repository**.

## 2. Push with GitHub Desktop (recommended)

1. Download/open GitHub Desktop: https://desktop.github.com/
2. Choose **File → Clone repository**.
3. Select `raghav4518/folio-commerce-platform` and choose a local destination.
4. Extract `FOLIO_GITHUB_IMPORT_READY.zip`.
5. Copy the *contents* of the extracted `FOLIO_GITHUB_IMPORT/` directory into the cloned repository root. The root should directly contain `FRONTEND_DESIGN_EXPORT/`, `BACKEND_BLUEPRINT_V3/`, `docs/`, `README.md`, and the prompt files. Do not copy the outer folder as an extra nested directory.
6. In GitHub Desktop, review the changed files. Commit with message `Add FOLIO frontend references and Backend Blueprint V3`.
7. Click **Push origin**.

## 3. Alternative: Git command line

After creating the empty repository and extracting this package, run these commands from the extracted `FOLIO_GITHUB_IMPORT` directory. Replace `<LOCAL-PATH-TO-EXTRACTED-FOLDER>` with the actual path:

```bash
git init -b main
git add .
git commit -m "Add FOLIO frontend references and Backend Blueprint V3"
git remote add origin https://github.com/raghav4518/folio-commerce-platform.git
git push -u origin main
```

Git may prompt you to authenticate using your GitHub credential manager/browser. Do not put a personal access token in a command, source file, or commit.

## 4. Verify before importing into AI Studio

Open https://github.com/raghav4518/folio-commerce-platform and verify that the repository root directly shows:

- `FRONTEND_DESIGN_EXPORT/`
- `BACKEND_BLUEPRINT_V3/`
- `MASTER-IMPLEMENTATION-PROMPT-GEMINI-AI-STUDIO.md`
- `00-START-HERE.md`

Then import this GitHub repository into Google AI Studio Build Mode.
