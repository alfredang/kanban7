# kanban7 — IT PMO Kanban

A single-file Kanban board for the IT PMO of a fictitious bank, in vanilla HTML, CSS and JavaScript.

[![CI and Deploy](https://github.com/alfredang/kanban7/actions/workflows/pages.yml/badge.svg)](https://github.com/alfredang/kanban7/actions/workflows/pages.yml)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-vanilla-F7DF1E?logo=javascript&logoColor=black)
![No dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-live-222222?logo=githubpages&logoColor=white)](https://alfredang.github.io/kanban7/)

**Live demo:** https://alfredang.github.io/kanban7/

![IT PMO Project Board with a navy header showing task totals (8 total, 2 overdue), a filter bar for project, assignee and priority, and four columns — Backlog, In Progress, Blocked and Done — holding ITPM-numbered task cards with priority and category badges and red Overdue flags](docs/screenshot.png)

A Kanban board for the IT Project Management Office of a **fictitious** bank, built as an internal demo and training tool. The whole app is one file, `index.html`, in vanilla HTML, CSS and JavaScript.

> **Sample data only.** Every task, project, person and ID on the board is made up. "IT PMO" is a neutral wordmark, not a real bank's brand.

## Features

- Four columns: **Backlog**, **In Progress**, **Blocked**, **Done**.
- Add tasks through a modal form with inline validation (title, description, project, category, assignee, priority, due date). IDs are generated as `ITPM-####`.
- Move cards by drag-and-drop, or with the keyboard-accessible **Move ▸** menu.
- Delete with an inline "Delete? Yes / No" confirmation on the card.
- Filter by project, assignee (substring) and priority.
- A summary strip with counts per status and an **Overdue** count (due date before today, status not Done).
- Optional email notification for each new task through [FormSubmit](https://formsubmit.co).

**Nothing is saved.** The board lives in memory, so refreshing the page resets it to the sample data. This is deliberate. Sample due dates are relative to today, so some tasks are always overdue.

## Installation

There is no build step, package manager or dependencies. Clone the repository:

```bash
git clone https://github.com/alfredang/kanban7.git
cd kanban7
```

## Running locally

Open `index.html` in a browser:

| OS | Command |
| --- | --- |
| Windows (PowerShell) | `Start-Process index.html` |
| macOS | `open index.html` |
| Linux | `xdg-open index.html` |

Email notifications usually fail from `file://`. The board keeps working and shows a warning toast. To test real email delivery, serve the folder over http:

```
python -m http.server 8000
```

Then open http://localhost:8000.

## Configuration

All settings are at the top of the `<script>` block in `index.html`:

| Constant | Purpose |
| --- | --- |
| `FORMSUBMIT_ENDPOINT` | Where new-task emails go. **Replace `YOUR_EMAIL@example.com`** with a real address. This is the only place the address appears. |
| `ID_PREFIX` | Task ID prefix (`ITPM`). |
| `SUBJECT_PREFIX` | Email subject prefix (`[IT PMO]`). |

FormSubmit needs a one-time activation. The **first** submission to a new address is not delivered. Instead, FormSubmit emails a confirmation link to that address, and later submissions arrive only after someone clicks it.

Projects, categories, priorities and statuses come from the `PROJECTS`, `CATEGORIES`, `PRIORITIES` and `STATUSES` arrays. These drive both the dropdowns and validation, so edit them there, not in the markup.

## CI/CD

[.github/workflows/pages.yml](.github/workflows/pages.yml) runs two jobs:

1. **Checks** run on every push and pull request to `main`:
   - a secret scan with [gitleaks](https://github.com/gitleaks/gitleaks)
   - HTML validation with [html-validate](https://html-validate.org) (rules in [.htmlvalidate.json](.htmlvalidate.json))
   - a project-rules check that fails on `localStorage`, `sessionStorage`, IndexedDB, cookies, `alert()`, `confirm()`, `!important` or external scripts and stylesheets
2. **Deploy** runs on pushes to `main`, and only if the checks pass. It publishes `index.html` to GitHub Pages. Only the app file is deployed, so repo files such as `CLAUDE.md` are not served.

You can also run the workflow manually from the Actions tab.

## Project structure

```
index.html                  The whole app: markup, <style> and <script>
docs/screenshot.png         Screenshot used in this README
.github/workflows/pages.yml CI checks and GitHub Pages deploy
.htmlvalidate.json          HTML validation rules
CLAUDE.md                   Architecture notes and constraints
```

## Contributing: conventions to keep

- Vanilla HTML/CSS/JS only: no frameworks, libraries, CDNs, web fonts or image assets. Icons are inline SVG or Unicode.
- No persistence: no `localStorage`, `sessionStorage`, IndexedDB or cookies.
- No `alert()` or `confirm()`. Errors appear inline, and delete uses the inline confirm row.
- No `!important`. Colours and spacing come from the CSS custom properties on `:root`.
- Run every user-supplied string through `escapeHtml()` before it goes into an HTML template.
- State flows one way: actions update `state`, then call `renderBoard()`. Don't edit card DOM directly.
- Dates use local time (`toISODate()` / `todayISO()`), not UTC.
- Keep neutral branding. Never use a real bank's name, logo or trademarks.

See [CLAUDE.md](CLAUDE.md) for a fuller architecture walkthrough.

## Testing

There is no automated test suite. Test manually in a browser: add, move, filter and delete tasks, check the inline validation messages, and try the keyboard-only Move menu.
