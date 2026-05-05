# Contributing

Notes for anyone editing the code or library after v3.2.

## Repo layout

```
.
├── index.html                                      ← main app (was nssu2.html)
├── nssu_photo_creator_with_rework_templates.html   ← photo creator
├── NSSU_Common_Practice_Library_v3_2.xlsx          ← canonical library
├── NSSU_Common_Practice_Library_v3_2.json          ← pre-converted library
├── README.md
├── CHANGELOG.md
├── RELEASE_NOTES_v3.2.md
└── CONTRIBUTING.md
```

The app is a single-file SPA per page — no build step, no bundler, no
package manager. Edit the HTML directly.

## Local dev

Open `index.html` in Chrome. That's the dev loop. There is no server-side
component. For File-System-Access-API-dependent flows (Save Draft to
folder, etc.) the file must be served over `http://` or `file://` — both
work in Chrome.

## Updating the library

The Common Practice Library is the source of truth for rework language
and the rule engine. Two formats are shipped:

- **`.xlsx`** — what people edit in Excel. Distributed to inspectors.
- **`.json`** — pre-parsed for offline / no-SheetJS environments.
  Regenerate this whenever the .xlsx changes.

### Editing rules in the spreadsheet

- **Defect tabs** (one per defect): Column A is the tool name (do not
  edit), Column B is the rework Step 1 prose. Blank Column B = no
  suggestion. Header rows occupy rows 1–3; data starts at row 4.
- **Rule sheets** (`Pressure_Overrides`, `Process_Overrides`, `PPE_Rules`,
  `Data_Rules`, `Focal_Point_Overrides`): same row layout — Column A is
  the condition/trigger key, Column B is the rule text. Don't change
  Column A values; the app reads them as exact keys.
- **Sheets the app ignores**: `Instructions`, `Engine_Logic`, and (as of
  v3.2) `Base_Behavior`. They can stay in the workbook for human
  reference but won't be loaded.

### Regenerating the .json after editing the .xlsx

Use the same parser shape the app does. The script below mirrors
`parseLibraryWorkbook` exactly:

```python
import json, datetime
from openpyxl import load_workbook

wb = load_workbook("NSSU_Common_Practice_Library_v3_2.xlsx", data_only=True)

result = {
    "version": "3.2",
    "reworkAssociations": {},
    "pressureOverrides": {},
    "processOverrides": {},
    "ppeRules": {},
    "dataRules": {},
    "focalOverrides": {},
    "parsedAt": datetime.datetime.now(datetime.UTC).isoformat(),
}

SKIP = {"Instructions", "Engine_Logic", "Base_Behavior"}
RULE_SHEETS = {
    "Pressure_Overrides":     "pressureOverrides",
    "Process_Overrides":      "processOverrides",
    "PPE_Rules":              "ppeRules",
    "Data_Rules":             "dataRules",
    "Focal_Point_Overrides":  "focalOverrides",
}

for name in wb.sheetnames:
    if name in SKIP: continue
    sh = wb[name]
    if name in RULE_SHEETS:
        target = result[RULE_SHEETS[name]]
        for r in range(4, sh.max_row + 1):
            cond = (str(sh.cell(r, 1).value).strip() if sh.cell(r, 1).value is not None else "")
            rule = (str(sh.cell(r, 2).value).strip() if sh.cell(r, 2).value is not None else "")
            if cond and rule: target[cond] = rule
        continue
    defect = name.replace(" - ", " / ").lower()
    assoc = {}
    for r in range(4, sh.max_row + 1):
        tool = (str(sh.cell(r, 1).value).strip().lower() if sh.cell(r, 1).value is not None else "")
        lang = (str(sh.cell(r, 2).value).strip() if sh.cell(r, 2).value is not None else "")
        if tool and lang and tool != "tool name":
            assoc[tool] = lang
    result["reworkAssociations"][defect] = assoc  # keep empty defects too

with open("NSSU_Common_Practice_Library_v3_2.json", "w", encoding="utf-8") as f:
    json.dump(result, f, indent=2, ensure_ascii=False)
```

If the rename version-bumps (e.g. v3.3), bump the `"version"` field, the
filename, and the references in `README.md` and `CHANGELOG.md`.

## How the rework lookup works (post-v3.2)

The relevant code is in `index.html`. Search for these symbols:

| Function | What it does |
| --- | --- |
| `parseLibraryWorkbook(workbook)` | Reads the workbook into a JSON object. Defect tabs become `reworkAssociations[defect][tool] = language`. Rule sheets become their respective dicts. |
| `applyLibraryOverride(parsed)` | Stores the parsed object in `libraryOverride` and `localStorage[nssuLibraryOverride]`, then re-renders. |
| `loadLibraryOverride()` | Restores `libraryOverride` from `localStorage` on page load. |
| `getLibraryReworkLanguage(defect, tool)` | Returns the language string from the library, or `null` if not present. |
| `getExactAssociation(defect, tool)` | **Library-only when a library is loaded.** Falls back to the bundled `exactReworkAssociations` table only when `libraryOverride === null` (true first-run case). |
| `buildReworkRule(defect, material, tool, recordMethodText)` | Wraps `getExactAssociation` and applies pressure overrides, paint-line guards, etc. If no template comes back, returns the manual-guide blue steps. |

What was removed in v3.2:
- `builtInBaseBehavior` — defect-agnostic templates.
- `buildStandardFallbackRule` — generated generic prose when no exact
  match existed.
- `STANDARD_FALLBACK_ENABLED` — feature flag for the above.
- `result.standardFallback` and `result.baseBehavior` — fields the parser
  used to populate but the lookup no longer consults.

If you need to bring any of this back, restore the deleted code rather
than rewriting it — git history has the original.

## Caveats when editing the script

- The main `<script>` block in `index.html` is ~3,100 brace pairs deep
  and ~600 KB. Validate brace balance after edits — `node --check`
  against an extracted copy of just the script catches syntax errors
  fast.
- `localStorage` survives across page loads. If you change the schema of
  the parsed library, bump the storage key (`LIBRARY_STORAGE_KEY`) so
  stale objects are not silently re-loaded.
- The photo creator and the main app share nothing in localStorage; they
  are linked only by `window.location.href` navigation.

## Testing changes

There is no automated test suite. Manual smoke test:

1. Clear `localStorage` for the page (`devtools → Application → Storage →
   Clear site data`).
2. Open `index.html`. Verify the bundled built-in rework language fills
   in for, e.g., Burr + Deburring Tool.
3. Press Update Library → load
   `NSSU_Common_Practice_Library_v3_2.xlsx`. Wait for the green toast.
4. Verify a populated cell still auto-fills (Burr + Deburring Tool).
5. Verify a deliberately-blank cell does NOT auto-fill (Burr + Magic
   Eraser, or any defect/tool you've blanked in the spreadsheet).
6. Verify a fully-blank defect tab (Chip, Crack, Cut, etc.) shows the
   manual-guide blue steps for every tool selection.
7. Press Update Library again with the `.json` file and confirm
   identical behavior.

## License

Internal — see project owner before redistributing.
