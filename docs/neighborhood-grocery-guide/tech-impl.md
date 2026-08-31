# Technical Implementation: Neighborhood Grocery Guide

This documents the technical decisions actually made while building the project, as they
happened — including the ones that revised `plan.md`. See `intent.md` for the product framing and
`plan.md` for the original architecture plan this was built from.

Repo: [neighborhood-grocery-guide](https://github.com/JohnSpencerTerry/neighborhood-grocery-guide)
(public). Live: https://johnspencerterry.github.io/neighborhood-grocery-guide/

## Architecture

Static HTML/CSS/JS, no framework, no bundler, no server, no database — a deliberate choice to
keep hosting free and the codebase small enough to hand-maintain. The one piece of "build" is a
GitHub Actions workflow that deploys the site and injects a cache-busting version string (see
Caching below); there's still no compile/transpile/bundle step for the source itself.

Three moving pieces:
1. **Frontend** (`index.html`, `assets/style.css`, `assets/app.js`) — fetches `data/stores.json`
   and renders everything client-side.
2. **Intake** — a public Google Form, responses land in a Google Sheet.
3. **Moderation → publish** — a human (the author) marks rows `Approved` in the Sheet, then runs
   `scripts/sync-submissions.mjs` to merge approved rows into `data/stores.json`, then commits and
   pushes. Nothing goes live automatically; publishing is a deliberate manual step.

## Sourcing the store data

The original plan assumed a hand-curated 10 stores. That changed mid-build: the author drew a
North Williamsburg polygon in [geojson.io](https://geojson.io) (`northbk.geojson`), and the store
list was generated from it instead of picked by hand.

- **OpenStreetMap Overpass API** (`overpass-api.de/api/interpreter`), queried with a `poly:`
  filter built from the polygon's coordinates (GeoJSON is `[lon, lat]`; Overpass wants
  `"lat lon lat lon ..."`, so the coordinates get swapped before querying).
- Filtered to `shop` tags `supermarket`, `grocery`, `greengrocer`, `butcher`. Excluded
  `convenience` (117 of 195 raw results) and `deli` (28) as mislabeling noise — the exact "delis
  tagged as grocery stores" problem the guide exists to cut through, so including them would
  undermine the product.
- Result: 50 candidates with real name/address/website data, saved to `candidates.json`, entirely
  free and requiring no API key.

## Enrichment (grounding, not fabricating)

Each of the 50 candidates needed specialty items, best buys, and review-based sentiment — none of
which OSM provides. Three background research agents ran in parallel (one per ~17-store batch),
each under a hard constraint: **every specialty item, best buy, and sentiment must trace back to
something actually found** (the store's own site, a real review snippet with a cited source URL).
Empty arrays were the expected, correct output when nothing concrete surfaced — not a prompt to
guess from the store name.

This caught real problems a purely automated pipeline would have missed or silently gotten wrong:
- **Closed businesses**: Midoriya, To The World Farm, and Mr. Pina were all confirmed closed via
  multiple sources and excluded.
- **Mistagged data**: Fresh My Day is a juice bar, not a grocery store (OSM mistagged it
  `supermarket`); Mr. Mushroom is a wholesale supplier with no retail storefront. Both excluded.
- **Stale/wrong addresses**: The Meat Hook's OSM address was its closed 2009-2016 location; Lidl's
  OSM record pointed at a Manhattan store, not the Williamsburg location that had opened five days
  before the research ran. Both corrected using independently-verified sources.
- **A real duplicate**: two different OSM nodes for The Meat Hook were independently corrected by
  two different research batches to the *same* real current address — not caught until visually
  testing the rendered page (see Testing below), then merged into one entry.

The full audit trail — every excluded/corrected store and why, with source citations — is in
`enrichment-audit.json`. It's kept separately from `data/stores.json` (the audit file has
provenance/notes fields; the production file doesn't) so the shipped data model stays lean.

Final count: **43 stores** (50 candidates minus 6 excluded, minus 1 duplicate merge).

## Data model (`data/stores.json`)

```json
{
  "id": "kebab-case-slug",
  "name": "Store Name",
  "address": "123 Example St, Brooklyn, NY 11211",
  "lat": 40.71, "lon": -73.95,
  "filterCategory": "Butcher & Meat",
  "specialtyItems": ["..."],
  "bestBuys": ["..."],
  "website": "https://... (optional)",
  "flyerUrl": "https://... (optional)",
  "items": [{ "name": "eggs", "category": "dairy", "price": 4.5, "approved": true }],
  "sentiments": [{ "text": "...", "source": "https://...", "approved": true }]
}
```

- `items[]` starts **empty for every store, by design**. Prices are the one thing genuinely hard
  to source in bulk without scraping something fragile or against terms of service — so instead of
  faking them, v1 launches with real stores/notes and lets the crowdsourced submission flow (see
  below) be the actual source of price data, as originally intended in `intent.md`.
  - Real-world validation: the author submitted a test item through the live form, approved it,
    ran the sync script, and confirmed it appeared on the live site — full pipeline, not just a
    simulated one — then it was removed since it wasn't real data.
- Google Maps links are **generated at render time** from name + address (`mapsUrl()` in
  `app.js`), not stored — one less field to keep in sync.
- `lat`/`lon` were added after the fact, joined from `candidates.json` by OSM id, to support the
  distance-sort feature (see UI below) — not part of the original plan.

### `filterCategory`: collapsing raw tags into a usable filter set

The first version used each store's raw `specialtyItems` strings as filter buttons directly. With
43 stores contributing free-text phrases, that produced **~60 individual filter buttons** — an
unusable wall of tags (caught by visual inspection of the live site, not by the design process).

Fixed by adding a small algorithmically-derived taxonomy — `filterCategory` — computed once from
each store's `shopType` (from the audit data) and keyword matches in its `specialtyItems`:
`Supermarket & General`, `Butcher & Meat`, `Produce`, `Kosher`, `Natural & Organic`,
`International & Ethnic`. This field is a distinct concept from `item.category` (the free-text
"dairy"/"produce"/etc. used for item-price search) — the two were briefly conflated (a
`store.category` field used for both filtering and leaking into item search results) and split
apart once the bug surfaced. The full, specific `specialtyItems` list still shows on a store's
detail view; it's just not used for directory-wide filtering anymore.

## Search

Client-side, three-tier ranked, no backend:
1. **Exact match** — item name equals the query (case-insensitive).
2. **Category match** — item's `category` field matches, or the item name contains the query.
3. **Sentiment match** — a submitted/curated sentiment's text contains the query, ranked last and
   visually marked as "mentioned in a note" rather than a priced result.

`filterCategory` is deliberately **not** part of search matching — it's a directory-filter concept
only. This was a bug initially (a search-side branch matched on it and leaked unrelated specialty
items into "category match" results); removed once caught by a targeted logic test.

## Economic/value score

Computed at render time (`computeEconomicScore()`), not stored: for each item category, the median
price across all stores' approved items in that category; a store's score is the percentage of its
own items priced at or below that median. Deliberately a placeholder — the real algorithm was
explicitly deferred in `intent.md` pending real submitted price data — isolated in one function so
it can be swapped later without touching search or rendering. Returns `null` (score not shown)
until a store has at least one approved item, which is every store at launch, since `items[]`
starts empty by design.

## UI: compact list + detail modal, not per-store pages

Originally planned as full cards showing everything inline. Revised after the tag-explosion issue
above prompted a broader UI pass: store rows are now compact (name, address, category, score,
distance), and clicking one (or pressing Enter/Space — rows are keyboard-focusable with
`role="button"`) opens a modal with full detail (specialty items, best buys, known prices,
sentiments, links).

Modal over separate per-store pages was a deliberate call to preserve the static/no-build-step
architecture: generating 43 individual HTML pages would need either a templating build step or
43 hand-maintained files, neither of which fit a site whose whole point is being cheap to run and
easy to keep updating.

## Location-based sorting

`navigator.geolocation.getCurrentPosition()`, wrapped in a promise (`getUserLocation()`) that
resolves to `null` on denial/timeout/unsupported rather than rejecting, so the UI always has a
defined fallback path. Distance uses the haversine formula (`milesBetween()`). When location is
unavailable, stores sort alphabetically instead, with a visible status line
(`#location-status`) telling the user which mode is active — not a silent fallback.

## Submissions: Google Form + Sheet, no login

Chosen over a custom backend specifically to keep the "no login to submit, but author approves
before anything's public" requirement from `intent.md` cheap to build:
- The Form's **Store ID** field is prefilled per store via a URL param
  (`SUBMIT_FORM_STORE_PARAM = "entry.816889128"`), resolved by fetching the form's HTML and
  reading its `FB_PUBLIC_LOAD_DATA_` embedded field metadata rather than guessing entry IDs.
- The Sheet has a manually-added **Approved** column (`TRUE`/`FALSE`) — the actual moderation
  gate. The Sheet itself is deliberately kept **unlisted** (published-to-web CSV export exists,
  but the Sheet is never linked from the app) since unapproved submissions sit there until
  reviewed.
- `scripts/sync-submissions.mjs` has **zero npm dependencies** — a hand-written CSV parser
  (handles quoted fields/embedded commas) rather than pulling in a library for something this
  small. It fetches the Sheet's published CSV, filters to `Approved === "TRUE"`, and merges new
  item/sentiment rows into the matching store by id, de-duplicating against what's already there.
- The Sheet's real header row had a stray leading space on `" Sentiment Text"` that broke an
  exact-match column lookup — found by testing against the actual sheet, not assumed from the
  schema on paper. Fixed by trimming every parsed cell.
- `SHEET_CSV_URL` lives in a **gitignored `.env`** (loaded by a ~15-line hand-written parser, no
  `dotenv` dependency), with `.env.example` committed as the template. This matters because the
  Sheet must stay unlisted — committing its URL into the public repo would defeat that, even
  though the URL itself isn't secret in the traditional sense (anyone with the link can read it).

## Deploy & caching

Two caching problems surfaced after the site was already live with real data, both found by the
author noticing stale content rather than by anticipating them upfront:

1. **Stale data.** `data/stores.json` changes independently of any deploy (via the submission sync
   script) and was being cached by the browser's default HTTP caching. Fixed by fetching it with
   `cache: "no-store"` — always revalidated, appropriate for a small JSON file at this traffic
   level.
2. **Stale assets.** GitHub Pages caches `style.css`/`app.js`, so a code change wasn't guaranteed
   to show up for a returning visitor. First fix was a manual `?v=N` query string bumped by hand;
   replaced with a **GitHub Actions deploy workflow** (`.github/workflows/deploy.yml`) that copies
   the repo to a build directory and `sed`-injects `?v=<short commit SHA>` into the deployed
   `index.html` automatically on every push — the source `index.html` in the repo stays plain, so
   there's nothing to remember to bump. This required switching GitHub Pages from legacy
   branch-based deployment to Actions-based deployment (`build_type: "workflow"` via the GitHub
   API), since a workflow-driven deploy needs the newer Pages deployment model.

## Testing approach

No test framework — the project is small enough that targeted, throwaway verification scripts
were used at each step instead:
- **Syntax**: `node --check assets/app.js` after every edit.
- **Logic**: slicing `app.js`'s source to just the pure functions (`computeEconomicScore`,
  `search`, `mapsUrl`, `milesBetween`, `sortStores`) via string slicing + `eval`, then exercising
  them against the real `data/stores.json` in a throwaway Node script — no browser needed to catch
  logic bugs like the `filterCategory`/`item.category` conflation.
- **Visual**: a temporary local Playwright install (removed after use, never committed) drove a
  real headless Chromium against a local static server, screenshotted the directory and an open
  modal, and checked the console for errors. This is what actually caught the duplicate Meat Hook
  row — a bug invisible to any of the logic-level tests, since both entries were individually
  "correct," just duplicated.
- **Pipeline**: the sync script was tested three ways before being trusted — a dry run against the
  real (empty) sheet, a fully simulated run with a stubbed `fetch()` and fake CSV rows (including
  one deliberately unapproved row, to confirm it's skipped), and finally a real submission through
  the live form, approved and synced for real.

## Cost

$0. GitHub Pages (static hosting) + Google Forms/Sheets (intake + moderation UI) + GitHub Actions
(free tier, well within limits for a low-traffic static deploy). No paid APIs; OpenStreetMap's
Overpass API and the enrichment research (public web sources) were also free.

## Still open

- **Shop Fair**'s address is only loosely matched — the chain has multiple same-name Brooklyn
  locations, and this one wasn't independently confirmed.
- **Lidl**'s Williamsburg address was inferred from press coverage of a days-old store opening; no
  store-specific URL exists for it yet.
- The economic-score algorithm is still the v1 placeholder — real refinement needs real submitted
  price data, which the site doesn't have much of yet.
- The write-up post (iframe-embedded in `wb`, per `intent.md`'s Delivery section) hasn't been
  drafted — planned as a follow-up via the `new-article` skill once the site's been used enough to
  write about honestly.
