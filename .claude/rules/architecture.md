# Architecture and Component Conventions

## Component Naming and Location

- Components: PascalCase `.jsx` files in `src/components/`
- Page-level components: PascalCase `.jsx` files in `src/pages/`
- Utilities: camelCase `.js` in `src/`
- Data layer: `src/db/Db.jsx` (singleton class, not a hook)

## The Singleton Db Class

`src/db/Db.jsx` exports a single instance. **All note fetches go through it.**

```
Db.loadIndex()       ← loads notes/index.json; called once on app mount
Db.getNote(noteId)   ← fetches and caches a note by its ID
```

The class holds two in-memory caches: `_notesContent` (resolved notes) and `_notesIndex`.
**Known race condition**: two concurrent `getNote()` calls for the same uncached note issue
parallel fetches. Fix spec in `IMPROVEMENTS.md §1` before adding any feature that calls
`Db.getNote` from multiple places simultaneously.

Do not create a second Db instance. Do not bypass Db to fetch notes directly.

## Routing

Two routes only (defined in `src/main.jsx`):
- `/` → redirects to `/:bookmarks[0]`
- `/:entrypoint` → renders `Evergreen.jsx`

Stacked panels are **not** additional routes — they are query params on the same route:
`/Home?stacked=lorem&stacked=ipsum`

`NoteColumnsContainer` reads `useParams().entrypoint` (first panel) and
`useSearchParams()[0].getAll("stacked")` (additional panels).

`NoteLink` writes stacked state with `setSearchParams({ stacked: [...] })` — this
replaces all query params. Currently safe because no other query params exist.

The basename is auto-detected at runtime:
- GitHub Pages: `/${repoName}` (derived from `window.location.pathname`)
- Everywhere else: `Config.basename` from `config.json` (undefined by default → `/`)

## Component Tree

```
Evergreen (page)
└── Header                        ← title + bookmark links
└── NoteColumnsScrollingContainer ← scroll wrapper with ref
    └── NoteColumnsContainer      ← manages noteIds[], popover state, scroll tracking
        ├── NoteContainer (×n)    ← renders one note's markdown + footer
        │   ├── NoteLink (×n)     ← internal wiki-link with hover/click handlers
        │   └── Footer            ← backlinks (referenced_by from index)
        └── Popover               ← hover preview; mounts only when popoverData is set
```

## React Patterns in Use

- Functional components with hooks only — no class components
- `useCallback` for all event handlers passed as props
- `useEffect` for data fetching and side effects — not `useLayoutEffect`
- Scroll position polled via `setInterval(200ms)` — not IntersectionObserver
- `/* eslint-disable react/prop-types */` at file top — prop-types validation is intentionally skipped

## Utilities (src/utils.js)

`noteToMarkdownContent(base, note)` — prepends `# title` and converts `[[wiki-links]]` to markdown links.
`useBase()` — returns the router basename for constructing href values.

Do not add unrelated utilities to this file. If a utility is only used by one component, define it in that component file.

## No TypeScript

All source files are `.jsx`, not `.tsx`. There is no `tsconfig.json`.
`@types/react` and `@types/react-dom` are present only for editor autocomplete.
Adding TypeScript requires: tsconfig.json, renaming all JSX files, adding type annotations throughout.
This is a significant refactor — confirm scope before starting.
