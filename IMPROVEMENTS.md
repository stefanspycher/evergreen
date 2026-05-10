# Improvement Specifications

Three targeted improvements to the evergreen implementation, derived from
comparing the original Andy Matuschak site with this codebase. Each spec
is self-contained: current behavior, exact changes, risks, and mitigations.

**Stack:** React 18, React Router v6.24, Vite 4, vanilla SCSS, singleton `Db` class.

---

## 1. Hover Prefetch

### Current behavior

When a user hovers a `NoteLink`, the fetch for the hovered note starts only
after two React render cycles:

```
mouseEnter fires
  → showPopoverForNote() sets popoverData state in NoteColumnsContainer
    → React re-renders, mounts <Popover>
      → Popover's useEffect runs
        → Db.getNote(noteId) called   ← fetch starts here
```

If the user hovers and clicks quickly — before the `Popover` has mounted and
its `useEffect` has fired — the note data is not yet cached. The new panel
opens but waits for the fetch to resolve before rendering content.

The `Db` singleton has an in-memory cache (`_notesContent`). Once a note is
fetched once, subsequent calls return instantly. The goal is to prime that
cache at the earliest possible moment: `mouseEnter`, not component mount.

There is also a race condition in `Db._getOrLoadNote`: if two callers request
the same uncached note before either resolves, two parallel fetch requests are
issued. Currently low-impact because only one caller (Popover) triggers
fetches. With eager prefetching, NoteLink and Popover would both call
`Db.getNote` for the same note, making the race real.

### Desired behavior

The fetch starts on `mouseEnter` in `NoteLink`, independent of whether the
popover has mounted. `Db` deduplicates in-flight requests so hovering the
same link twice — or having Popover and NoteLink both call `Db.getNote` — 
issues exactly one network request.

### Changes

**`src/db/Db.jsx`** — add a pending-fetch map to deduplicate concurrent requests:

```jsx
class DB {
  _notesContent
  _notesIndex
  _pendingFetches   // ADD THIS

  constructor() {
    this._notesContent = {}
    this._pendingFetches = {}   // ADD THIS
  }

  // Replace _getOrLoadNote with:
  async _getOrLoadNote(noteId) {
    if (noteId in this._notesContent) return this._notesContent[noteId]
    if (noteId in this._pendingFetches) return this._pendingFetches[noteId]

    const promise = (async () => {
      const index = await this._index()
      const response = await _loadNoteContent(index[noteId])
      const content = await response.text()
      const note = { ...index[noteId], id: noteId, content }
      this._notesContent[noteId] = note
      delete this._pendingFetches[noteId]
      return note
    })()

    this._pendingFetches[noteId] = promise
    return promise
  }
}
```

**`src/components/NoteLink.jsx`** — fire prefetch immediately on mouseEnter,
before the popover render cycle:

```jsx
// Add import at top
import Db from "../db/Db"

// Replace onMouseEnter with:
const onMouseEnter = useCallback(
  (e) => {
    if (isRemote) return
    Db.getNote(targetNoteId)   // fire and forget — warms cache immediately
    showPopoverForNote({
      noteId: targetNoteId,
      elementPosition: e.target.getBoundingClientRect(),
    })
  },
  [isRemote, showPopoverForNote, targetNoteId],
)
```

### Risks and mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| `Db.getNote` called before `loadIndex()` completes (app just opened, user hovers before index JSON arrives) | Low — index loads on app mount, hover requires page to be visible | `_getOrLoadNote` already calls `this._index()` which awaits `loadIndex()`. Safe. |
| Prefetch for a note that doesn't exist (`.404` path) | Low — only real `NoteLink` elements trigger hover | `_loadNoteContent` falls back to `404.md` path. Cache stores the 404 note. Harmless. |
| Rapid hover across many links causes many parallel fetches | Medium — fast cursor movement | The `_pendingFetches` deduplication prevents duplicate requests per noteId. Distinct notes still fetch in parallel, which is acceptable browser behavior (HTTP/2 multiplexing). |
| `delete this._pendingFetches[noteId]` inside the async IIFE — if two awaits resolve in the same tick, could double-delete | Negligible | `delete` on a missing key is a no-op in JS. |

