---
description: Security-scan this project, then publish it to GitHub with README, About section, GitHub Pages and CI/CD
argument-hint: <owner/repo or https://github.com/owner/repo> [public|private]
allowed-tools: Bash, PowerShell, Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# Publish this project to GitHub

GitHub info from the user: `$ARGUMENTS`

Work through the phases below **in order**. Phase 1 is a gate: nothing leaves this machine until it passes. Report progress briefly after each phase.

## Phase 0 — Gather GitHub info and check tooling

1. Parse `$ARGUMENTS` for the repository (`owner/repo` or a `https://github.com/owner/repo(.git)` URL) and optional visibility (`public` / `private`, default `public` because GitHub Pages on a free plan needs a public repo).
2. If the repository is missing, check `git remote get-url origin`. If there is still no repository, ask the user with AskUserQuestion. Do not guess an owner or repo name.
3. Check the tools:
   - `git --version`
   - `gh --version` and `gh auth status`. If `gh` is missing or not logged in, stop and tell the user to run `gh auth login`. Never ask for, paste or store a token.
4. Run `git status` and `git log --oneline -5` so you know what is about to be pushed.

## Phase 1 — Security scan (blocking gate)

The aim: **no secrets, credentials, personal data or private files get uploaded to the internet.** Scan the working tree *and* the full git history, since a secret deleted in a later commit is still published by a push.

1. **List exactly what would be published**: `git ls-files` plus untracked, non-ignored files (`git ls-files --others --exclude-standard`). Flag anything that should not be public, such as `.env*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa*`, `*.sqlite`/`*.db`, `*.log`, `.claude/settings.local.json`, `.playwright-mcp/`, `node_modules/`, OS junk (`Thumbs.db`, `.DS_Store`) and files over 5 MB.
2. **Make sure `.gitignore` covers these.** Add missing entries; keep the existing ones.
3. **Scan for secrets**:
   - If `gitleaks` is installed, run `gitleaks detect --source . --redact -v` (this covers the history) and `gitleaks detect --source . --no-git --redact -v` (working tree).
   - Otherwise, use Grep over the tracked files, and `git log -p --all` for the history, with these patterns at least:
     - `AKIA[0-9A-Z]{16}` (AWS key)
     - `gh[pousr]_[A-Za-z0-9]{36,}`, `github_pat_[A-Za-z0-9_]{50,}` (GitHub tokens)
     - `sk-[A-Za-z0-9_-]{20,}`, `sk-ant-[A-Za-z0-9_-]{20,}` (OpenAI / Anthropic keys)
     - `AIza[0-9A-Za-z_-]{35}` (Google API key)
     - `xox[baprs]-[A-Za-z0-9-]{10,}` (Slack token)
     - `-----BEGIN [A-Z ]*PRIVATE KEY-----`
     - `(?i)(password|passwd|secret|api[_-]?key|token|client[_-]?secret)\s*[:=]\s*['"][^'"]{6,}['"]`
     - `(?i)(mongodb(\+srv)?|postgres(ql)?|mysql|redis)://[^\s'"]+:[^\s'"]+@` (connection strings with credentials)
4. **Check for personal data.** Grep for email addresses and phone numbers. Placeholders such as `YOUR_EMAIL@example.com` and `noreply@anthropic.com` are fine. For this project, check that `FORMSUBMIT_ENDPOINT` in `index.html` still points at the placeholder and not at a real inbox. A real address there would be published on GitHub Pages and invite spam.
5. **Check project-specific rules from CLAUDE.md**: no real bank names, logos or trademarks; no `localStorage`/`sessionStorage`/IndexedDB/cookies; the only network call is `notifyNewTask()` to `FORMSUBMIT_ENDPOINT`. Grep for `fetch(`, `XMLHttpRequest`, `sendBeacon`, `WebSocket`, `<script src=`, `<link href="http` and `@import url(` to confirm nothing else calls out to the internet.
6. **Result**:
   - If anything is found, **stop**. Show each finding as `file:line`, with the secret value redacted, and propose a fix: remove it, move it into an ignored `.env`, or replace it with a placeholder. If the secret is in git history, tell the user it must be **revoked/rotated** and the history rewritten (for example with `git filter-repo`) before pushing. Do not rewrite history without explicit approval.
   - Only continue when the scan is clean, or the user has explicitly accepted each remaining finding.

## Phase 2 — README

Create `README.md` if it is missing, or update it if it exists. Keep accurate existing content and the project's tone. It must include:

1. Title, with a one-line summary.
2. **Tech badges** (shields.io), matching what the project actually uses. For this project: HTML5, CSS3, JavaScript (vanilla), "No dependencies", GitHub Pages, a workflow status badge (`https://github.com/<owner>/<repo>/actions/workflows/<workflow-file>/badge.svg`) and a licence badge if a LICENSE file exists.
3. A **Live demo** link: `https://<owner>.github.io/<repo>/` (all lower case for the owner).
4. A screenshot, if one exists under `docs/`, with descriptive alt text.
5. Features.
6. **How to install**: clone with `git clone https://github.com/<owner>/<repo>.git` and `cd <repo>`; state plainly that there is no build step or package manager.
7. **How to run**: open `index.html` directly (`Start-Process index.html` on Windows, `open index.html` on macOS, `xdg-open index.html` on Linux), *and* serve over http (`python -m http.server` → `http://localhost:8000`) to test the FormSubmit email, which fails from `file://`.
8. Configuration, such as where to set the FormSubmit email (`FORMSUBMIT_ENDPOINT`) and the one-time activation email.
9. A CI/CD section that describes the workflow.
10. Project structure, and a note that all data is fictitious.

