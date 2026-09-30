# Changelog

Captures the iteration history from the Claude.ai chat this project was handed off from, for context on *why* things are the way they are.

## v1 — Initial build
- 25-book demo dataset assembled (popular books, last ~100 years), with hand-estimated spine thickness per book (see `docs/DATA_MODEL.md` for the formula).
- Built the core packing visualization: books rendered as colored spines on a 24" shelf, wrapping to additional shelves as needed.
- Six sort options implemented: title, author, year published, story setting, thickness, genre — plus a reverse-order toggle.
- "Story setting year" introduced as a distinct concept from publication year, including handling for books with no single real-world setting (grouped separately).
- Card-catalog visual design established (wood/brass/cream palette, serif + typewriter type pairing).
- Full catalog card list added below the shelf view, mirroring the same sort order.
- Published as a Claude.ai artifact (interactive HTML page).

## v2 — Inline editing
- Added the ability to tap/click any book and edit its **story-setting year and description** directly in the detail panel, with a Save action (and Enter-to-save).
- Editing re-sorts and re-packs the shelf immediately, since the edited value feeds directly into the "story setting" sort.
- Explicitly scoped as **in-memory only** — no persistence layer yet (see CLAUDE.md's "Likely next steps").

## v3 — Nonfiction titles added
- User asked for nonfiction representation in the example dataset.
- Removed 5 fiction titles (*The Alchemist*, *Gone Girl*, *Life of Pi*, *The Girl with the Dragon Tattoo*, *Lord of the Flies*) to make room while keeping the total at 25.
- Added 5 nonfiction titles: *Sapiens*, *The Diary of a Young Girl*, *In Cold Blood*, *Unbroken*, *A Brief History of Time* — spanning 4 new genres (History, True Crime, Biography, Science) each with a newly assigned color.
- Clarified in the UI copy (footer note, empty-setting group label, edit-field help text) that "story setting" means different things for fiction vs. nonfiction, and that a few titles genuinely have no single setting year (invented worlds *or* sweeping/non-chronological nonfiction).

## v4 — Repo handoff (this commit)
- Extracted the single HTML file into a proper repo structure (`src/index.html`) with `CLAUDE.md` and supporting docs, so work can continue in Claude Code instead of the original chat.
- No functional changes to the app itself in this step.
