# Data Model

## Schema

Each entry in the `books` array (in `src/index.html`, inside the `<script>` block) has this shape:

```js
{
  id: 1,                     // unique integer, stable across edits — used for lookups, not display order
  title: "The Great Gatsby",
  author: "F. Scott Fitzgerald",
  pubYear: 1925,              // year the book was published
  pages: 180,                 // page count of a representative edition
  thickness: 0.45,            // ESTIMATED spine width in inches — see formula below
  height: 7.9,                // book height in inches, for visual variety only (not used in shelf-width math)
  genre: "Classic Fiction",   // must match a key in GENRE_COLORS, or add a new one there too
  settingYear: 1922,          // number, or null if the book has no single setting year
  settingLabel: "1922"        // display string shown on cards — free text, user-editable in the UI
  // optional: dimSource: "Open Library" | "entered by hand" | ...  (where thickness came from)
}
```

`id`s currently in use: 1–4, 6–16, 18, 20–21, 24–30 (there are gaps — some early draft titles were removed during iteration; gaps are harmless, just don't reuse an id that might still be referenced elsewhere, e.g. in future persistence).

## Thickness estimation formula

There's no live lookup — thicknesses are hand-estimated:

```
thickness (in) ≈ pages / 460
```

...with a bump of roughly +0.15" to +0.3" added by eye for hardcovers or books known to use thicker stock (added directly to the stored `thickness` value, not computed at runtime). This is a rough real-world approximation (~460 pages per inch is typical for trade paperback stock) and is **explicitly caveated to the user in the app's footer** — actual thickness varies by printing/edition. If you add real dimension lookups later (see CLAUDE.md next steps), this formula and the caveat text should be removed/updated together.

## "Story setting" — what it means and how it's decided

`settingYear` / `settingLabel` capture **when the book's content takes place**, not when it was published. This was the main differentiating feature requested for this app (most library/cataloging tools only sort by publication date).

Rules of thumb used when assigning these during dataset creation:

- For a book with a clear, bounded setting, `settingYear` is that year (or the midpoint of a range), and `settingLabel` is the human-readable range/description.
- For **nonfiction**, "story setting" means the era the book's content focuses on — e.g. a biography or narrative history uses the period it covers, not the year it was written.
- `settingYear: null` is used for two distinct cases, both grouped together in the UI under a "no single date" label since there's no meaningful single year to sort by:
  1. **Invented worlds** with no real-world calendar mapping (e.g. Tolkien's Middle-earth).
  2. **Nonfiction that spans an enormous or unspecified timeframe** (e.g. *Sapiens* covers ~70,000 years; *A Brief History of Time* isn't narrative/chronological at all).
- Users can override any book's `settingYear`/`settingLabel` from the UI (tap the book → edit fields → Save). This only changes in-memory state; see CLAUDE.md's persistence note.

## Genre colors

Defined in the `GENRE_COLORS` object, one entry per genre currently in use. The legend and every spine/card color are driven entirely by this map — **only genres present in `GENRE_COLORS` will render correctly**, so adding a new genre to a book requires adding a matching color entry too. Colors were chosen by hand to stay visually distinct from each other within the wood/brass/cream palette (see `docs/DESIGN_DECISIONS.md` for the reasoning) — when adding a new genre, pick a new color deliberately rather than reusing something close to an existing one.

Current map:

| Genre | Hex | Used by (examples) |
|---|---|---|
| Classic Fiction | `#7A3B2E` | The Great Gatsby, Catcher in the Rye |
| Dystopian | `#45566B` | 1984, Brave New World |
| Fantasy | `#3F5B3D` | The Hobbit, Harry Potter |
| Historical Fiction | `#A9782F` | Beloved, The Book Thief |
| Magical Realism | `#6B3F63` | One Hundred Years of Solitude |
| Thriller | `#3A3A3A` | The Da Vinci Code |
| Post-Apocalyptic | `#8C4A2F` | The Road |
| Sci-Fi | `#2C3E63` | Slaughterhouse-Five, The Martian |
| Memoir | `#6B4A34` | Educated, The Diary of a Young Girl |
| History | `#4A6FA5` | Sapiens |
| True Crime | `#5B1F2B` | In Cold Blood |
| Biography | `#6E5B3E` | Unbroken |
| Science | `#3E6B5C` | A Brief History of Time |

## Current dataset (25 books)

| # | Title | Author | Pub. | Setting | Genre | Pages | Thickness |
|---|---|---|---|---|---|---|---|
| 1 | The Great Gatsby | F. Scott Fitzgerald | 1925 | 1922 | Classic Fiction | 180 | 0.45" |
| 2 | To Kill a Mockingbird | Harper Lee | 1960 | 1933–1935 | Classic Fiction | 281 | 0.65" |
| 3 | 1984 | George Orwell | 1949 | 1984 | Dystopian | 328 | 0.75" |
| 4 | The Catcher in the Rye | J.D. Salinger | 1951 | 1949–1950 | Classic Fiction | 277 | 0.65" |
| 6 | The Hobbit | J.R.R. Tolkien | 1937 | Third Age (no year) | Fantasy | 310 | 0.90" |
| 7 | The Lord of the Rings | J.R.R. Tolkien | 1954 | Third Age (no year) | Fantasy | 1178 | 2.90" |
| 8 | One Hundred Years of Solitude | Gabriel García Márquez | 1967 | 1850s–1920s | Magical Realism | 417 | 0.95" |
| 9 | Beloved | Toni Morrison | 1987 | 1873 | Historical Fiction | 324 | 0.75" |
| 10 | The Grapes of Wrath | John Steinbeck | 1939 | 1930s | Classic Fiction | 464 | 1.05" |
| 11 | Fahrenheit 451 | Ray Bradbury | 1953 | ~2049 | Dystopian | 256 | 0.60" |
| 12 | Brave New World | Aldous Huxley | 1932 | 2540 | Dystopian | 311 | 0.70" |
| 13 | The Handmaid's Tale | Margaret Atwood | 1985 | Early 2000s | Dystopian | 311 | 0.70" |
| 14 | Harry Potter and the Sorcerer's Stone | J.K. Rowling | 1997 | 1991 | Fantasy | 309 | 0.95" |
| 15 | The Da Vinci Code | Dan Brown | 2003 | 2003 | Thriller | 454 | 1.15" |
| 16 | The Kite Runner | Khaled Hosseini | 2003 | 1975–2001 | Historical Fiction | 371 | 0.85" |
| 18 | The Book Thief | Markus Zusak | 2005 | 1939–1945 | Historical Fiction | 552 | 1.40" |
| 20 | The Road | Cormac McCarthy | 2006 | unspecified (no year) | Post-Apocalyptic | 287 | 0.65" |
| 21 | Slaughterhouse-Five | Kurt Vonnegut | 1969 | 1945 | Sci-Fi | 275 | 0.65" |
| 24 | Educated | Tara Westover | 2018 | 1990s–2010s | Memoir | 334 | 0.80" |
| 25 | The Martian | Andy Weir | 2011 | ~2035 | Sci-Fi | 369 | 0.85" |
| 26 | Sapiens: A Brief History of Humankind | Yuval Noah Harari | 2015 | spans ~70,000 BCE–present (no year) | History | 443 | 1.00" |
| 27 | The Diary of a Young Girl | Anne Frank | 1952 | 1942–1944 | Memoir | 283 | 0.65" |
| 28 | In Cold Blood | Truman Capote | 1966 | 1959–1965 | True Crime | 343 | 0.75" |
| 29 | Unbroken | Laura Hillenbrand | 2010 | 1917–1945 (WWII focus) | Biography | 528 | 1.20" |
| 30 | A Brief History of Time | Stephen Hawking | 1988 | N/A — not narrative (no year) | Science | 212 | 0.50" |

### History of changes to this dataset

Five fiction titles from the original draft were swapped out for nonfiction on request: *The Alchemist*, *Gone Girl*, *Life of Pi*, *The Girl with the Dragon Tattoo*, and *Lord of the Flies* were removed (mostly to reduce redundancy with other thrillers/classics already present); *Sapiens*, *The Diary of a Young Girl*, *In Cold Blood*, *Unbroken*, and *A Brief History of Time* were added in their place, bringing the nonfiction count to 8 of 25 (including *Educated*, which was in the original set).


## Custom genres

Genres added through the UI get the next color from `EXTRA_COLORS` in `src/index.html` (persisted in `customColors`), falling back to a hashed HSL color when those run out.
