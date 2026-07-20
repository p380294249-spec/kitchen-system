# Kitchen System maintainer instructions

This repository is the production kitchen inventory PWA. Before making any change, read `CLAUDE_HANDOFF_PRIVATE.md` if it exists in the local worktree. It contains the current production URLs, API key, deployment details, data model, and verification checklist. Never print or commit its secrets.

## Non-negotiable safeguards

- Do not run `kt_init()` unless the owner explicitly requests a full database rebuild. It can overwrite or duplicate production setup.
- Do not rotate the API key, replace the spreadsheet, or create a new Apps Script deployment without explicit approval.
- Preserve the four user-facing tabs: 今天吃什么、库存、食谱、购物清单. The experimental 管家 tab was deliberately removed.
- The Google Sheet is the source of truth. Do not replace production records with sample data.
- Duplicate ingredient names are valid when their IDs or storage locations differ. For example, 纳豆 can exist in both freezer and refrigerator.
- Accept mixed Sheet cell types. Location, quantity, booleans, and ingredient lists may arrive as strings or numbers; normalize before comparison.
- `Code.gs` and `CLAUDE_HANDOFF_PRIVATE.md` are intentionally ignored by Git. Deploy backend changes with clasp; never force-add secrets.
- Preserve unrelated or untracked local files such as `.claspignore` and `appsscript.json`.

## Normal workflow

1. Run `git status --short` and read the relevant files before editing.
2. Confirm the production API with the read-only `ping` and `get_all` checks from the private handoff.
3. Explain the proposed change and its affected files before editing when the request is ambiguous.
4. Make the smallest compatible change. Do not redesign the interface unless asked.
5. Check JavaScript syntax and test the read path plus the exact changed behavior.
6. For backend changes, run `clasp push -f`, update the existing deployment, and verify production. Do not create another deployment URL.
7. Commit only intentional public files and push `main`. Wait for GitHub Pages, then verify the stable app URL.
8. Report exactly what changed, what was deployed, and what remains for the owner to test on iPhone.

## Architecture summary

- `index.html`: single-page frontend and API client.
- `sw.js`: PWA shell caching. Increment the cache name when frontend assets must refresh on installed devices.
- `manifest.webmanifest` and `icons/`: installable PWA metadata/assets.
- `Code.gs`: Apps Script JSON/JSONP API backed by Google Sheets; local-only because it contains the API key.
- GitHub Pages serves the frontend; Apps Script serves the database API.
- The frontend keeps a last-successful read-only snapshot so a cold start or brief Apps Script delay does not show an empty kitchen.

Do not assume a Google Drive connector is required. Source maintenance uses this local repository and clasp. Runtime inventory reads/writes use the existing Apps Script API.
