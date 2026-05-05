# NSSU v3.2 — Release Notes

## What's new for inspectors and SRPs

### Update Library now does what it says
In every prior version, when you blanked out a tool on a defect tab in the
Common Practice Library and pressed **Update Library** in the app, the
suggestion did not actually go away — the app continued to show its
original built-in suggestion in that cell, even though the spreadsheet
clearly said "leave blank for no suggestion."

Starting with v3.2, **the spreadsheet is the only source of truth for
rework language**. If you blank a cell, save the spreadsheet, and press
Update Library, that defect/tool combination will no longer auto-fill any
prose. You'll see the blue manual-guide steps instead, ready for you to
type the correct rework wording for your part.

### No more generic "Use the {tool} provided…" sentences
The app used to construct generic rework prose ("Use the deburring tool
provided to rework the part, so that it matches the Good Part Photo…")
when it couldn't find an exact match. Those generic sentences are gone.
If the library doesn't have language for a defect/tool combination, you
get the manual-guide blue steps to fill in by hand — no auto-generated
filler text.

### No more defect-agnostic "base behavior" auto-fills
Similarly, the app used to fall back to short defect-agnostic templates
(e.g. "Using the scissors provided, carefully cut the [defect] away from
the suspect area") for tools where no exact match existed. Those are also
gone in v3.2 — defect-specific language must come from the library.

## What hasn't changed

- The hamburger menu, the Update Library button, Open / Open Drafts /
  Clone, Export Edit Report, Text Field and Manual toggles — all the
  same.
- The Photo Creator and all its templates — unchanged.
- Pressure overrides, PPE rules, data rules, focal-point overrides — all
  still loaded from their respective tabs in the library.
- Library lives in browser storage and persists offline.

## How to update at your site

1. Pull the latest files from the repo (or open the published GitHub
   Pages URL).
2. Open the app in Chrome.
3. Hamburger menu → **Update Library** → select
   `NSSU_Common_Practice_Library_v3_2.xlsx`.
4. Look for the green "Library updated" toast. Done.

If you keep your spreadsheet edits in a custom location, just point Update
Library at your file instead — the format is unchanged.

## What to check after upgrading

Walk through a few defect/tool combinations you know you previously left
blank in the library, and confirm:

- Picking those combinations no longer auto-fills any prose.
- The blue manual-guide steps appear instead.
- Combinations that *are* populated in the library still auto-fill the
  expected language.
