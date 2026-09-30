# Design Decisions

## Concept: "library card catalog," not generic bookshelf clip-art

The visual language is built around a real card catalog / physical library aesthetic rather than a generic "cozy bookshelf" look (deliberately avoided the common cream + terracotta template look). Two things carry the theme:

1. The shelf itself — rendered as an actual wood plank (CSS gradient) with brass shelf-pin dots at each end, drop shadow beneath it, books as colored spine blocks sitting on top.
2. The catalog cards below — styled like physical library index cards, with a dashed rule under the title, an "accession number" (`No. 01`, `No. 02`...) in the corner, and metadata in a typewriter font, because that's literally how card-catalog entries were typed.

## Typography

- **Source Serif 4** — headings, book titles (including the vertical spine labels), card titles. A serif was chosen to feel bookish/literary rather than app-generic.
- **Courier Prime** — all metadata, labels, stats, sort-control microcopy, catalog card details. This is a *thematic* choice, not decoration-for-its-own-sake: it's meant to evoke a typewritten catalog card. Keep this distinction in mind if extending the UI — don't reach for monospace as a generic "data" signifier elsewhere in the app; it should stay tied to the catalog-card motif.

Both are loaded via a single Google Fonts `<link>` in `<head>`, with real fallback stacks (`Georgia, 'Times New Roman', serif` and `'Courier New', monospace`) in the CSS `font-family` declarations.

## Color system

CSS custom properties on `:root`, with a dark-mode override block (both a `prefers-color-scheme: dark` media query and an explicit `[data-theme="dark"]` attribute selector, so a future theme toggle can force either mode):

| Token | Light | Dark | Used for |
|---|---|---|---|
| `--wood-dark` / `--wood-mid` / `--wood-light` / `--wood-edge` | browns | near-black browns | the shelf plank gradient |
| `--paper` | warm cream `#F1E9D8` | near-black `#1B1712` | page background |
| `--ink` / `--ink-soft` | dark brown/black, muted brown | cream, muted cream | primary/secondary text |
| `--brass` / `--brass-light` | gold tones | same | accent lines, active states, shelf-pin dots, focus rings |
| `--card-bg` / `--card-line` | off-white / tan | dark brown / muted brown | catalog card + detail panel surfaces |

**Genre colors are separate** from this system — they live in the `GENRE_COLORS` JS object (see `docs/DATA_MODEL.md`), not as CSS variables, because they're assigned per-book at render time via inline styles. They were manually chosen to (a) all read clearly with cream spine-label text on top, and (b) stay visually distinguishable from each other. When adding a genre, eyeball it against the existing palette for this reason rather than picking an arbitrary hex value.

## Sort behavior details worth knowing

- **Title sort** strips a leading "The", "A", or "An" before comparing — library-catalog convention — so "The Road" sorts under R, not T.
- **Author sort** uses last name only (splits on whitespace, takes the final token). Doesn't handle suffixes (Jr., III) or multi-word last names specially — fine for the current 25-book dataset, worth revisiting if the dataset grows.
- **Story-setting sort** pushes `settingYear === null` entries to the end (treated as `Infinity`), and within that null group falls back to title order. The catalog list (not the shelf itself) inserts a one-time divider label when it crosses from dated to undated entries, so the user can see the grouping explicitly. The shelf visualization does *not* insert any visual break between shelves for this — a shelf is just a shelf.
- **Reverse toggle** is a simple `array.reverse()` applied after whichever comparator ran — it's a generic flip, not a per-sort "descending" mode, which keeps the UI to one control instead of doubling the button count.

## Shelf-packing behavior (intentional, not a bug)

`packBooks()` is first-fit-in-order, not an optimal bin-packing algorithm. It never reorders books to minimize the number of shelves or wasted space — it walks the array in whatever order the current sort produced and starts a new shelf only when the next book would overflow the current one. This mirrors how someone actually shelves a physical collection: they pick an order (alphabetical, chronological, whatever) and place books in that order, accepting some wasted space at the end of each shelf, rather than rearranging for a tighter pack. If this ever needs to change (e.g. an "optimize" mode), it should be additive — a new option, not a replacement for the default behavior.

## Responsive scaling approach

Rather than fixed pixel-per-inch constants, `render()` measures the actual available container width at render time and derives `scale = containerWidth / 24`, applied to both book width (`thickness * scale`) and height (`height * scale * 0.62`, an empirical factor chosen so spines look proportionate rather than absurdly tall). This keeps the 24" shelf visually accurate at any viewport width, phone through desktop, and re-runs on a debounced `resize` listener.
