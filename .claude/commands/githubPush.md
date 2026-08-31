---
description: Security-scan, document, push to GitHub, and publish GitHub Pages
argument-hint: <github-repo-url> (e.g. https://github.com/user/repo.git)
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

## Goal

Publish this repository to GitHub safely: run a security scan first, write/refresh
the README, push the code, wire up GitHub Pages via a GitHub Action, and set the
repo's About section (description + Pages link).

## Input

The target GitHub repository is: `$ARGUMENTS`

- If `$ARGUMENTS` is empty, ask the user for the repo URL (HTTPS or SSH) before doing anything.
- Accept either an existing empty repo or a repo that already has this project's history.
- Derive `OWNER/REPO` from the URL for later `gh`/API steps.

## Order of operations (do NOT reorder — the scan gates everything)

### 1. Security scan — BEFORE anything leaves the machine

Nothing is pushed until this passes. Scan the working tree and the full git history
that would be pushed:

- List everything that would be uploaded: `git status`, `git log --oneline origin/main..HEAD 2>/dev/null || git log --oneline`, and `git ls-files`.
- Grep the tracked files AND history for secrets and confidential data:
  - Credentials / tokens: `password`, `passwd`, `secret`, `api[_-]?key`, `token`,
    `bearer`, `authorization`, `client[_-]?secret`, `private[_-]?key`,
    `-----BEGIN .*PRIVATE KEY-----`, AWS keys (`AKIA[0-9A-Z]{16}`), Google API keys
    (`AIza[0-9A-Za-z_-]{35}`), Slack (`xox[baprs]-`), GitHub PATs (`ghp_`, `github_pat_`),
    JWTs (`eyJ[A-Za-z0-9_-]+\.`).
  - Env / config files that should not ship: `.env`, `.env.*`, `*.pem`, `*.key`,
    `*.p12`, `*.keystore`, `id_rsa`, `credentials`, `*.pfx`, service-account JSON.
  - PII: real personal emails, phone numbers, national IDs, card numbers (Luhn-looking
    16-digit runs), internal hostnames / IP ranges.
  - Search history too: `git grep -nEi '<pattern>' $(git rev-list --all)` for the
    high-signal patterns above.
- This project has explicit constraints in [CLAUDE.md](../../CLAUDE.md): it must stay
  a single offline `index.html`. Confirm no backend config, no real UOB data, and that
  placeholders (e.g. `1800-XXX-XXXX`) are still placeholders.
- Ensure a `.gitignore` covers `.env*`, key material, `.DS_Store`, and editor/OS cruft.

Report findings as a short table (file · line · what · severity). Then:

- **If anything confidential is found: STOP.** Do not push. Tell the user exactly what
  and where, and propose remediation (remove file + `git rm --cached`, scrub history
  with `git filter-repo`, rotate the exposed secret). Wait for the user to confirm the
  fix before continuing.
- **If clean:** state "Security scan clean — safe to push" and continue.

### 2. Create / edit the README

- If `README.md` exists, update it; otherwise create it.
- It must describe: what the project is (training/demo mockup, **not** a real product,
  not affiliated with UOB), how to run it (`open index.html`, works from `file://`,
  no build/deps), what's inside, and the deliberate security/privacy constraints from
  [CLAUDE.md](../../CLAUDE.md).
- Add a live demo link line pointing at the Pages URL from step 5:
  `https://OWNER.github.io/REPO/`.
- Keep the disclaimer: "Training demo — not affiliated with or endorsed by United
  Overseas Bank Limited."

### 3. GitHub Pages via GitHub Action

- Check `.github/workflows/` for an existing Pages deploy workflow. This repo already
  has [.github/workflows/deploy-pages.yml](../../.github/workflows/deploy-pages.yml) —
  reuse/verify it rather than adding a duplicate.
- The workflow must: trigger on push to `main` + `workflow_dispatch`; set
  `permissions: contents: read, pages: write, id-token: write`; assemble a `_site/`
  dir with `index.html`; use `actions/configure-pages`, `actions/upload-pages-artifact`,
  `actions/deploy-pages`.
- If it's missing or wrong, write it.

### 4. Push the code

- `git remote add origin $ARGUMENTS` (or `git remote set-url origin $ARGUMENTS` if
  `origin` already points elsewhere — show the user the change first).
- Ensure the branch is `main`.
- Commit any pending scan-related fixes with a clear message (do not amend existing
  history unless the user asked). Co-author line per repo convention.
- `git push -u origin main`.
- If the push is rejected (non-fast-forward), stop and ask — do not force-push without
  explicit user approval.

### 5. About section + Pages link

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
- If step 2's README used a placeholder URL, update it now with the real
  `html_url` and push that follow-up commit.

## Final report

Summarise: scan result, README status, workflow status, push result (commit SHA),
Pages URL, and the About/description/homepage that were set. Call out anything the
user still needs to do manually (e.g. run `gh` steps, approve the first Pages
deployment environment).
