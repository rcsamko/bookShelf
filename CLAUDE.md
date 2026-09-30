# CLAUDE.md — The 24-Inch Shelf

This file gives Claude Code the context to pick up this project where a prior chat conversation left off. Read this first, then `docs/DATA_MODEL.md` and `docs/DESIGN_DECISIONS.md` for detail.

## What this is

A single-page web app that helps someone physically arrange books on a real bookshelf. It ships with a demo dataset of 25 well-known books (fiction + nonfiction, spanning ~1925–2018) and lets the user:

1. Sort the collection by title, author, year published, **story setting** (the year/era the book's content takes place in, not publication year), thickness, or genre.
2. See the books packed left-to-right onto a **24-inch-wide shelf**, wrapping onto additional shelf rows as needed, using real (estimated) spine thickness in inches.
3. Click/tap any book to see full details, and **edit its "story setting" year and description** inline — this immediately re-sorts and re-packs the shelf when sorted by setting.

It was originally built and iterated on as a single published HTML artifact in a Claude.ai chat. This repo is a clean handoff point to continue the work in Claude Code.

## Current state

- **Fully working**. Two self-contained pages: `src/index.html` (the shelf) and `src/settings.html` (bookcase width/height). They share state through the same `localStorage` key; the settings page rewrites only `shelfW`/`shelfH` and keeps every other stored field. The shelf page reloads on Back (`pageshow`) to pick up changes.
- No build step, no package.json, no dependencies to install. Just open the file in a browser.
- The only external resources are two Google Fonts (loaded via `<link>`): `Source Serif 4` and `Courier Prime`.
- App state lives in plain JS variables and is persisted to `localStorage` (see below). Books can carry an optional `dimSource` string recording where a looked-up thickness came from.

## Branch flow

Three long-lived branches: `dev` → `test` → `prod`.

- **`dev`** — active development. All new work is built and iterated here (feature branches, if used, merge into `dev`).
- **`test`** — promoted from `dev` for verification before release. No direct feature work; only fixes needed to pass verification (make them in `dev` and re-promote where possible).
- **`prod`** — released, stable version. Only updated by promoting from `test`.

Promotion is one-directional: merge `dev` into `test`, then `test` into `prod`. Never commit straight to `test` or `prod`, and never promote skipping a stage.

## Tech stack & constraints (please preserve unless asked to change)

- **Vanilla HTML/CSS/JS only.** No React, no build tooling, no npm dependencies. This was a deliberate choice to keep it a single portable file.
- **Self-contained pages.** All CSS and JS are inlined in each HTML file (`src/index.html`, `src/settings.html`); the settings page duplicates the `:root` color tokens, so keep them in sync. The only allowed external calls are the two Google Fonts stylesheet links and the on-demand book-dimension lookups (Open Library, then Google Books) made from `lookupDimensions()` when the user clicks "Look up real dimensions".
- **Persistence uses `localStorage`** (key `shelf24.v1`: book list, `shelfW`/`shelfH`, `groupBy`, `sortChain`, `nextId`, `customColors`; older saves with `sort`/`reversed` are migrated), saved on every `render()` and loaded at startup by `loadState()`. All reads/writes are in try/catch, and saved data is validated (`validBook`) before use; corrupt or missing data falls back to the built-in dataset. A footer link resets to defaults. If you change the book schema, bump the key version or migrate.
- **Responsive, mobile-first.** The shelf visualization computes its px-per-inch scale from the actual rendered container width (see `render()` in the `<script>` block), so it works from ~320px phone screens up through desktop. Preserve this approach rather than hardcoding pixel widths.
- In dark mode `--wood-dark` and `--paper` are both near-black: never pair them as background/text. Use `--ink` on `--paper` (inverted) for filled buttons/tags.
- **Light/dark mode via CSS variables**, keyed off `prefers-color-scheme` and an optional `data-theme` attribute override. Any new UI should use the existing CSS custom properties (`--paper`, `--ink`, `--brass`, `--wood-*`, `--card-bg`, etc.) rather than hardcoded colors.
- **Safe-area aware**: `env(safe-area-inset-*)` padding is already set up on `:root`; keep this if you add fixed/sticky elements.

## Visual/design language

"Library card catalog" aesthetic — warm wood tones for the shelves, cream paper background, brass accents, a serif display font for headings/titles and a typewriter font for metadata/labels (evoking actual catalog cards, not decorative monospace). Full rationale and the color palette are documented in `docs/DESIGN_DECISIONS.md` — read it before changing colors or fonts so new additions stay consistent with the existing system (e.g., new genres need a new, sufficiently distinct color assigned deliberately, not a random pick).

## Data model

Each book is a plain JS object with: `id`, `title`, `author`, `pubYear`, `pages`, `thickness` (inches, estimated), `height` (inches, for visual variety only — not used in the packing math), `genre`, `settingYear` (number or `null`), `settingLabel` (display string). Full schema notes, the thickness-estimation formula, and the current 25-book dataset (with per-book rationale for ambiguous setting years) are in `docs/DATA_MODEL.md`. **Read that file before adding/editing books** so new entries follow the same conventions.

## Core logic (all in the `<script>` block of `src/index.html`)

- `packBooks(arr, width, keyFn)` — greedy bin-packing: walks the (sorted) array in order and starts a new shelf whenever the next book would exceed the user-set `shelfW` (default 24") or, when grouping, the group changes. This is intentionally *not* an optimal packing algorithm — it preserves the chosen sort order on the shelf, which is the point of the tool (a real person shelves books in a chosen order, not in whatever order minimizes empty space).
- `sortChain` — ordered list of `{id, dir}`; users drag "tags" (pointer events, plus keyboard) to build a nested sort: tag 1 first, ties broken by tag 2, etc. `compareBooks` applies it. Each `SORTS` comparator must look at ONE field only (no built-in tie-breaks) so chaining works. `groupBy` (none / genre "subject" / author) sorts by group first and puts each group on its own shelf.
- `shelfH` is a clearance height: taller books are flagged (`toolong`), not rejected. Add books with the form (`addBook`, id from `nextId`, never reused); remove via detail panel or card ×. New genres get the next color in `EXTRA_COLORS`.
- `SORTS` — array of `{id, label, fn}` comparator definitions. Title sort ignores leading "The/A/An"; author sort uses last name. Setting sort pushes `settingYear === null` to the end and groups them under a "no single date" label in the catalog list.
- `render()` — re-sorts, re-packs, and redraws both the shelf visualization and the catalog card grid. Called on sort-button click, reverse toggle, book edit save, and window resize (debounced).
- `showDetail(book)` / `saveSetting()` — the click-to-view / edit-in-place flow for a single book's setting year and label.

## Likely next steps (not yet built — good candidates for Claude Code to tackle)

Roughly in order of likely value:

1. ~~**Persistence.**~~ Done in `dev`.
2. ~~**Add/remove books via UI**~~ Done in `dev`.
3. ~~Configurable shelf width and height~~ Done in `dev`. Still open: a fixed number of shelves / different widths per shelf.
4. ~~**Real dimensions lookup**~~ Done in `dev` for existing books (detail-panel button; Open Library editions' `physical_dimensions`, then Google Books, then a page-count estimate). Also available in the add-book form (title/author; no ISBN yet). Original note: optionally look up actual trim size / page count via a books API (e.g. Open Library or Google Books) instead of the hand-estimated thickness formula, when the user adds a new book by title/ISBN.
5. **Drag-to-reorder** on the shelf itself as an alternative to picking a sort — lets someone do a custom arrangement and see it "count" against the 24".
6. **Export/share** — turn the current shelf layout into a shareable image or printable list.
7. **Unit tests** for `packBooks` and the sort comparators, since those are the parts most likely to silently break during future edits.

None of these were requested yet — confirm scope with the user before starting a large one.

## Things to *not* change without asking

- The no-build-step, plain-files architecture (each page self-contained; `settings.html` is the one approved second page).
- The card-catalog visual identity (colors/fonts) — see `docs/DESIGN_DECISIONS.md`.
- The fact that shelf packing follows sort order rather than optimizing for minimum shelves — that's a deliberate, discussed choice, not an oversight.
