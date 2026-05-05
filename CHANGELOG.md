# Changelog

All notable changes to the NSSU Input Portal are recorded here. The format
is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project follows the version number in the Common Practice Library.

## [3.2] — 2026-05-05

### Fixed
- **Update Library now actually applies blank cells.** Previously, pressing
  Update Library and selecting a spreadsheet with a blank cell for a
  given defect/tool combination did *not* clear the suggestion in the
  app — the lookup silently fell through to a hard-coded built-in copy
  of the rework language. The library is now the sole source of truth
  whenever one is loaded; blank means no association.

### Removed
- `builtInBaseBehavior` (defect-agnostic template object, ~16 entries).
- `buildStandardFallbackRule` function (constructed-prose fallback path).
- `STANDARD_FALLBACK_ENABLED` feature flag.
- `Base_Behavior` sheet handling in the library parser. If a future
  workbook contains this sheet it will be silently ignored.
- `result.standardFallback` template the parser used to inject into
  every loaded library (the "Use the {tool} provided to rework the part…"
  generic sentence).

### Changed
- `getExactAssociation` short-circuits on `libraryOverride` presence.
  Library loaded → library only. No library → bundled
  `exactReworkAssociations` table (first-run convenience).
- `parseLibraryWorkbook` now records every defect tab as an entry, even
  when every cell on that tab is blank. Previously, fully-blank defects
  were omitted from the parsed library, which let the lookup fall through
  to the bundled built-ins.
- Repository layout: main app renamed `nssu2.html` → `index.html` for
  GitHub Pages flat hosting. The photo creator's "Back to NSSU" link was
  updated to match.

### Added
- `NSSU_Common_Practice_Library_v3_2.json` — pre-converted library that
  loads without SheetJS, suitable for fully offline environments.
- `CHANGELOG.md`, `RELEASE_NOTES_v3.2.md`, `CONTRIBUTING.md`.

## [3.1] and earlier

Tracked in spreadsheet metadata only — no public changelog.
