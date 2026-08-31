---
description: document, push to GitHub, and publish GitHub Pages
argument-hint: <github-repo-url> (e.g. https://github.com/user/repo.git)
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, mcp__playwright__browser_navigate, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_resize, mcp__playwright__browser_close
---

## Goal

Publish this repository to GitHub safely: capture a
screenshot of the page, write/refresh the README (with that screenshot), push the
code, wire up GitHub Pages via a GitHub Action, and set the repo's About section
(description + Pages link).

## Input

The target GitHub repository is: `$ARGUMENTS`

- If `$ARGUMENTS` is empty, ask the user for the repo URL (HTTPS or SSH) before doing anything.
- Accept either an existing empty repo or a repo that already has this project's history.
- Derive `OWNER/REPO` from the URL for later `gh`/API steps.

## Order of operations (do NOT reorder — the scan gates everything)

### 1. Skip this step

### 2. Capture a screenshot for the README

Use the project-level Playwright MCP server (`.mcp.json` → `playwright`). If those
tools are unavailable, tell the user to reload so the project MCP server starts, then
continue without the screenshot rather than blocking the push.

- Playwright blocks `file://`, so serve the page first:
  `python3 -m http.server 8777` from the repo root (run it backgrounded; stop it when done).
- `browser_resize` to 1280×900, `browser_navigate` to `http://localhost:8777/index.html`.
- `browser_take_screenshot` with `fullPage: true`, `type: png`, `filename: screenshot.png`.
- Move the result to `docs/screenshot.png` (create `docs/` if needed), overwriting any
  previous capture. Delete the Playwright scratch dir (`.playwright-mcp/`) and make sure
  `.gitignore` lists it.
- `browser_close` and kill the HTTP server.

### 3. Create / edit the README

- If `README.md` exists, update it; otherwise create it.
- Embed the screenshot near the top under a `## Preview` heading:
  `![Screenshot of the UOB IT Service Desk training demo page](docs/screenshot.png)`.
  If step 2 was skipped, omit this and note it in the final report.
- It must describe: what the project is (training/demo mockup, **not** a real product,
  not affiliated with UOB), how to run it (`open index.html`, works from `file://`,
  no build/deps), what's inside, and the deliberate security/privacy constraints from
  [CLAUDE.md](../../CLAUDE.md).
- Add a live demo link line pointing at the Pages URL from step 6:
  `https://OWNER.github.io/REPO/`.
- Keep the disclaimer: "Training demo — not affiliated with or endorsed by United
  Overseas Bank Limited."

### 4. GitHub Pages via GitHub Action

- Check `.github/workflows/` for an existing Pages deploy workflow. This repo already
  has [.github/workflows/deploy-pages.yml](../../.github/workflows/deploy-pages.yml) —
  reuse/verify it rather than adding a duplicate.
- The workflow must: trigger on push to `main` + `workflow_dispatch`; set
  `permissions: contents: read, pages: write, id-token: write`; assemble a `_site/`
  dir with `index.html`; use `actions/configure-pages`, `actions/upload-pages-artifact`,
  `actions/deploy-pages`.
- If it's missing or wrong, write it.

### 5. Push the code

- `git remote add origin $ARGUMENTS` (or `git remote set-url origin $ARGUMENTS` if
  `origin` already points elsewhere — show the user the change first).
- Ensure the branch is `main`.
- Commit any pending changes (scan fixes, the `docs/screenshot.png` capture, README
  edits) with a clear message (do not amend existing history unless the user asked).
  Co-author line per repo convention.
- `git push -u origin main`.
- If the push is rejected (non-fast-forward), stop and ask — do not force-push without
  explicit user approval.

### 6. About section + Pages link

Prefer the `gh` CLI. If `gh` is not installed or not authenticated, print the exact
manual steps / `curl` API calls for the user and ask them to run them.

- Enable Pages (source = GitHub Actions):
  `gh api -X POST repos/OWNER/REPO/pages -f build_type=workflow` (ignore "already exists").
- Wait for / trigger the deploy: `gh workflow run "Deploy to GitHub Pages"` then
  `gh run watch` (or point the user at the Actions tab).
- Get the live URL: `gh api repos/OWNER/REPO/pages --jq .html_url`.
- Set the About section:
  `gh repo edit OWNER/REPO --description "UOB IT Service Desk — offline training/demo mockup (not a real product)" --homepage "https://OWNER.github.io/REPO/"`
- Optionally add topics: `gh repo edit OWNER/REPO --add-topic demo,training,html,security`.
- If step 3's README used a placeholder URL, update it now with the real
  `html_url` and push that follow-up commit.

## Final report

Summarise: scan result, screenshot status (captured / skipped), README status,
workflow status, push result (commit SHA), Pages URL, and the
About/description/homepage that were set. Call out anything the user still needs to
do manually (e.g. run `gh` steps, approve the first Pages deployment environment).
