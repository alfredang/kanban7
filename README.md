# kanban7 — IT PMO Kanban

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

## Running locally

There is no build step, package manager or dependencies. Open `index.html` in a browser:

```powershell
Start-Process index.html
```

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

## Deployment

Every push to `main` deploys the repository root to GitHub Pages via [.github/workflows/pages.yml](.github/workflows/pages.yml). You can also run the workflow manually from the Actions tab.

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
