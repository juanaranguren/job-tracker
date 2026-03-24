# Pipeline — Job Opportunity Tracker

A local-first, zero-dependency single-file web app for tracking multiple recruiting processes in parallel. Built for IT professionals managing several job opportunities at once.

## Philosophy

- **Single HTML file** — no install, no build step, no npm. Open in browser and use.
- **Local-first** — data lives in a JSON file the user picks on their machine, not in any cloud or server.
- **Zero dependencies** — vanilla HTML + CSS + JS only. One external resource: Google Fonts (cosmetic, non-breaking if offline).
- **Cross-browser** — File System Access API on Chrome/Edge (auto-save), localStorage fallback on Firefox/Safari.

---

## Stack

| Layer | Technology |
|---|---|
| Structure | HTML5 |
| Styling | CSS3 with custom properties (no framework) |
| Logic | Vanilla JavaScript ES2020 (async/await, no bundler) |
| Storage (primary) | File System Access API → writes to user-picked `pipeline.json` |
| Storage (fallback) | `localStorage` (Firefox / Safari) |
| Fonts | Google Fonts — DM Sans + DM Mono |
| Build tooling | None |

---

## Project structure

```
pipeline-project/
├── pipeline.html        ← entire app (single file)
├── README.md            ← this file
└── CLAUDE.md            ← instructions for Claude Code
```

All source lives in `pipeline.html`. There is no src/, dist/, or build/.

---

## Data model

The app reads and writes a single `pipeline.json` file. Structure:

```json
{
  "version": 2,
  "lastSaved": "2026-03-24T14:32:00Z",
  "processes": [
    {
      "id": "lf3k2a9m",
      "company": "Stripe",
      "role": "Staff Engineer",
      "salary": "$180k–$220k",
      "mode": "remote",
      "status": "interview",
      "contact": "james@stripe.com",
      "notes": "System design round next week.",
      "date": 1742550000000,
      "interviews": [
        {
          "id": "ir001",
          "type": "HR screen",
          "date": "2026-03-03",
          "interviewer": "Sarah K.",
          "outcome": "pass",
          "note": "Great culture fit conversation."
        }
      ]
    }
  ]
}
```

### Field reference

**Process fields**

| Field | Type | Values |
|---|---|---|
| `id` | string | uid (base36 timestamp + random) |
| `company` | string | free text |
| `role` | string | free text |
| `salary` | string | free text (e.g. "$140k–$170k") |
| `mode` | string | `remote` \| `hybrid` \| `onsite` \| `""` |
| `status` | string | `applied` \| `screening` \| `interview` \| `offer` \| `rejected` |
| `contact` | string | recruiter name or email |
| `notes` | string | free text |
| `date` | number | Unix timestamp (ms) — creation date |
| `interviews` | array | list of interview round objects |

**Interview round fields**

| Field | Type | Values |
|---|---|---|
| `id` | string | uid |
| `type` | string | `HR screen` \| `Technical screen` \| `System design` \| `Coding interview` \| `Hiring manager` \| `Final / panel` \| `Offer call` \| `Take-home` \| `Recruiter call` \| `Other` |
| `date` | string | ISO date `YYYY-MM-DD` |
| `interviewer` | string | free text (optional) |
| `outcome` | string | `pass` \| `fail` \| `pend` |
| `note` | string | free text notes from the round |

---

## Key functions reference

All logic lives inside the `<script>` tag in `pipeline.html`.

### Storage engine
| Function | Purpose |
|---|---|
| `initFS_new()` | Prompts user to create a new `.json` file via File System Access API |
| `initFS_open()` | Prompts user to open an existing `.json` file |
| `persistData()` | Writes current `processes` array to file or localStorage |
| `updateStorageIndicator()` | Updates the header status dot and filename label |

### Render
| Function | Purpose |
|---|---|
| `render()` | Full board re-render — rebuilds all 5 columns |
| `buildCard(p)` | Creates a single kanban card DOM element |
| `bindDrop()` | Attaches drag-and-drop handlers to all columns |
| `updateStats()` | Updates Total / Active chips in header |

