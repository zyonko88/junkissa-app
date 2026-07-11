# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

昭和純喫茶めぐり手帖 — a Japanese-language, mobile-first web app for logging visits to Shōwa-era retro coffee shops (純喫茶) in Takamatsu, Kagawa. The entire application is a **single file: `index.html`** — inline CSS, inline ES5 JavaScript, and base64-encoded assets (PWA manifest, apple-touch-icon, cover photo) embedded as `data:` URIs. There are no other source files, no dependencies, no build step, no tests, and no linter.

## Development workflow

- Run it by opening `index.html` in a browser, or serve it statically: `python3 -m http.server 8000`.
- There is no build/lint/test command; verification means loading the page and exercising the flow you changed.
- App state persists in `localStorage` under `junkissaAppData_v1`. To see seed data or a fresh-install state after changing the data model, clear that key (or use an incognito window) — `loadData()` only seeds from `TAKAMATSU_DIRECTORY` when no saved data exists.

### Reading the file

`index.html` is ~1,084 lines but ~300 KB because of base64 image lines — reading it whole blows past token limits. Grep for the function or section you need, or read line ranges. Layout: `<style>` block at lines ~13–392, single `<script>` IIFE at ~397–1082.

## Architecture

Everything lives in one IIFE at the bottom of `index.html`:

- **State**: a single `data` object (`{ shops: [...] }`) loaded via `loadData()` and persisted with `saveData()` after every mutation. `TAKAMATSU_DIRECTORY` is the hardcoded seed/directory of real Takamatsu cafés.
- **Routing**: hash-based. `render()` parses `location.hash` and dispatches to a `render*` function; `hashchange` re-renders. Routes: `#/` (cover), `#/list`, `#/directory`, `#/add-shop`, `#/edit-shop/:id`, `#/shop/:id`, `#/shop/:id/add-visit`.
- **Rendering**: each `render*` function builds an HTML string, assigns it to `#app.innerHTML`, then wires event handlers by `getElementById`/`querySelectorAll`. All user-supplied text must go through `esc()` before interpolation into HTML.
- **Data model**:
  - Shop: `{ id, name, address, area, description, recommendedMenu: [], createdAt, visits: [], isSample }` (seeded shops get `id: "seed-N"` and show a サンプル tag until edited).
  - Visit: `{ id, date, companions, mood: {emoji, label}, memo, photo }` where `photo` is a canvas-downscaled JPEG data URL (quality 0.7) captured via `<input type="file" capture="environment">` or the photo library.
- **Extras** worth knowing before touching UI:
  - `mascotSvg(size, withBadge)` generates the inline-SVG mascot じょんこちゃん (a tanuki) used on the cover and headers.
  - A floating music button (`mountMusicButton`/`toggleMusic`) plays a looping Shōwa-style melody synthesized with the Web Audio API (`MELODY`/`NOTE_FREQ`/`playNote`) — no audio files.
  - Selecting a mood in the visit form speaks its label via `speechSynthesis` (`speakText`, `ja-JP`).
  - Shop detail links to Google Maps directions via `mapsUrl(address)`.

## Conventions

- Keep the app a single self-contained file: no external scripts, stylesheets, fonts, or image files. New assets go inline (SVG strings or base64 `data:` URIs).
- The JavaScript is deliberately plain ES5 (`var`, `function`, string concatenation, no modules). Match that style.
- All UI text is Japanese, in a nostalgic Shōwa tone. Styling uses the CSS custom properties defined on `:root` (`--brown-dark`, `--cream`, `--mustard`, `--red`, `--green`, `--ink`, …) with a serif Mincho font stack; reuse these rather than introducing new colors.
- Layout is mobile-first, constrained to `max-width: 520px` on `#app`, with sticky `.header-bar` and `.fab` patterns shared across views.
- If you change the shape of stored data, handle existing `junkissaAppData_v1` payloads (migrate or bump the storage key) — real devices already have saved data.
