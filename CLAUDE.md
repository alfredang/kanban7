# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An IT PMO Kanban board for a **fictitious** bank, used as an internal demo/training tool. The whole app is one file, `index.html`: markup, a `<style>` block and a `<script>` block.

## Running

There is no build, lint, test suite or package manager. Open `index.html` directly in a browser (on Windows, `Start-Process index.html`).

The FormSubmit email call usually fails from `file://`, and the app is designed to cope: the board keeps working and shows a warning toast. To test real email delivery, serve the folder over http, e.g. `python -m http.server`, then open `http://localhost:8000`.

## Hard constraints (from the original brief — do not violate)

- Vanilla HTML/CSS/JS only. No frameworks, libraries, CDNs, web fonts, image files or build step. Icons are inline SVG or Unicode.
- **No persistence.** Never use `localStorage`, `sessionStorage`, IndexedDB or cookies. A refresh resets the board to the seed data on purpose, and the header note tells users so.
- No `alert()` or `confirm()`. Validation errors appear inline under each field. Delete is confirmed with an inline "Delete? Yes / No" row on the card.
- No `!important`. Colours and spacing come from the CSS custom properties on `:root`.
- Neutral branding only: the "IT PMO" wordmark, IDs prefixed `ITPM-####`, and email subjects starting `[IT PMO]`. Never use a real bank's name, logo or trademarks.
- Escape every user-supplied string with `escapeHtml()` before it goes into an HTML template.
- The only network call is `notifyNewTask()` to `FORMSUBMIT_ENDPOINT`. It must stay inside try/catch, so a failure can never break the board.

## Architecture (inside the `<script>`)

- **Config at the top**: `FORMSUBMIT_ENDPOINT`, `ID_PREFIX` and `SUBJECT_PREFIX`. Change the email address only there. The HTML comment above the `<script>` explains FormSubmit's one-time activation email.
- **Single source of truth**: `state = { tasks, filters, ui }`. `state.ui` holds transient card UI (`moveMenuId`, `confirmDeleteId`), so even toggles are re-rendered from state rather than mutated in the DOM.
- **One-way flow**: actions (`addTask`, `moveTask`, `deleteTask`, plus handlers that change `state.ui`/`state.filters`) update state, then call `renderBoard()`. `renderBoard()` rebuilds every column from `applyFilters(state.tasks)` with `renderCard()` and refreshes `renderSummary()`. Don't edit card DOM anywhere else.
- **Keyboard focus**: because the board is rebuilt with `innerHTML`, `renderBoard(focus)` accepts `{ id, action }` or `{ column }`, and `restoreFocus()` uses it to put focus back.
- **Events**: one delegated `click` listener on `#board` dispatches on the buttons' `data-action` (`move-toggle`, `move-to`, `move-cancel`, `delete-ask`, `delete-yes`, `delete-no`). Drag-and-drop uses native HTML5 events, also delegated on `#board`, and calls `moveTask()`. The "Move ▸" control is the keyboard alternative to dragging.
- **Add Task**: on submit, `validateForm()` → `addTask()` (the card appears at once, optimistic UI) → reset and close the modal → then `notifyNewTask()` runs while `setSending()` shows "Sending…". On failure the card is kept and a warning toast appears. `notifyNewTask()` also treats FormSubmit's `{ success: "false" }` as a failure even when the HTTP status is 200.
- **Dates**: `toISODate()`/`todayISO()` use the local date, not UTC, so dates don't shift a day near midnight. Seed due dates come from `addDays()`, so some tasks are always overdue whenever the page is opened. `isOverdue()` = due date before today and status ≠ Done.
- **Reference lists**: `PROJECTS`, `CATEGORIES`, `PRIORITIES` and `STATUSES` drive both the selects and validation. Change them there, not in the markup.
