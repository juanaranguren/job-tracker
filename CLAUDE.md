# CLAUDE.md — Instructions for Claude Code

This file tells Claude Code how to work on this project. Read it fully before making any changes.

---

## What this project is

A single-file (`pipeline.html`) local-first web app for tracking job recruiting processes. No framework, no build tooling, no npm. Everything — HTML, CSS, JavaScript — lives in one file.

---

## Core constraints — never break these

1. **Stay single-file.** All changes go into `pipeline.html`. Do not create separate `.js` or `.css` files unless the user explicitly asks to split the project. The whole point is one file you can double-click.

2. **Zero new dependencies.** Do not add npm packages, CDN libraries, or any external scripts beyond the existing Google Fonts link. The app must work offline except for fonts.

3. **Preserve the storage engine.** The File System Access API + localStorage fallback is a core feature. Do not replace it with a different storage approach without explicit user instruction.

4. **Keep backward compatibility with `pipeline.json`.** If you change the data model, ensure existing files can still be loaded. Add fields with defaults; never remove or rename fields silently.

5. **Theme support.** The app supports both dark and light themes via CSS custom properties. Theme preference is stored in the `pipeline.json` file and persists across sessions and devices.

---

## How to make changes

Since there is no build step, all edits are direct modifications to `pipeline.html`:

- CSS changes → edit the `<style>` block
- HTML structure changes → edit the `<body>` markup
- Logic changes → edit the `<script>` block
- New features should follow the existing patterns (vanilla JS, async/await for storage, CSS custom properties for styling)

When adding a new function, place it in the correct section of the script (Storage / Render / Drawer / Interview Log / CRUD / Export) and add a comment header consistent with existing ones (`// ─── SECTION NAME ───`).

---

## Data model

See `TECHNICAL.md` for the full schema reference. Key points:

- `processes` is the single global array — all reads/writes go through it
- After every mutation, call `await persistData()` to save
- After every mutation that affects the board, call `render()`
- If the active drawer is open, call `populateDrawer(p)` to refresh it

### Adding a new field to a process

1. Add it to the form in the modal HTML (`f-yourfield`)
2. Add it to `saveProcess()` in the data object
3. Add it to `editProcess()` to pre-fill on edit
4. Add it to `populateDrawer()` if it should appear in the drawer
5. Add it to `exportCSV()` COLS array if it should appear in exports
6. Make sure `seedData()` includes it with a sensible default
7. Handle missing values gracefully (existing files won't have the field): use `p.yourfield || ''`

---

## Patterns to follow

### Persisting after a change
```javascript
async function doSomething() {
  // mutate processes array
  processes[idx].someField = newValue;
  // always persist
  await persistData();
  // always re-render board
  render();
  // show feedback
  toast('Done');
}
```

### Creating a new uid
```javascript
const id = uid(); // base36 timestamp + random suffix
```

### Escaping HTML before inserting into innerHTML
```javascript
el.innerHTML = `<div>${esc(untrustedString)}</div>`;
// never: el.innerHTML = `<div>${untrustedString}</div>`
```

### Showing the inline round form
The round form injects HTML into `#round-form-wrap`. Always call `cancelRoundForm()` before showing a new one to avoid duplicates.

---

## Feature roadmap (prioritised)

When the user asks to build new features, tackle them in this order unless they specify otherwise:

### 1. Comparison view (highest priority)
A new view (accessible from header) showing selected processes side-by-side. Each process is a column. Rows are comparison criteria: salary, work mode, interview progress, notes. Should allow the user to select which processes to compare (2–4 at a time).

**Suggested implementation:** Add a "Compare" button to the header. Clicking it opens a full-width panel below the board (similar to the drawer pattern). Use checkboxes on kanban cards to select processes for comparison.

### 2. Scoring / weighted ranking
Allow the user to define criteria (e.g. Salary, Culture, Growth, Tech stack, Location) and assign a weight (1–5) and a score (1–5) per process. Compute a weighted total. Display scores in the comparison view and optionally as a badge on the card.

**Data model addition:**
```json
"scores": {
  "salary": 4,
  "culture": 5,
  "growth": 3
}
```
Global criteria weights stored separately:
```json
"criteria": [
  { "id": "salary", "label": "Salary", "weight": 3 },
  { "id": "culture", "label": "Culture", "weight": 5 }
]
```
Store `criteria` at the top level of `pipeline.json` alongside `processes`.

### 3. Deadline / next action field
Add a `nextAction` object to each process:
```json
"nextAction": {
  "label": "System design interview",
  "date": "2026-03-28"
}
```
Show on card as a small date badge if set and within 7 days. Show overdue in red.

### 4. Filter / search bar
A search input in the header that filters cards in real time by company, role, or notes text. A status filter dropdown. Does not need to persist — reset on reload is fine.

### 5. CSV import
A file input (hidden, triggered by a button) that parses a CSV and creates process records. Map columns by header name. Skip rows with no company value. Show a summary toast: "Imported 5 processes".

---

## What to ask the user before building

- **Comparison view**: "Do you want the comparison to open as a panel below the board (like the drawer), or as a separate full-page view?"
- **Scoring**: "Do you want criteria to be fixed (predefined list) or fully custom (user-defined labels and weights)?"
- **Any UI change**: "Should I keep the current dark theme, or are you open to changes?"

---

## Things to avoid

- Do not use `innerHTML` with unescaped user data — always use `esc()` helper
- Do not call `render()` inside a loop — batch mutations first, then call once
- Do not add `position: fixed` elements — the drawer uses `position: sticky`-compatible layout
- Do not use `alert()` for feedback — use the `toast(msg)` function
- Do not hardcode the storage key string — use the `LS_KEY` constant
- Do not break the welcome screen flow — it must always be the first thing a new user sees
