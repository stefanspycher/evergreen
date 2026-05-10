# Styling Conventions

## Technology

SCSS only. No Tailwind. No CSS modules. No CSS-in-JS.
Sass is compiled by Vite via the `sass` package.

## File Co-location

Every component has a co-located `.scss` file with the same base name:

```
Header.jsx        ← imports
Header.scss       ← co-located styles
```

Import the `.scss` at the top of the component file: `import "./Header.scss"`

## Class Naming (BEM-style)

Classes use PascalCase matching the component name as the block:

```scss
.NoteContainer { ... }           // block
.NoteContainer .PrimaryNote { }  // element (space-separated, not BEM __)
.NoteContainer.verticalMode { }  // modifier (adjoin, no space)
```

Do not introduce CSS modules, `styled-components`, or `@emotion` — they would require
touching every component file and break the existing class references.

## Global Styles

`src/index.scss` — font, background, box-sizing reset. Minimal. Do not add component styles here.

## Breakpoints

Single breakpoint: `800px` width.
- `> 800px`: multi-panel horizontal scroll
- `≤ 800px`: single-panel navigation

The breakpoint is enforced in JS (`window.innerWidth < 800`) in `NoteColumnsContainer.jsx:82`,
not via a CSS media query. If you add responsive SCSS, use the same value: `@media (max-width: 800px)`.

## Panel Width

`NOTE_WIDTH = 585` (px) is hardcoded in `NoteColumnsContainer.jsx:12`.
All horizontal positioning math (`left`, scroll offsets) multiplies by this value.
If changing the panel width, update the constant — do not override it in CSS.

## Adding Styles for a New Component

1. Create `ComponentName.scss` next to `ComponentName.jsx`
2. Import it in the JSX file
3. Use `.ComponentName` as the root class
4. Do not add styles to another component's `.scss` file