### Drawer (detail panel)
| Function | Purpose |
|---|---|
| `toggleDrawer(id)` | Opens drawer for a process, or closes if same card clicked |
| `closeDrawer()` | Closes drawer and clears active state |
| `populateDrawer(p)` | Fills left pane with process metadata |
| `markNotesDirty()` | Shows "Save notes" button when notes textarea changes |
| `saveNotes()` | Persists notes field for active process |
| `editFromDrawer()` | Opens edit modal pre-filled from active drawer |

### Interview log
| Function | Purpose |
|---|---|
| `renderTimeline(p)` | Renders interview rounds as timeline in drawer right pane |
| `showRoundForm()` | Injects inline add-round form into drawer |
| `cancelRoundForm()` | Removes the inline form |
| `saveRound()` | Appends new round to active process and persists |
| `deleteRound(pid, rid)` | Removes a round by id |

### Process CRUD
| Function | Purpose |
|---|---|
| `openModal(prefillStatus?)` | Opens add process modal |
| `closeModal()` | Closes modal |
| `editProcess(id)` | Opens modal pre-filled for editing |
| `saveProcess()` | Creates or updates process and persists |
| `deleteProcess(id)` | Removes process after confirmation |

### Export
| Function | Purpose |
|---|---|
| `exportCSV()` | Downloads all processes as `.csv` (RFC 4180, UTF-8 BOM) |
| `exportJSON()` | Downloads raw `pipeline.json` (fallback mode only) |

---

## CSS architecture

All styles are in the `<style>` tag. Design system uses CSS custom properties:

```css
--bg / --bg2 / --bg3 / --bg4 / --bg5   /* background layers (dark theme) */
--border / --border2 / --border3        /* border opacity levels */
--text / --text2 / --text3              /* text hierarchy */
--accent / --accent2                    /* purple brand accent */
--green / --amber / --red / --cyan      /* semantic status colors */
--c-applied/screening/interview/offer/rejected  /* kanban column accents */
```

Theme is dark-only. Fonts: `DM Sans` (UI), `DM Mono` (metadata, chips, code-like labels).

---

## Planned features / next steps

These have been discussed but not yet built:

1. **Comparison view** — side-by-side evaluation of selected processes with a weighted scoring engine (criteria: salary, role fit, culture, growth, location). Lets user rank opportunities objectively.
2. **Scoring engine** — define custom criteria with weights, score each process 1–5 per criterion, compute weighted total.
3. **CSV import** — load processes from a CSV file (reverse of export).
4. **Stage checklist** (optional hybrid) — predefined pipeline checklist visible on card (HR → Technical → System Design → Offer).
5. **Deadline / next action field** — date field per process for "next interview" or "offer deadline", shown on card.
6. **Filter / search** — filter board by status, mode, or free-text search across company/role/notes.

---

## Development notes

- **No build step** — edit `pipeline.html` directly. Reload browser to see changes.
- **Testing** — open `pipeline.html` in Chrome. On first load it shows the welcome/file-picker screen.
- **Seed data** — the `seedData()` function populates sample processes. Called only when no data exists.
- **File System API** requires a user gesture (button click) to activate — cannot be triggered programmatically on page load. This is a browser security requirement and cannot be worked around.
- **localStorage key** is `pipeline_v2`. Changing the data model shape requires a migration or key bump.

---

## Chrome users: File access permissions

When using Chrome/Edge with the File System Access API, you may see a permission dialog from macOS or Windows when the app saves changes:

![Chrome file access permission dialog](images/chrome-access-docs.jpg)

> "Google Chrome.app" would like to access files in your Documents folder.

### Why this happens

- When you create or open a `pipeline.json` file in a protected folder (Documents, Desktop, Downloads)
- And make changes (add/edit/delete a card, update notes, etc.)
- Chrome needs to write to that file, so the OS asks for folder access permission

### What to do

**Click "OK" or "Allow"** — this is safe and necessary for auto-save to work. The app only writes to the specific file you selected.

### To avoid permission prompts

Save your `pipeline.json` file in a non-protected location:
- Create a dedicated folder like `~/pipeline-data/`
- Or use any folder outside Documents/Desktop/Downloads

### Alternative: Browser storage

If you prefer to avoid these prompts entirely, use the browser storage fallback:
- Firefox and Safari use localStorage by default (no file permissions needed)
- In Chrome, the app automatically falls back to localStorage if file access fails
