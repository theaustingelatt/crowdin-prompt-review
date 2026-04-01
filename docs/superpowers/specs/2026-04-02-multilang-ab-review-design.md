# Multi-Language A/B Translation Review Tool — Design Spec
**Date:** 2026-04-02

## Overview

Expand the existing French A/B translation review tool to support Spanish and German, and add a "Neither" option plus an optional per-choice note field to help extract translator reasoning for prompt crafting.

---

## Files

| File | Language | String ID prefix |
|------|----------|-----------------|
| `french.html` | French | `FR_` |
| `spanish.html` | Spanish | `ES_` |
| `german.html` | German | `DE_` |

`index.html` is renamed to `french.html`. Spanish and German files share the same structure and JavaScript logic — only `META` and `STRINGS` differ.

Data source: `3market.xlsx` — `basic` tab = `translation_a`, `pipeline` tab = `translation_b`, across 28 strings per language.

---

## Template Changes

### 1. Rule tags removed
- `card-rule` badge removed from every translator card
- `card-meta` row removed if it only contained the rule tag (the string ID stays)
- Rule breakdown section removed from Tech Summary entirely
- `rule` field removed from `STRINGS` array entries

### 2. Three-option translation grid

The two-column A/B grid becomes a three-column grid:

```
[ A ]  [ B ]  [ Neither ]
```

- **A** — blue, existing style
- **B** — green, existing style  
- **Neither** — neutral grey (`#94a3b8` border, `#f1f5f9` background when selected)
- Grid: `grid-template-columns: 1fr 1fr 1fr` on desktop, collapses to `1fr 1fr` then `1fr` on mobile
- "Neither" card has no translation text body — just the label `"Neither works"`
- Selecting "Neither" clears any previously typed note

### 3. Optional note field

After any selection (A, B, or Neither), a textarea appears below the translation grid:

```
[ Add a note (optional) — why did you choose this? ]
```

- Placeholder text as above
- `rows="2"`, resizable vertically
- Value stored in `choices` object alongside the selection: `choices[id] = { side: "a"|"b"|"neither", note: "" }`
- Note persists if the translator switches between A and B; clears when switching to or from "Neither"
- Note is NOT required — translator can leave it blank

### 4. Progress and action bar

- Progress counts strings where `choices[id]` exists (any side, including Neither)
- "All reviewed" threshold: every diff string has a choice recorded

---

## Data Model

`choices` object changes from `{ id: "a"|"b" }` to:

```js
{ id: { side: "a"|"b"|"neither", note: "..." } }
```

---

## Export — Results Link

URL hash encodes the full `choices` object (choice + note per string):

```js
// Resolved format encoded into hash:
{
  "FR_B001": { source: "pipeline", note: "More natural word order" },
  "FR_T001": { source: "basic",    note: "" },
  "FR_G003": { source: "neither",  note: "Both too formal for this context" }
}
```

- `btoa(JSON.stringify(resolved))` — same mechanism as current
- Old hash format (plain `{ id: "basic"|"pipeline" }`) decoded gracefully: wrapped into `{ source, note: "" }` on import
- URL format unchanged: `?tech#<base64>`

---

## Tech Summary Changes

### Removed
- "Breakdown by rule" section

### Kept
- Overall score: pipeline wins vs basic wins (large numbers)
- "How this worked" pipeline diagram

### Updated: Results table
Columns: **ID · Source · Chosen · Note**

- "Chosen" badge shows `pipeline` / `basic` / `neither` 
- "Neither" counted separately — not as a win for either workflow
- Overall score denominator excludes "Neither" strings (only counts strings where a side was chosen)
- Note column shows translator's note if present, `—` if blank

---

## Backward Compatibility

- Old results links (no notes, plain `{ id: "source" }` format) decode correctly
- No changes to the URL scheme

---

## Out of Scope

- Cross-language aggregation (each file is self-contained)
- Rule-based filtering or tagging
- Any server-side component — all files remain static HTML
