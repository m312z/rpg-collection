# RPG Collection

A single-file, dependency-free web page for searching and browsing a tabletop
RPG collection that lives in a Google Sheet.

The page reads the sheet over the network on every load, so it is never out of
date and there is nothing to export, sync or rebuild. It is read-only by
construction: the page only ever issues a `GET`, and a published-to-web CSV
link cannot be written through.

No build step, no dependencies, no framework — `index.html` is the whole thing.

## Using it with your own sheet

1. In Google Sheets: **File → Share → Publish to web**.
2. In the left dropdown pick **the single tab** holding your collection — *not*
   "Entire document". Other tabs in your spreadsheet may contain things you
   would rather not publish; publishing one tab exposes only that tab.
3. In the right dropdown pick **Comma-separated values (.csv)** → **Publish**.
4. Open the page and paste the link into the box it offers.

The only required column is `Title`. Everything else is discovered.

### Pointing the page at a sheet

There are three ways, in precedence order:

| How | Scope | Use it for |
|---|---|---|
| `?csv=<link>` in the address | Travels with the link | **Sharing.** Anyone opening that URL sees that collection. Bookmark it. |
| Pasting into the page | Remembered in that browser only | Your own day-to-day use on one device. |
| `CONFIG.csvUrl` in `index.html` | Baked into the file | A private deployment where the page should just work for everyone. |

This repository ships with `CONFIG.csvUrl` empty on purpose, so a public repo
carries no link to anybody's sheet. The `?csv=` form gives you a shareable URL
without committing one.

## Hosting

Any static host works, and hosting is the point: browsers refuse to fetch the
CSV from a page opened directly off disk as a `file://` address, because Google
sends no CORS header to a null origin. The page detects that case and says so.

- **GitHub Pages** — Settings → Pages → deploy from branch.
- **Netlify / Cloudflare Pages** — drop the folder in.
- **Locally** — `python -m http.server 8777`, then
  `http://127.0.0.1:8777/index.html`. Serving it is what matters, not the port.

## Features

- Search across every text column, insensitive to accents and punctuation, so
  `mork borg` finds *MÖRK BORG* and `cyborg` finds *CY_BORG*
- Faceted filters with live result counts that update against the other filters
- Card and table views; sortable columns; grouping by any category or by decade
- A detail panel for each entry
- Light and dark themes, following the system setting
- Shareable URLs — search, filters, sort and grouping are all in the address
- Works down to phone width

## Columns are inferred, not configured

There is no column list in the code. Add a column to the sheet and it appears
in the UI on the next load, with the control chosen from the data in it:

| What the column holds | What you get |
|---|---|
| `TRUE` / `FALSE` | A Yes / No / Any toggle |
| Numbers | A min–max range filter and a sort option |
| Comma-separated values | Selectable tag chips |
| Short repeated labels | Chips if there are few, a dropdown if many |
| Sentences | Searchable, and shown in the detail panel |
| A URL | A link; image URLs render as a cover image |

The distinction between a category and free prose is drawn on phrase length
rather than cardinality, so a `Notes` or `Review` column stays searchable text
instead of collapsing into a useless dropdown of unique values.

## Data expectations

The header row is located by looking for `Title`, so leading title or spacer
rows above it are tolerated. Rows with an empty `Title` are skipped. The CSV
parser is RFC 4180 compliant — quoted fields, embedded commas and newlines,
doubled quotes and a UTF-8 BOM are all handled.

## Licence

MIT.