---

## 2. URL Parameter Name

### Current behavior

The stacked panel state is encoded as repeated `stacked` params:

```
/Home?stacked=lorem&stacked=ipsum
```

Read in `NoteColumnsContainer.jsx`:
```js
query.getAll("stacked")
```

Written in `NoteLink.jsx`:
```js
setSearchParams({ stacked: [...noteIdsStack.slice(1, from + 1), targetNoteId] })
```

The param name `stacked` is ambiguous out of context. The original Matuschak
site uses `stackedNotes`, which is self-describing and widely recognized in
implementations of this pattern. Additionally, the param name is a string
literal duplicated in two files with no shared constant — a rename today
requires touching both files and risks divergence.

### Desired behavior

The URL reads:

```
/Home?stackedNotes=lorem&stackedNotes=ipsum
```

The param name is defined once as a constant and imported wherever needed.
Existing URLs using `stacked` are silently redirected to `stackedNotes` at
app startup so no bookmarked links break.

### Changes

**`src/utils.js`** — add the shared constant:

```js
export const STACKED_PARAM = 'stackedNotes'
```

**`src/components/NoteColumnsContainer.jsx`** — import and use constant;
add one-time migration for old `stacked` param:

```jsx
import { noteToMarkdownContent, useBase, STACKED_PARAM } from "../utils"

// Inside NoteColumnsContainer, add migration effect BEFORE the existing
// noteIds effect:
const [searchParams, setSearchParams] = useSearchParams()

useEffect(() => {
  const legacy = searchParams.getAll("stacked")
  if (legacy.length > 0) {
    const next = new URLSearchParams(searchParams)
    legacy.forEach(v => next.append(STACKED_PARAM, v))
    next.delete("stacked")
    setSearchParams(next, { replace: true })
  }
}, [])  // runs once on mount

// Update existing noteIds effect:
useEffect(() => {
  setNoteIds(
    [entrypoint, ...query.getAll(STACKED_PARAM)].map((e) =>
      decodeURIComponent(e),
    ),
  )
}, [entrypoint, query])
```

Note: the existing code destructures `useSearchParams` as `const query =
useSearchParams()[0]`. Change this line to destructure both values so the
migration effect can also write:

```jsx
// Replace:
const query = useSearchParams()[0]
// With:
const [query, setSearchParams] = useSearchParams()
```

**`src/components/NoteLink.jsx`** — import and use constant:

```jsx
import { useBase, STACKED_PARAM } from "../utils"

// Replace:
setSearchParams({ stacked: [...noteIdsStack.slice(1, from + 1), targetNoteId] })
// With:
setSearchParams({ [STACKED_PARAM]: [...noteIdsStack.slice(1, from + 1), targetNoteId] })
```

### Risks and mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Migration effect runs, sets new params, triggers another render, runs again (infinite loop) | Medium — `useEffect` with empty deps runs once, but `setSearchParams` causes a re-render | Empty dependency array `[]` ensures effect runs only on mount. The `stacked` check short-circuits on re-render because `legacy.length === 0` after migration. |
| `setSearchParams({ [STACKED_PARAM]: array })` replaces ALL query params, not just the stacked key | Low — app currently has no other query params | Acceptable for current scope. If other params are added later, change to: `const next = new URLSearchParams(searchParams); next.delete(STACKED_PARAM); array.forEach(v => next.append(STACKED_PARAM, v)); setSearchParams(next)` |
| `useSearchParams` now destructured as `[query, setSearchParams]` — `setSearchParams` name collides with the import in the migration effect | None — both are the same function from the same hook call | Verify there is no second `useSearchParams` call in the component after refactor. Remove any duplicate. |
| Note filenames containing `&`, `=`, or `#` break the URL | Low — current notes use plain filenames | `encodeURIComponent` is already applied in `Footer.jsx` and `NoteLink.jsx` href construction. `setSearchParams` also encodes values. Safe. |

