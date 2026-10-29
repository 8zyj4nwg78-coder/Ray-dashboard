# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Ray Builders Project Controls: a shared, phone-first project-controls app for a construction team. The whole app is `index.html` (plain HTML/CSS/JS, no build step, no dependencies besides Google Fonts). There is no package manager, linter or test suite.

It runs as a **claude.ai artifact**. The file intentionally has no `<!doctype>`/`<head>`/`<body>` wrapper; the artifact host adds it. Deploying means republishing `index.html` to the existing artifact (not creating a new one), which keeps the shared data.

## Running locally

Open `index.html` in a browser (or serve the directory). Outside the artifact host `window.claude` is missing, so the app renders with `conn = "offline"` and the "Can't reach the shared job data" banner; writes are refused. Real data and uploads only work on claude.ai.

## Architecture

Everything lives in one IIFE inside the `<script>` at the bottom of `index.html`, laid out in `/* ---------- Section ---------- */` blocks.

**Runtime capabilities** (requested in `connect()` via `window.claude.use(name)`):
- `db`: shared database. Four collections in `COLS`: `jobs`, `team`, `items`, `files`. Each is subscribed with `onSnapshot`, which replaces `state[col]` wholesale and triggers a render.
- `assets`: photo/document uploads (`ASSETS.upload` / `ASSETS.delete`). Files are served from `/_blob/<assetId>`. Images are shrunk client-side (`shrinkImage`) before upload; allowed MIME types are in `OK_TYPES`.
- `user`: viewer identity. `can("data.write") === false` sets `readOnly`; the viewer's first name is auto-matched to a team member to set `me`.

**Data model**
- `items` holds every list entry for every category; the `cat` field selects one of the categories in `CATS`. Each `CATS` entry defines that category's labels, number prefix (`pre`, used by `nextNum` to make e.g. `RFI-004`), the ordered `statuses`, which statuses count as `done`, and feature flags (`money`, `paid`, `dir`, `ns` no-show counter, `email`, `noDue`, `types`). Adding or changing a category is mostly a `CATS` edit; forms and lists read these flags.
- Status color comes from `tone()` using the `NEG` / `NEUTRAL` lists plus `done`.
- `files` records point at an asset id and are linked to a job, a category (`FILE_CATS`, which adds `general`), and optionally an item.
- Records seeded as examples carry `sample: true` (jobs use `sampleInfo`); "Clear sample data" removes only those, and editing a sample item strips the flag.

**Writes are optimistic.** `putDoc` / `delDoc` update `state` immediately, schedule a render, then call `write()`. `write()` serializes writes per doc key through `chains`, retries once on `unavailable`, and routes failures to `writeError`, which maps error codes to toasts (and flips to `readOnly` on permission errors). Always go through `putDoc`/`delDoc`, and check `guardWrite()` before user-initiated writes.

**Rendering** is string templates into `innerHTML`. `render()` calls every `render*` section; use `scheduleRender()` (rAF-coalesced) after state changes. Escape all user data with `esc()`. Clicks are handled by one delegated `document` listener keyed on `data-*` attributes (`data-tab`, `data-job`, `data-file`, `data-act`, ...). Modals are bottom sheets via `openSheet(html, onMount)` / `closeSheet()`. Destructive buttons use a tap-twice confirm (`armButton` or the `data-act` arm pattern).

**Browser storage** (`ls` wrapper, always try/catch) holds only per-device conveniences: UI state (`raybuilders.ui.v2`), who's using this phone (`raybuilders.me`). `raybuilders.pc.v1` is the legacy pre-sharing localStorage data; `localLeftovers()` / `importData()` offer to migrate it into the shared db once. The Settings sheet also offers JSON backup export/import (`{v:3, jobs, items, team, files}`).

**Styling**: CSS custom properties on `:root` (dark "job-site" theme only, `color-scheme: dark`), fonts Big Shoulders Display / Barlow / IBM Plex Mono. Phone first; the list goes two-column on desktop.

## Access

Team members get in by being invited from the artifact's Share menu as **Editor** and signing in to claude.ai. Viewers without write access see a view-only banner.
