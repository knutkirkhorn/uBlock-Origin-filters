---
name: ublock-filters
description: Authors uBlock Origin static filters for blocking ads, sponsored content, trackers, and cookie-consent popups, and for hiding or restyling page elements (including form inputs). Use when editing filters.txt, writing My filters rules, or when the user mentions uBlock Origin, uBO, ad blocking, cosmetic filters, element hiding, or pastes HTML of an ad to block.
---

# uBlock Origin filters

References: [Static filter syntax](https://github.com/gorhill/uBlock/wiki/Static-filter-syntax), [Procedural cosmetic filters](https://github.com/gorhill/uBlock/wiki/Procedural-cosmetic-filters), [Scriptlets](https://github.com/gorhill/uBlock/wiki/Resources-Library).

## Workflow

1. **Pick the host** from the page URL. `example.no##` also matches subdomains; use `www.example.no##` only to exclude other subdomains.
2. **Find a stable anchor** in the HTML the user provides (see selector priority below).
3. **Choose the filter type**: cosmetic (`##`) to hide elements, network (`||`) to stop a request, scriptlet (`##+js(...)`) to act on the page.
4. **Write the narrowest filter that catches every instance**, not just the pasted one.
5. **Tell the user how to verify**: Dashboard → My filters → Apply changes, then reload. Mention anything likely to need a follow-up (e.g. empty gaps left behind).

## Selector priority (most to least stable)

1. Custom element tag: `finn.no##advt-component`
2. Ad-specific id/class prefix: `[id^="adPlacement"]`, `[class^="advertory-"]`, `.polarisMarketing`
3. Semantic attributes: `[aria-label="Reklame"]`, `[data-edition-name^="partnerstudio-"]`
4. Parent of a known ad marker: `li:has(a.aksjelive-content-ad)`, `div.container:has(> [href^="https://ads.example"])`
5. Text match: `article:has-text(/^Annonse$/)` — use when nothing structural exists
6. `:xpath(...)` with absolute paths — last resort; breaks on any layout change

Avoid hashed/generated classes (`css-1x2y3z`, `hyperion-css-abc`) unless matched by prefix, and avoid utility classes (`.flex.mx-auto.my-4`).

## Filter types

| Goal | Syntax |
|------|--------|
| Hide element | `example.no##.ad-slot` |
| Remove element from DOM | `example.no##.ad-slot:remove()` |
| Hide wrapper that leaves a gap | `example.no##*:has(> advt-component)` |
| Block request | `\|\|ads.example.no^` or `\|\|example.no/path/to/script.js` |
| Allow request (exception) | `@@\|\|cmp.example.no^$domain=example.no` |
| Fix scroll lock | `example.no##body.no-scroll:style(overflow: auto !important;)` |
| Click a consent button | `cmp.example.no##+js(trusted-click-element, button[title="Avvis alle"], , 1000)` |
| Unhide (exception) | `example.no#@#.ad-slot` |

`:style()` and `+js()` must be site-specific (hostname required).

## Norwegian sites

Common ad markers: `Annonse`, `Reklame`, `Sponset`, `Annonsørinnhold`, `partnerstudio`, `brandstudio`, `polarisMarketing`, `advertory`, `finn-` placements (FINN/Schibsted/Vend ad network).

## Form inputs

When hiding inputs (`input`, `textarea`, `select`), anchor on `type`, `name`, `autocomplete`, or the parent `form`, e.g. `example.no##form[action*="newsletter"] input[type="email"]`. Hiding inputs can break login or checkout; tell the user to verify those flows.

## Anti-patterns

- Generic filters without a hostname (`##.ad`) — affect every site.
- Selectors copied from a single ad instance (specific ids like `advt_middle_1`, product URLs) — miss the other placements.
- Network-blocking a script the page needs to work; prefer cosmetic hiding when unsure.