---

## 3. Hover Debounce

### Current behavior

`showPopoverForNote` is called synchronously inside `onMouseEnter` with zero
delay. Moving the cursor across a paragraph of wikilinks triggers the popover
to mount and unmount for every link the cursor passes through, causing:

- Redundant fetches (partially addressed by Improvement 1, but mounts/unmounts
  still occur)
- Visible flicker as the popover appears and disappears rapidly
- Excess re-renders in `NoteColumnsContainer` (acknowledged in the existing
  `//TODO: bug with popover, causes MANY re-renders` comment)

### Desired behavior

The popover appears only if the cursor dwells on a link for 150ms or longer.
Moving the cursor away before 150ms cancels the pending show. The prefetch
from Improvement 1 still fires immediately on `mouseEnter` — the debounce
applies only to the UI, not the data fetch.

The bounding rect of the hovered element is captured at `mouseEnter` time, not
inside the setTimeout callback, because the element position can change if the
page scrolls during the 150ms window.

### Changes

**`src/components/NoteLink.jsx`**:

```jsx
// Add to imports:
import { useRef } from "react"
import Db from "../db/Db"

// Inside NoteLink component, add ref:
const hoverTimerRef = useRef(null)

// Replace onMouseEnter:
const onMouseEnter = useCallback(
  (e) => {
    if (isRemote) return
    Db.getNote(targetNoteId)                        // prefetch — immediate
    const rect = e.target.getBoundingClientRect()   // capture position now
    clearTimeout(hoverTimerRef.current)
    hoverTimerRef.current = setTimeout(() => {
      showPopoverForNote({ noteId: targetNoteId, elementPosition: rect })
    }, 150)
  },
  [isRemote, showPopoverForNote, targetNoteId],
)

// Replace onMouseLeave:
const onMouseLeave = useCallback(() => {
  if (isRemote) return
  clearTimeout(hoverTimerRef.current)
  showPopoverForNote()   // clear popover immediately
}, [isRemote, showPopoverForNote])
```

### Risks and mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| `hoverTimerRef.current` not cleaned up if component unmounts while timer is pending (e.g., panel closes mid-hover) | Low — panels don't typically unmount while a link is hovered | Add cleanup in a `useEffect` return: `useEffect(() => () => clearTimeout(hoverTimerRef.current), [])`. Prevents a setState call on an unmounted component. |
| Captured `rect` is stale if the page scrolls in 150ms | Very low — 150ms is shorter than a typical scroll gesture | Acceptable. Alternative is capturing `rect` inside the callback, but that reads layout at an unpredictable time. Capturing at event time is the safer default. |
| 150ms feels too slow or too fast for the specific content density | Subjective | Extract as a named constant: `const HOVER_DELAY_MS = 150` at the top of `NoteLink.jsx` for easy tuning. |
| Combined with Improvement 1: `Db.getNote` fires on every mouseEnter including rapid passes. Could this cause UI jank from background fetches? | Low — fetch responses are processed off the main thread; only state updates cause re-renders, and the cache check prevents redundant fetches (Improvement 1) | Monitor with React DevTools Profiler if jank is observed. The `_pendingFetches` deduplication from Improvement 1 is essential here. |

---

## Implementation order

Apply in this sequence to avoid compound breakage:

1. **Db deduplication** (Improvement 1, Db.jsx only) — isolated change, no UI impact, testable by opening DevTools Network tab and hovering the same link twice.
2. **Hover debounce** (Improvement 3) — depends on nothing, immediately reduces re-renders.
3. **NoteLink prefetch** (Improvement 1, NoteLink.jsx) — add `Db.getNote` call. Verify with Network tab that hovering starts a fetch before popover appears.
4. **URL rename** (Improvement 2) — last, because it changes observable URL shape. Test by: opening a stacked URL, refreshing, confirming panels re-open; also test that old `?stacked=` URLs redirect cleanly.
