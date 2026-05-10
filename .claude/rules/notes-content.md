# Notes Content System

## How Notes Work

Notes are plain `.md` files in `/notes/`. The app never writes them — only reads.
Backlinks and metadata live in `notes/index.json`, which is **generated**, not hand-edited.

## Index Rebuild Is Required

`notes/index.json` must be regenerated whenever notes are added, removed, or renamed:

```bash
bun run build:index   # regenerates notes/index.json
bun start             # also rebuilds index before starting dev server
```

If the index is stale: new notes appear blank, backlinks are wrong, and the `.404` note shows instead.

## Wiki-Links

Internal links in note content use `[[Note Title]]` syntax.
These are converted to markdown links at render time by `src/utils.js:noteToMarkdownContent`.
The conversion uses the note's `id` (its key in `index.json`) as the URL path segment.

To link to a note, use its exact title: `[[Home]]`, `[[lorem]]`, `[[Equi Ilion]]`.
Do not write markdown links manually inside note files — they won't integrate with backlinks.

## Note ID Convention

A note's ID is its path relative to `/notes/`, without the `.md` extension.
Examples: `Home`, `lorem`, `Equi/Equi Ilion`, `Tamen/Tamen in`

IDs appear in URLs as encoded path segments: `/Home?stacked=Equi%2FEqui%20Ilion`

Renaming a note file changes its ID and breaks any bookmarked URL pointing to it.
Always check `config.json.bookmarks` after renaming.

## Folder Structure in /notes/

Subfolders are supported (`notes/Equi/`, `notes/Tamen/`).
The `index.json` builder walks the tree recursively.
Folder nesting does not affect wiki-link resolution — links use the note title, not the path.

## Adding a Note

1. Create `notes/YourTitle.md` (or in a subfolder)
2. Write content; use `[[Other Note]]` for internal links
3. Run `bun run build:index` to register it in the index
4. Add to `config.json.bookmarks` if it should appear in the Header nav
