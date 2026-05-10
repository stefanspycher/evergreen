# Evergreen Notes

React + Vite digital garden app inspired by Andy Matuschak's evergreen notes.
**All client-side** — no backend, no SSR, no database. Notes are `.md` files served statically.

## Stack

| Layer | Technology |
|---|---|
| UI | React 18.3, JSX (not TSX — no TypeScript) |
| Build | Vite 4.5, output → `_site/` |
| Router | React Router v6.24 |
| Styling | SCSS only — no Tailwind, no CSS modules |
| Package manager | Bun (`bun.lockb`), but scripts use `npx` prefix |
| Markdown | react-markdown v8 |
| Backlink indexer | `obsidian-index-wikilinks` (GitHub dep, not npm) |
| Deployment | Vercel (primary) + GitHub Pages (fallback) |

## Architecture

```
config.json              ← title, index path, bookmarks (homepage note IDs)
notes/*.md               ← content files; wiki-links: [[Note Title]]
notes/index.json         ← GENERATED — must rebuild after note changes
src/db/Db.jsx            ← singleton in-memory cache; all note fetches go here
src/main.jsx             ← router (2 routes only: / and /:entrypoint)
src/pages/Evergreen.jsx  ← loads index on mount, then renders Header + columns
src/components/          ← NoteColumnsContainer → NoteContainer → NoteLink/Footer
```

**Data flow:** `App mounts → Db.loadIndex() → navigate to /:entrypoint → Db.getNote(id) → NoteContainer renders markdown`

Stacked panels are encoded as URL query params: `/Home?stacked=lorem&stacked=ipsum`

## config.json — Primary Customization Surface

```json
{
  "title": "shown in Header",
  "index": "notes/index.json",
  "bookmarks": ["Home"]   // first entry = default route (/ redirects here)
}
```

Changing `bookmarks[0]` changes the homepage. Changing `index` to a wrong path breaks the entire app.

## Commands

```bash
bun start            # rebuild index + start dev (use this for content changes)
bun run dev          # dev server only (skip if notes were added/removed)
bun run build:index  # regenerate notes/index.json
bun run build        # Vite build → _site/ (run build:index first)
bun run format       # Prettier write (always run after edits)
bun run lint         # ESLint zero-warning policy
bun run preview      # preview the _site/ build locally
```

## Critical Constants — Know Before Changing

| Constant | Location | Risk if changed |
|---|---|---|
| `NOTE_WIDTH = 585` | `NoteColumnsContainer.jsx:12` | All scroll math derives from this; change breaks panel alignment |
| `< 800` breakpoint | `NoteColumnsContainer.jsx:82` | Single vs multi-panel toggle threshold |
| `"stacked"` param | `NoteColumnsContainer.jsx:51` + `NoteLink.jsx:57` | String literal in two places; rename both atomically |

## Known Issues — Check Before Implementing Related Features

1. **Db race condition** (`Db.jsx:33`): two concurrent callers for the same uncached note issue parallel fetches. Fix spec: `IMPROVEMENTS.md §1`.
2. **Popover alternates .404/note** (`NoteColumnsContainer.jsx:114`): causes excess re-renders. Fix spec: `IMPROVEMENTS.md §3`.
3. **`"stacked"` param has no shared constant**: rename spec in `IMPROVEMENTS.md §2`.
4. **`vercel.json` bug**: `buildCommand` references `"npm run vercel"` — that script does not exist. Deployments through Vercel CLI may fail. Warn user before any deploy work.

## Change Impact Map

| Change requested | Verify / warn about |
|---|---|
| Add/rename/delete a note | Run `build:index`; update any `config.json` bookmarks referencing that ID |
| Change site title | Edit `config.json.title` only |
| Change homepage note | Edit `config.json.bookmarks[0]` |
| Change panel width | Update `NOTE_WIDTH` in `NoteColumnsContainer.jsx` — affects all scroll math |
| Add a new component | Co-locate a `.scss` file; follow BEM class naming; see `.claude/rules/styling.md` |
| Add TypeScript | Requires tsconfig.json, renaming all `.jsx` → `.tsx`, adding type annotations. Not a small change — confirm scope. |
| Add Tailwind | Would replace all SCSS files — major refactor. Confirm scope. |
| Rename URL param `stacked` | Touch `NoteColumnsContainer.jsx` + `NoteLink.jsx` atomically; add migration effect (spec in `IMPROVEMENTS.md §2`) |
| Change Vite `base` or `outDir` | GitHub Pages routing/artifact breaks |
| Remove SPA script in `index.html` | GitHub Pages deep links 404 |

## Code Style

No semicolons. Double quotes. 2-space indent. Trailing commas. LF. Run `bun run format` after every edit.
ESLint: zero warnings allowed (`--max-warnings 0`). Prop-types validation is intentionally skipped (`/* eslint-disable react/prop-types */`).

## Note Writing Style

Rules for writing `.md` notes in `incubator/` and `notes/`: see `.claude/rules/note-writing-style.md`.
