# Intent: Neighborhood Grocery Guide

## Problem
In a neighborhood like Williamsburg, it's genuinely hard to tell which corner "grocery store" is an actual grocery store versus a bodega/deli mislabeled as one, and which of the real grocery options offer good value versus a poor price-to-quality ratio. There's no single place to see, at a glance, where to buy a specific item at a fair price, or which stores are known for particular specialties. Today this knowledge lives scattered across Google Maps saves, word of mouth, and trial and error.

## Who it's for
Primarily the author, as a personal reference tool — but built to be genuinely useful to other Williamsburg residents browsing the site, not just a private list. Designed as a concept that could be adapted to other neighborhoods later, if someone assembled the data for that area (multi-neighborhood support is a future extension, not part of this build).

## What it does
A searchable, filterable guide to grocery stores in a neighborhood. The core interaction is item-first: a visitor searches for something they want to buy (e.g. "eggs," "good produce," "cheap meat"). If there's an exact item match, that surfaces first; if not, the search falls back to matching on category. Search results also surface stores based on general sentiment notes people have left about them (e.g. "this store has good deals on meat"), ranked below exact item matches since they're less precise.

Each store has a profile: name, address, a derived economic score (a value-for-money signal computed from the item/price data collected about it, rather than a number the author assigns by hand), specialty items, and "best buys."

Anyone can contribute to a store's data without creating an account — adding an item, a price, or a general sentiment about a store. Because submissions are open and anonymous, every submission is queued for the author's approval before it appears publicly, to keep out spam or bad-faith entries.

## User stories
- As a neighborhood resident, I want to search for an item and see which nearby stores carry it and at what price, so that I can decide where to shop without guessing.
- As a visitor unfamiliar with the area, I want to browse stores by specialty or economic score, so that I can find good-value options without already knowing the neighborhood.
- As someone who just had a good (or bad) experience at a store, I want to add an item price or a general note about it without signing up for anything, so that the guide stays current with minimal friction.
- As the site owner, I want to review and approve submissions before they go live, so that the guide doesn't get polluted with spam or malicious entries.

## Success criteria
- The guide launches with every real grocery-type store (supermarket, grocery, greengrocer, butcher — mislabeled convenience stores and delis excluded) within a North Williamsburg polygon the author defined, sourced from OpenStreetMap and enriched with real, grounded public data (name, address, specialty items, best buys) rather than a small hand-picked set.
- A visitor can search for an item and get a ranked result: exact item matches first, category matches next, sentiment-based matches last.
- A visitor can submit an item, price, or sentiment for a store without logging in, and the author can review and approve/reject it before it's visible to others.
- The store's economic score reflects submitted price data rather than a manually assigned number.

## Out of scope (for now)
- Support for more than one neighborhood in this build — the concept should be designed so another neighborhood's data could plug in later, but Williamsburg is the only neighborhood shipped.
- The exact scoring algorithm for the derived economic score — the mechanism (derived from submitted price/item data) is set now; the precise formula can be refined after launch.
- User accounts or login of any kind, for browsing or for contributing.
- Automated spam/abuse detection beyond manual approval — moderation is a manual review step by the author.

## Delivery
- Write Up (site article introducing the guide).
- Live Demo — a live page on the site (matches the Outputs from `ideas.md`).
