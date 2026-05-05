# NSSU Input Portal — Deployment Package

Complete GitHub Pages deployment for the NSSU Input Portal and the Photo Page Creator.

## Files

| File | Purpose |
|------|---------|
| `index.html` | NSSU Input Form (the main app) |
| `nssu_photo_creator_with_rework_templates.html` | Additional Photo Page Creator |
| `NSSU_Common_Practice_Library_v3_2.xlsx` | Common defect / rework rules library |
| `NSSU_Common_Practice_Library_v3_2.json` | Library used at runtime by the app |
| `.nojekyll` | Tells GitHub Pages to serve files as-is, no Jekyll processing |
| `scripts/regenerate_library_json.py` | Optional helper to rebuild the JSON from the XLSX |

## Deploying to GitHub Pages

1. Replace **all files** in your repo's root with the files in this package.
2. Commit and push to your default branch (the one Pages serves from).
3. GitHub Pages should redeploy in ~30 seconds.
4. Open the site in an **incognito / private window** and hard-refresh (Ctrl+Shift+R / Cmd+Shift+R) to bypass any browser cache. Aggressive caching is the most common reason changes don't appear.

## Verifying a successful deployment

After hard-refresh, you should see:

- **Top-left of the masthead:** a "Photo Creator" button (dark gray).
- **Top-right of the masthead:** a "Home" button + hamburger menu + the NSSU logo.
- **Right column:** Page 3 Work Instructions Preview, then Page 4 Rework Steps Preview (when rework is selected), then a Save as Draft button, then **Page 2 — Safety Section** below all of that.
- The Page 1 form on the left ends at "DTL / Back and Forth Frequency". No "Page 1 Required Statements" row, no AI suggestion note under Save as Draft.

## What this build includes (all session changes layered)

### NSSU Input Form (`index.html`)
- "Photo Creator" button in the masthead with one-click handoff
- Page 2 Safety Section moved into the right column under the Rework Steps Preview (PDF output unchanged)
- Page 1 Required Statements section removed
- Misleading AI suggestion note removed from under Save as Draft
- Signature canvases 20% taller for easier use on touch devices
- Revision flow: clears only the bottom 3 NSSU Approval signatures (SRP, RI/TM, RI/GL); keeps the top 2 trained-on initials so they survive a revision
- Auto-saves the current NSSU as a draft when navigating to Photo Creator; auto-restores it on return
- Defensive CSS so the masthead-left actions can't collapse under any layout condition

### Photo Page Creator (`nssu_photo_creator_with_rework_templates.html`)
- Receives NSSU handoff via localStorage: top-section fields autofill (Tracking #, Kanban, Defect, Sort Location, Part Name, Supplier, Part #, Date)
- Caption autofill is category-aware:
  - Inspection templates: green "Good" caption pulled from inspection step #3, red "No Good" from step #4 (first sentence, step prefix stripped)
  - Captions with commas split into two centered lines (e.g. "If no Splits are present, / Part = Good")
  - Rework templates ignore handoff and use short hardcoded labels
- Witness Mark slot added — blue photo border, black caption with white text
- Witness panel made editable with bordered styling and content "Witness Mark Location: / 1st. = Blue / 2nd. = Yellow"
- Slot reorder via drag handle in top-left of each slot — insert before/after other slots, or drop into empty grid cells
- Independent row resizing — resize one slot without dragging row siblings
- WIS Photos section with 12 camera buttons (Work Area, Whole Part, Good, NG, Witness Mark, Suspect Area, Kanban, Rejection Step, Unpacking, Repacking, Rework Step, Rework Tools); buttons hide when their type is already in the current template
- Each WIS button uses a real SVG camera icon (camera-only intent, opens to back camera on mobile via `capture="environment"`)
- "Export JPEGs" button:
  - Photo page exported as a single landscape JPEG named `Additional Photo Page - <kanban> - <sortLocation>.jpeg`
  - Each filled template slot photo exported as its own JPEG named by slot type (e.g. `Good - <kanban> - <sortLocation>.jpeg`)
  - Each WIS photo exported as its own JPEG named by type
  - Resolution targets ~1600px (page) / ~1280px (individual photos), JPEG quality 0.82 — small enough for email, readable
- Fit-to-page printing — landscape US Letter, all colored borders/captions preserved
- "Back to NSSU" button correctly navigates to `index.html`

## Caveats / known limitations

- WIS photos and slot positions don't currently persist to JSON drafts — they live in memory for the current session only
- Photo page JPEG export uses html2canvas loaded from the Cloudflare CDN; without network connectivity the page-as-JPEG export won't work, but individual photo JPEGs will (they don't need html2canvas)
- "Camera-only" on the WIS buttons relies on the browser's `capture="environment"` hint. On Android Chrome and iOS Safari this opens the back camera directly. Desktop browsers always show a file picker. To enforce camera-only on Android, the eventual APK wrapper should swap these inputs for an Android `MediaStore.ACTION_IMAGE_CAPTURE` intent.

## Cache-busting

If after pushing you still see the old version:

1. Open the site in a fresh incognito/private window
2. Hard-refresh with Ctrl+Shift+R (Windows) or Cmd+Shift+R (Mac)
3. If still stale, view page source (right-click → "View Page Source") and search for `toPhotoCreatorBtn`. If you find it in the source, the file is deployed correctly and only the browser cache is stale. If you don't find it, the deployed file isn't the one in this package.
