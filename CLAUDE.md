# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A fictitious bank's internal IT PMO Kanban board (demo/training tool). The entire app is one file: `index.html` (markup, `<style>`, `<script>`). There is no build, lint, or test tooling and no package manager; run it by opening the file in a browser (`file://` works, no server needed).

## Hard constraints (from the original spec)

- Vanilla HTML/CSS/JS only: no frameworks, bundler, npm, CDN scripts, web fonts, or image files. System font stack and inline glyphs/SVG for icons.
- No persistence of any kind: no localStorage, sessionStorage, IndexedDB, or cookies. Board state is an in-memory array reseeded on every load (the header note says so).
- The only network call is the FormSubmit AJAX POST in `notifyNewTask()`. The notification email address must appear only in `FORMSUBMIT_ENDPOINT` (top of the script). FormSubmit needs one-time activation via a confirmation email to that address.
- No `!important`, no `alert()`/`confirm()`; use inline error text, toasts, and the inline "Delete? Yes / No" card toggle.
- Neutral "IT PMO" text wordmark and blue palette only: no real bank logo or branding. Task IDs (`UOB-ITPM-####`) and the email subject prefix are intentionally literal per the spec.

## Architecture

State-driven rendering: `state = { tasks, filters, confirmDeleteId }` is the source of truth, and `renderBoard()` is the only place card/column DOM is rebuilt (via `innerHTML`, with every user string passed through `escapeHtml()`). Mutators (`addTask`, `moveTask`, `deleteTask`, filter and delete-confirm handlers) change state and then call `renderBoard()`; never edit card DOM directly.

Event handling is delegated on `#board` (drag and drop, delete toggle clicks, Move select `change`), so it survives re-renders. Don't attach listeners to individual cards.

The add-task flow in `handleSubmit()` is optimistic: validate, `addTask()`, reset and close the dialog, success toast, then `notifyNewTask()` in try/catch. A failure only shows a warning toast and must never break the board. The submit button shows "Sending…" while the request is in flight.

Overdue is computed on the fly by `isOverdue()` (ISO date string comparison, status not Done). Seed due dates are offsets from today via `offsetDate()` so the demo always has overdue items.