Do not put real personal data in the README.

## Phase 3 — CI/CD with GitHub Actions

Create or update `.github/workflows/pages.yml` (reuse the existing file if there is one) as a **two-job pipeline** triggered on `push` to `main`, `pull_request` to `main`, and `workflow_dispatch`:

1. **`ci` job** (runs on every trigger), with `permissions: contents: read`:
   - `actions/checkout@v4` with `fetch-depth: 0`.
   - Secret scan with `gitleaks/gitleaks-action@v2` (env `GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}`).
   - HTML validation, for example `npx --yes html-validate@8 index.html`. Add a minimal `.htmlvalidate.json` (`{"extends":["html-validate:recommended"]}`) and relax rules only where the existing markup is intentionally different. Do not change the app's behaviour to satisfy the linter.
   - A project-rules check in a shell step that fails the build when `index.html` contains `localStorage`, `sessionStorage`, `indexedDB`, `document.cookie`, `alert(`, `confirm(` or `!important`, or when it loads any external `<script src=` or stylesheet.
2. **`deploy` job**: `needs: ci`, run only when `github.event_name != 'pull_request'` and on `refs/heads/main`. Permissions `contents: read`, `pages: write`, `id-token: write`. Use the `github-pages` environment, `actions/configure-pages@v5` (with `enablement: true`), `actions/upload-pages-artifact@v3` and `actions/deploy-pages@v4`.
   - Upload a **clean `_site` folder** that holds only what the site needs (`index.html` plus any assets it references, e.g. `docs/`), *not* `path: .`, so that `CLAUDE.md`, `.claude/` and other repo files are not served on the public site.
3. Keep `concurrency: { group: pages, cancel-in-progress: false }`.

## Phase 4 — Commit and push

1. Show the user a short summary: files changed, target repo, visibility, branch. **Ask for confirmation before pushing**, since this publishes the code.
2. If the GitHub repo does not exist yet (`gh repo view <owner>/<repo>` fails), create it with `gh repo create <owner>/<repo> --<visibility> --source . --remote origin`. If `origin` exists but points elsewhere, ask before changing it.
3. Stage files **by name** (never `git add -A` or `git add .`), re-run the quick secret grep on `git diff --cached`, then commit with a clear message ending with the attribution line from the session's system reminder.
4. Push with `git push -u origin main`. Never force-push without explicit approval.

## Phase 5 — GitHub Pages

1. Enable Pages with the Actions source:
   `gh api -X POST repos/<owner>/<repo>/pages -f build_type=workflow`
   If it already exists (HTTP 409), update it: `gh api -X PUT repos/<owner>/<repo>/pages -f build_type=workflow`.
2. Watch the workflow run: `gh run list --workflow pages.yml -L 1`, then `gh run watch <run-id> --exit-status`.
3. If the run fails, read `gh run view <run-id> --log-failed`, fix the cause, commit and push again (with the same confirmation rule), and repeat.
4. Get the live URL with `gh api repos/<owner>/<repo>/pages --jq .html_url`, then check it responds: `curl -sI <url>` should return 200.

## Phase 6 — About section

Update the repository's About section with the Pages link, a description and topics:

```bash
gh repo edit <owner>/<repo> \
  --description "<one-line summary of the project>" \
  --homepage "https://<owner>.github.io/<repo>/" \
  --add-topic kanban --add-topic html --add-topic css --add-topic javascript \
  --add-topic vanilla-js --add-topic github-pages --add-topic project-management
```

Choose topics that fit the project (lower case, hyphenated, no more than 20). Confirm with `gh repo view <owner>/<repo> --json description,homepageUrl,repositoryTopics`.

## Phase 7 — Post-publish security check

1. Compare the published files with what you approved: `gh api repos/<owner>/<repo>/git/trees/main?recursive=1 --jq '.tree[].path'`. Nothing from the Phase 1 deny-list may appear.
2. Check that the live site serves only the intended files: `curl -s -o /dev/null -w "%{http_code}" https://<owner>.github.io/<repo>/CLAUDE.md` should return 404.
3. If GitHub secret scanning is available, check it: `gh api repos/<owner>/<repo>/secret-scanning/alerts` (a 404 or "disabled" response is fine on free private repos; say so).

## Final report

End with a short summary:
- Security scan result (clean, or findings and how each was resolved)
- Repo URL, live GitHub Pages URL, latest workflow run status
- About section: description, homepage and topics that were set
- Anything skipped or still needing the user's action (e.g. the FormSubmit activation email, or rotating a leaked key)
