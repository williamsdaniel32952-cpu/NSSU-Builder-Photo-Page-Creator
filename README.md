# NSSU Input Portal

Web-based NSSU Input form and supporting Photo Page Creator. Single-folder
deploy, no build step, works offline once loaded. The rework language and
rule engine are driven by a separate **Common Practice Library** spreadsheet
that you swap in via the **Update Library** button.

## Quick start

### Hosted (GitHub Pages)
Just open the published URL — `index.html` is the entry point.

### Local
1. Download or clone this repo.
2. Open `index.html` in Chrome (or any modern Chromium browser).
3. Open the hamburger menu → **Update Library** → select
   `NSSU_Common_Practice_Library_v3_2.xlsx`
   (or the faster `.json` version — see below).

That's it. The library is cached in `localStorage` under the key
`nssuLibraryOverride` and persists across sessions.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | Main NSSU Input Portal (was `nssu2.html`). |
| `nssu_photo_creator_with_rework_templates.html` | Secondary photo-page creator. The portal links to it from the **Photo Creator** button at the top. |
| `NSSU_Common_Practice_Library_v3_2.xlsx` | Canonical library — edit this in Excel to change rework language. |
| `NSSU_Common_Practice_Library_v3_2.json` | Pre-converted version of the same library. Loads instantly without needing SheetJS. Regenerate it whenever the .xlsx changes (see CONTRIBUTING.md). |
| `CHANGELOG.md` | Version history. |
| `RELEASE_NOTES_v3.2.md` | End-user notes for this release. |
| `CONTRIBUTING.md` | Notes for anyone editing the code or library. |

## Editing rework language

Open `NSSU_Common_Practice_Library_v3_2.xlsx` in Excel. Each defect has its
own tab. Find the row for the tool you want to change and edit the
**Language (Rework Step 1)** column.

- **Filled in** → the app shows that exact prose when the inspector picks
  that defect + tool.
- **Blank** → no suggestion; the app shows the blue manual-guide steps so
  the inspector fills them in by hand.

After saving the spreadsheet, press **Update Library** in the app to load
the new version. Changes take effect immediately and persist offline.

## Rework engine, in plain words

Once you've loaded a library:

> The library is the only source of truth. A blank cell means no
> association — period. Built-in defaults do not shine through.

Before you've ever loaded a library (first-run only), the app falls back
to its bundled `exactReworkAssociations` table so it isn't blank on first
open. The moment you press Update Library, that bundled fallback stops
being consulted.

There is no "base behavior" template, no "standard fallback prose"
generator, and no "Use the {tool} provided…" generic sentence. Those code
paths were removed in v3.2 — see `CHANGELOG.md`.

## Known limitations

- Browser-only: relies on `localStorage`, the File System Access API where
  available, and SheetJS (loaded from CDN) for `.xlsx` parsing. Use the
  `.json` library if you need fully offline library updates.
- Tested in Chrome. Other Chromium browsers should work; Safari/Firefox
  are untested.
