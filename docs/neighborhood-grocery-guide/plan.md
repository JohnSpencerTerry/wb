# Plan: Neighborhood Grocery Guide

## Architecture
A standalone static web app, in its own repo, deployed on GitHub Pages (free) and embedded into the `wb` site's write-up via an `<iframe>`, the same pattern used for the [clinical guideline assistant](../../_writing/clinical-guideline-assistant.md).

Three pieces:
1. **Static frontend** — browse/search UI over a single JSON data file checked into the repo. No server, no build step beyond static hosting.
2. **Intake** — a Google Form (free), one response per submitted item/price/sentiment, landing in a Google Sheet. No custom backend code needed for intake.
3. **Moderation → publish pipeline** — the author reviews rows in the Sheet and marks the ones to approve. A small local script pulls approved rows from the Sheet's published CSV export, merges them into the repo's JSON data file, and the author commits/pushes to redeploy. Publishing is therefore a deliberate, manual step — nothing goes live until the author runs it.

```
Visitor → Google Form → Google Sheet (pending)
                              |
                    author marks "Approved"
                              |
                    sync script (local, on demand)
                              v
                    data/stores.json (in repo)
                              |
                        git push → GitHub Pages redeploy
```

## Stack
- **Frontend**: plain HTML/CSS/JS, no framework — matches how `wb` itself is built (see [writing.md](../../writing.md)'s tag-filter pattern) and makes it straightforward to visually match the blog's style guide, which a framework's default styling would fight against.
- **Data**: a single `data/stores.json`, no database. At MVP scale (10 stores) this is simpler than standing up any DB, free, and trivially version-controlled.
- **Intake**: Google Forms + Google Sheets. Free, no auth required for submitters, and gives the author a familiar review surface for free (satisfies the "approve before publish" requirement from `intent.md` with zero custom UI).
- **Sync script**: a small Node script (`scripts/sync-submissions.mjs`) using the Sheet's published-to-web CSV export URL — no Google API credentials needed, just a URL fetch and a CSV parse.
- **Hosting**: GitHub Pages on the new repo. Free, matches how the main site is presumably already hosted (custom domain via `CNAME` in `wb`), no server to run or pay for.

## Data / external dependencies
- **Google Form/Sheet**: created manually by the author (not scriptable meaningfully); the plan assumes one Form with fields for store, item/price OR general sentiment text, and a hidden/prefilled store ID param linked from each store card. The Sheet must stay unlisted (link-only, not indexed) since pending/unapproved rows sit there before review.
- **GitHub Pages**: free static hosting on the new repo.
- **No paid APIs.** No cost beyond the author's own data-entry time for the 10 seed stores.

## Data model (`data/stores.json`)
```json
{
  "stores": [
    {
      "id": "city-fresh-market",
      "name": "City Fresh Market",
      "address": "57 Kingsland Ave",
      "specialtyItems": ["produce"],
      "bestBuys": ["seasonal fruit"],
      "items": [
        { "name": "eggs", "category": "dairy", "price": 4.50, "approved": true }
      ],
      "sentiments": [
        { "text": "Good deals on meat.", "approved": true }
      ]
    }
  ]
}
```
- `items[].category` backs the category-fallback search.
- Economic score is **not stored** — computed at render time from `items[]` across all stores (see below), so it stays derived rather than hand-set, per `intent.md`.

## Search behavior
Client-side, over the loaded JSON, on every keystroke or submit:
1. **Exact match**: any store item whose `name` equals the query (case-insensitive) — ranked first.
2. **Category match**: if no exact match, any item whose `category` equals the query, or whose `name` contains it — ranked second.
3. **Sentiment match**: any store sentiment whose text contains the query — ranked last, visually marked as "mentioned in a note" rather than a priced result.

Each result groups by store and shows which tier it matched on, so the ranking is visible, not just implicit ordering.

## Economic score (v1 placeholder)
Per `intent.md`, the exact algorithm is deferred — v1 uses a simple, replaceable placeholder: for each item category, compute the median price across all stores' approved items in that category; a store's score is the share of its items priced at or below that category median. Isolated in one function (`computeEconomicScore(store, allStores)`) so the formula can change later without touching search or rendering.

## Task breakdown
1. Scaffold the new repo (`neighborhood-grocery-guide`): `index.html`, `assets/style.css`, `assets/app.js`, `data/stores.json`.
2. Pull the relevant design tokens from `wb`'s [assets/style.css](../../assets/style.css) (`--color-bg`, `--color-ink`, `--color-muted`, `--color-hairline`, `--font-serif` "Newsreader", `--font-sans` "Work Sans") into the new project's stylesheet so it reads as the same site.
3. Build the store directory/browse view: cards per store (name, address, specialty items, best buys, computed economic score), filterable by specialty using the same button-toggle pattern as `.media-filter-btn` in [writing.md](../../writing.md).
4. Build the item-first search bar and the three-tier ranked results (exact → category → sentiment), per the search behavior above.
5. Populate `data/stores.json` from the full candidate set: an OpenStreetMap Overpass query against the author's North Williamsburg polygon (`northbk.geojson`) for `shop=supermarket|grocery|greengrocer|butcher` (convenience and deli excluded as mislabeling noise), giving real name/address/website data for free — see `candidates.json` (50 candidates). Each candidate then gets a web-research enrichment pass grounded in real public sources (store site, public reviews) for specialty items/best buys/sentiments; item-level prices are deliberately left empty at launch and filled in by the crowdsourced submission flow rather than fabricated.
6. Implement `computeEconomicScore()` and wire it into both the directory and search result rendering.
7. Create the Google Form (store selector, item name, price, and/or free-text sentiment) and link it from each store card, prefilling the store field.
8. Write `scripts/sync-submissions.mjs`: fetch the Sheet's published CSV, filter to rows marked approved, append them into the matching store's `items`/`sentiments` in `data/stores.json`.
9. Deploy the repo to GitHub Pages; confirm the live URL renders correctly on its own.
10. Add the write-up post in `wb`'s `_writing/` (new article, via the `new-article` skill — flagged as a follow-up, not built in this skill) with an iframe embed pointing at the GitHub Pages URL, following `clinical-guideline-assistant.md`'s embed markup.

## Open risks / unknowns
- **Google Sheet exposure**: the sync script needs the Sheet reachable by URL (published CSV). Since pending/unapproved rows sit there too, the Sheet must stay unlisted (never linked from the public app) — a deliberate convention, not enforced by tooling, so worth double-checking before each publish.
- **Spam on the Form**: Google Forms has no built-in CAPTCHA on the free tier. Accepted for v1 given manual approval catches it before anything goes public; revisit if volume becomes a problem.
- **Economic score formula**: the v1 median-based placeholder may produce odd results with only 10 stores and sparse item data early on. Isolated behind one function so it's cheap to revise once real data exists.
- **Publish cadence**: since publishing is a manual script run + push, approved submissions won't appear until the author runs it — acceptable per the chosen static/rebuild flow, but worth the author knowing there's no auto-publish.
