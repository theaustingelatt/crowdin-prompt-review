# Multi-Language A/B Review Tool — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Expand the French-only A/B translation review tool to French, Spanish, and German, adding a "Neither" option and an optional note field per choice.

**Architecture:** Three self-contained static HTML files (`french.html`, `spanish.html`, `german.html`) sharing the same structure. `index.html` is renamed to `french.html`. All logic is embedded JS — no build system. Data is hardcoded in each file from `3market.xlsx` (basic tab = translation_a, pipeline tab = translation_b).

**Tech Stack:** Vanilla HTML/CSS/JS, no dependencies, no framework.

---

## File Map

| Action | File | What changes |
|--------|------|-------------|
| Rename + modify | `index.html` → `french.html` | All template changes + French data from XLSX |
| Create | `spanish.html` | Same template, ES_ data |
| Create | `german.html` | Same template, DE_ data |
| Delete | `index.html` | Superseded by `french.html` |

---

## Task 1: Update CSS in `french.html`

**Files:**
- Modify: `french.html` (rename from `index.html` first)

- [ ] **Step 1.1: Rename index.html to french.html**

```bash
cp index.html french.html
```

- [ ] **Step 1.2: Update the `.translations` grid to 3 columns**

Find this in the CSS:
```css
.translations { display: grid; grid-template-columns: 1fr 1fr; gap: 0.75rem; }
@media (max-width: 560px) { .translations { grid-template-columns: 1fr; } }
```

Replace with:
```css
.translations { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 0.75rem; }
@media (max-width: 700px) { .translations { grid-template-columns: 1fr 1fr; } }
@media (max-width: 460px) { .translations { grid-template-columns: 1fr; } }
```

- [ ] **Step 1.3: Add Neither and note styles**

Find the `/* ── Empty state ── */` comment and insert these styles immediately before it:

```css
/* ── Neither option ── */
.t-option.chosen-neither { border-color: #94a3b8; background: #f1f5f9; }
.chosen-neither .chosen-badge { display: block; background: #64748b; color: white; }
.t-label.neither { color: #64748b; }

/* ── Note field ── */
.note-field { width: 100%; margin-top: 0.75rem; padding: 0.5rem 0.75rem; border: 1px solid var(--border); border-radius: 6px; font-family: inherit; font-size: 0.82rem; resize: vertical; outline: none; background: #fafafa; color: var(--text); line-height: 1.5; }
.note-field:focus { border-color: var(--a); background: #fff; }
```

- [ ] **Step 1.4: Update results table to 4 columns**

Find:
```css
.results-header { display: grid; grid-template-columns: 70px 1fr 1fr 1fr 90px; gap: 0.75rem; padding: 0.6rem 0.75rem; font-size: 0.72rem; font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em; color: var(--muted); border-bottom: 2px solid var(--border); }
.result-row { display: grid; grid-template-columns: 70px 1fr 1fr 1fr 90px; gap: 0.75rem; padding: 0.75rem; font-size: 0.82rem; border-bottom: 1px solid var(--border); align-items: start; }
.result-row:last-child { border-bottom: none; }
.result-row:nth-child(even) { background: #fafafa; }
.result-id { font-family: monospace; font-size: 0.72rem; color: var(--muted); padding-top: 0.1rem; }
.result-choice { display: inline-block; padding: 0.2rem 0.5rem; border-radius: 4px; font-size: 0.72rem; font-weight: 700; text-transform: uppercase; }
.result-choice.a { background: var(--a-bg); color: var(--a); }
.result-choice.b { background: var(--b-bg); color: var(--b); }
.result-choice.none { background: var(--rule-bg); color: var(--muted); }
@media (max-width: 700px) { .results-header, .result-row { grid-template-columns: 60px 1fr 1fr; } .results-header > *:nth-child(3), .result-row > *:nth-child(3), .results-header > *:nth-child(4), .result-row > *:nth-child(4) { display: none; } }
```

Replace with:
```css
.results-header { display: grid; grid-template-columns: 70px 1fr 100px 1fr; gap: 0.75rem; padding: 0.6rem 0.75rem; font-size: 0.72rem; font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em; color: var(--muted); border-bottom: 2px solid var(--border); }
.result-row { display: grid; grid-template-columns: 70px 1fr 100px 1fr; gap: 0.75rem; padding: 0.75rem; font-size: 0.82rem; border-bottom: 1px solid var(--border); align-items: start; }
.result-row:last-child { border-bottom: none; }
.result-row:nth-child(even) { background: #fafafa; }
.result-id { font-family: monospace; font-size: 0.72rem; color: var(--muted); padding-top: 0.1rem; }
.result-choice { display: inline-block; padding: 0.2rem 0.5rem; border-radius: 4px; font-size: 0.72rem; font-weight: 700; text-transform: uppercase; }
.result-choice.pipeline { background: var(--b-bg); color: var(--b); }
.result-choice.basic { background: var(--a-bg); color: var(--a); }
.result-choice.neither { background: var(--rule-bg); color: var(--muted); }
@media (max-width: 700px) { .results-header, .result-row { grid-template-columns: 60px 1fr 80px; } .results-header > *:nth-child(4), .result-row > *:nth-child(4) { display: none; } }
```

- [ ] **Step 1.5: Remove the `.card-rule` CSS rule**

Find and delete:
```css
.card-rule { font-size: 0.72rem; background: var(--rule-bg); color: var(--muted); padding: 0.2rem 0.55rem; border-radius: 4px; }
```

---

## Task 2: Update JS data model and helper functions

**Files:**
- Modify: `french.html`

- [ ] **Step 2.1: Update the `choices` comment and `choose()` function**

Find:
```js
// Translator choices — populated by clicking A/B cards
// or imported via JSON paste in the Tech view
let choices = {}; // { "FR_B001": "a", "FR_T002": "b", ... }
```

Replace with:
```js
// Translator choices — populated by clicking A/B/Neither cards
// Format: { "FR_B001": { side: "a"|"b"|"neither", note: "" }, ... }
let choices = {};
```

- [ ] **Step 2.2: Replace the `choose()` function**

Find:
```js
function choose(id, side) {
  choices[id] = side;
  renderTranslator();
}
```

Replace with:
```js
function choose(id, side) {
  const prev = choices[id];
  // Preserve note when switching between A and B; clear when Neither is involved
  const note = (prev && prev.side !== 'neither' && side !== 'neither') ? prev.note : '';
  choices[id] = { side, note };
  renderTranslator();
}

function updateNote(id, note) {
  if (choices[id]) choices[id].note = note;
}
```

- [ ] **Step 2.3: Update `clearChoices()`**

The existing `clearChoices()` sets `choices = {}` — no change needed, it still works.

- [ ] **Step 2.4: Update `exportChoices()`**

Find:
```js
function exportChoices() {
  // Resolve a/b back to basic/pipeline, encode into URL, copy link
  const resolved = {};
  Object.keys(choices).forEach(id => {
    resolved[id] = sides[id][choices[id]].source;
  });
  const encoded = btoa(JSON.stringify(resolved));
  const url = `${location.origin}${location.pathname}?tech#${encoded}`;
  navigator.clipboard.writeText(url).then(() => {
    toast('Results link copied — send this to your team ✓');
  }).catch(() => {
    prompt('Copy this link and send it to your team:', url);
  });
}
```

Replace with:
```js
function exportChoices() {
  const resolved = {};
  Object.keys(choices).forEach(id => {
    const { side, note } = choices[id];
    const source = side === 'neither' ? 'neither' : sides[id][side].source;
    resolved[id] = { source, note: note || '' };
  });
  const encoded = btoa(unescape(encodeURIComponent(JSON.stringify(resolved))));
  const url = `${location.origin}${location.pathname}?tech#${encoded}`;
  navigator.clipboard.writeText(url).then(() => {
    toast('Results link copied — send this to your team ✓');
  }).catch(() => {
    prompt('Copy this link and send it to your team:', url);
  });
}
```

Note: `btoa(unescape(encodeURIComponent(...)))` handles non-ASCII characters in notes safely.

---

## Task 3: Update `renderTranslator()`

**Files:**
- Modify: `french.html`

- [ ] **Step 3.1: Update the progress / reviewed count**

The existing `const reviewed = diffs.filter(s => choices[s.id]).length;` still works — `choices[s.id]` is truthy when set. No change needed.

- [ ] **Step 3.2: Replace the card rendering inside the `diffs.forEach` block**

Find this block inside `renderTranslator()`:
```js
  diffs.forEach(s => {
    const c = choices[s.id];
    html += `
      <div class="card" id="card-${s.id}">
        <div class="source-text">${escHtml(s.source)}</div>
        <div class="translations">
          <div class="t-option${c === 'a' ? ' chosen-a' : ''}" onclick="choose('${s.id}','a')" title="">
            <div class="chosen-badge">chosen</div>
            <div class="t-label a">A</div>
            <div class="t-text">${escHtml(s.sides.a.text)}</div>
          </div>
          <div class="t-option${c === 'b' ? ' chosen-b' : ''}" onclick="choose('${s.id}','b')" title="">
            <div class="chosen-badge">chosen</div>
            <div class="t-label b">B</div>
            <div class="t-text">${escHtml(s.sides.b.text)}</div>
          </div>
        </div>
      </div>`;
  });
```

Replace with:
```js
  diffs.forEach(s => {
    const c = choices[s.id];
    const chosenSide = c ? c.side : null;
    html += `
      <div class="card" id="card-${s.id}">
        <div class="card-meta">
          <span class="card-id">${escHtml(s.id)}</span>
        </div>
        <div class="source-text">${escHtml(s.source)}</div>
        <div class="translations">
          <div class="t-option${chosenSide === 'a' ? ' chosen-a' : ''}" onclick="choose('${s.id}','a')">
            <div class="chosen-badge">chosen</div>
            <div class="t-label a">A</div>
            <div class="t-text">${escHtml(s.sides.a.text)}</div>
          </div>
          <div class="t-option${chosenSide === 'b' ? ' chosen-b' : ''}" onclick="choose('${s.id}','b')">
            <div class="chosen-badge">chosen</div>
            <div class="t-label b">B</div>
            <div class="t-text">${escHtml(s.sides.b.text)}</div>
          </div>
          <div class="t-option${chosenSide === 'neither' ? ' chosen-neither' : ''}" onclick="choose('${s.id}','neither')">
            <div class="chosen-badge">chosen</div>
            <div class="t-label neither">Neither</div>
            <div class="t-text empty">Neither works</div>
          </div>
        </div>
        ${chosenSide ? `<textarea class="note-field" rows="2"
          placeholder="Add a note (optional) — why did you choose this?"
          oninput="updateNote('${s.id}', this.value)">${escHtml(c.note || '')}</textarea>` : ''}
      </div>`;
  });
```

---

## Task 4: Update `renderTech()`

**Files:**
- Modify: `french.html`

- [ ] **Step 4.1: Replace the entire `renderTech()` function**

Find `function renderTech() {` and replace the entire function with:

```js
function renderTech() {
  const container = document.getElementById('tech-content');
  const diffs = diffStrings();
  const reviewed = diffs.filter(s => choices[s.id]);
  const total = reviewed.length;

  if (total === 0) {
    container.innerHTML = `
      <div class="empty-state">
        <div class="icon">📊</div>
        <h3>Awaiting translator results</h3>
        <p>This view populates automatically when you open the results link sent by the translator.</p>
      </div>`;
    return;
  }

  const sided   = reviewed.filter(s => choices[s.id].side !== 'neither');
  const basicWins    = sided.filter(s => sides[s.id][choices[s.id].side].source === 'basic').length;
  const pipelineWins = sided.filter(s => sides[s.id][choices[s.id].side].source === 'pipeline').length;
  const neitherCount = reviewed.length - sided.length;

  const tableRows = reviewed.map(s => {
    const { side, note } = choices[s.id];
    const source = side === 'neither' ? 'neither' : sides[s.id][side].source;
    return `
      <div class="result-row">
        <div class="result-id">${escHtml(s.id)}</div>
        <div style="font-size:0.8rem;line-height:1.4">${escHtml(s.source)}</div>
        <div><span class="result-choice ${source}">${source}</span></div>
        <div style="font-size:0.8rem;line-height:1.4;color:${note ? 'var(--text)' : 'var(--muted)'}">${note ? escHtml(note) : '—'}</div>
      </div>`;
  }).join('');

  container.innerHTML = `
    <div class="section" style="text-align:center;padding:2rem 1.5rem;">
      <div style="font-size:0.72rem;font-weight:700;text-transform:uppercase;letter-spacing:0.1em;color:var(--muted);margin-bottom:1.25rem;">Overall translator preference</div>
      <div style="display:flex;align-items:center;justify-content:center;gap:2.5rem;flex-wrap:wrap;">
        <div>
          <div style="font-size:4.5rem;font-weight:700;color:var(--b);line-height:1">${pipelineWins}</div>
          <div style="font-size:0.85rem;color:var(--muted);margin-top:0.35rem">${META.label_b}</div>
        </div>
        <div style="font-size:1.5rem;color:var(--border);font-weight:300">vs</div>
        <div>
          <div style="font-size:4.5rem;font-weight:700;color:var(--muted);line-height:1">${basicWins}</div>
          <div style="font-size:0.85rem;color:var(--muted);margin-top:0.35rem">${META.label_a}</div>
        </div>
        ${neitherCount > 0 ? `
        <div>
          <div style="font-size:4.5rem;font-weight:700;color:#94a3b8;line-height:1">${neitherCount}</div>
          <div style="font-size:0.85rem;color:var(--muted);margin-top:0.35rem">Neither</div>
        </div>` : ''}
      </div>
    </div>

    <div class="section">
      <div class="results-header">
        <div>ID</div>
        <div>Source</div>
        <div>Chosen</div>
        <div>Note</div>
      </div>
      ${tableRows}
    </div>

    <div class="section">
      <h2 style="text-align:center;">How this worked</h2>
      <div style="display:flex;align-items:center;justify-content:center;gap:0.4rem;margin-top:0.75rem;overflow-x:auto;">
        ${[
          ['Engage Translators',     '#2563eb'],
          ['Create Custom Prompt',   '#2563eb'],
          ['Create Dataset to Test', '#2563eb'],
          ['Present A-B Output',     '#2563eb'],
          ['Analyse Results',        '#2563eb'],
          ['Iterate',                '#2563eb'],
        ].map(([title, color], i, arr) => `
          <div style="background:${color}18;border:1.5px solid ${color}55;border-radius:8px;padding:0.55rem 0;width:210px;text-align:center;white-space:nowrap;font-size:0.82rem;font-weight:600;color:${color};">${title}</div>
          ${i < arr.length - 1 ? `<div style="color:#cbd5e0;font-size:1rem;flex-shrink:0;">&rarr;</div>` : ''}
        `).join('')}
      </div>
    </div>`;
}
```

---

## Task 5: Update the hash decoder and STRINGS data in `french.html`

**Files:**
- Modify: `french.html`

- [ ] **Step 5.1: Replace the hash decoder in the init block**

Find:
```js
// Decode choices from URL hash if present (set by translator's "Copy results link")
if (location.hash) {
  try {
    const decoded = JSON.parse(atob(location.hash.slice(1)));
    // decoded is { id: 'basic'|'pipeline' } — reverse-map to a/b for this session
    Object.keys(decoded).forEach(id => {
      const source = decoded[id];
      const side = Object.keys(sides[id]).find(s => sides[id][s].source === source);
      if (side) choices[id] = side;
    });
  } catch(e) {}
}
```

Replace with:
```js
// Decode choices from URL hash if present (set by translator's "Copy results link")
if (location.hash) {
  try {
    const decoded = JSON.parse(decodeURIComponent(escape(atob(location.hash.slice(1)))));
    Object.keys(decoded).forEach(id => {
      if (!sides[id]) return;
      const val = decoded[id];
      if (typeof val === 'string') {
        // Old format: { id: "basic"|"pipeline" }
        const side = Object.keys(sides[id]).find(s => sides[id][s].source === val);
        if (side) choices[id] = { side, note: '' };
      } else {
        // New format: { source: "basic"|"pipeline"|"neither", note: "..." }
        const { source, note } = val;
        if (source === 'neither') {
          choices[id] = { side: 'neither', note: note || '' };
        } else {
          const side = Object.keys(sides[id]).find(s => sides[id][s].source === source);
          if (side) choices[id] = { side, note: note || '' };
        }
      }
    });
  } catch(e) {}
}
```

- [ ] **Step 5.2: Replace the STRINGS array with XLSX data**

Find the entire `const STRINGS = [` block (from `const STRINGS = [` through the closing `];`) and replace with the French data below. Remove the `"rule"` field — it no longer exists:

```js
const STRINGS = [
  {"id": "FR_B001", "source": "For God so loved the world that he gave his one and only Son, that whoever believes in him shall not perish but have eternal life. (John 3:16)", "translation_a": "Car Dieu a tant aimé le monde qu\u2019il a donné son Fils unique, afin que quiconque croit en lui ne périsse point, mais qu\u2019il ait la vie éternelle. (Jean 3:16)", "translation_b": "Oui, Dieu a tant aimé le monde qu'il a donné son Fils, son unique, pour que tous ceux qui placent leur confiance en lui échappent à la perdition et qu'ils aient la vie éternelle. (Jean 3:16)"},
  {"id": "FR_B002", "source": "I can do all this through him who gives me strength. (Philippians 4:13)", "translation_a": "Je puis tout par celui qui me fortifie. (Philippiens 4:13)", "translation_b": "Je peux tout, grâce à celui qui me fortifie. (Philippiens 4:13)"},
  {"id": "FR_B003", "source": "The Lord is my shepherd, I lack nothing. (Psalm 23:1)", "translation_a": "L\u2019Éternel est mon berger : je ne manquerai de rien. (Psaume 23:1)", "translation_b": "L'Éternel est mon berger, je ne manquerai de rien. (Psaume 23:1)"},
  {"id": "FR_B004", "source": "And we know that in all things God works for the good of those who love him. (Romans 8:28)", "translation_a": "Nous savons, du reste, que toutes choses concourent au bien de ceux qui aiment Dieu. (Romains 8:28)", "translation_b": "Du reste, nous savons que Dieu fait concourir toutes choses au bien de ceux qui l'aiment. (Romains 8:28)"},
  {"id": "FR_B005", "source": "Come to me, all you who are weary and burdened, and I will give you rest. (Matthew 11:28)", "translation_a": "Venez à moi, vous tous qui êtes fatigués et chargés, et je vous donnerai du repos. (Matthieu 11:28)", "translation_b": "Venez à moi, vous tous qui êtes accablés sous le poids d'un lourd fardeau, et je vous donnerai du repos. (Matthieu 11:28)"},
  {"id": "FR_T001", "source": "What if you were the answer someone has been waiting for?", "translation_a": "Et si vous étiez la réponse que quelqu\u2019un attendait ?", "translation_b": "Et si c'était toi la réponse que quelqu'un attend depuis longtemps ?"},
  {"id": "FR_T002", "source": "You are not alone in this.", "translation_a": "Vous n\u2019êtes pas seul dans cette épreuve.", "translation_b": "Vous n'êtes pas seuls dans tout ça."},
  {"id": "FR_T003", "source": "Have you ever felt like you didn't belong?", "translation_a": "Avez-vous déjà eu l\u2019impression de ne pas être à votre place ?", "translation_b": "Tu as déjà eu l'impression de ne pas être à ta place ?"},
  {"id": "FR_T004", "source": "Start a conversation today.", "translation_a": "Commencez une conversation aujourd\u2019hui.", "translation_b": "Lance la conversation aujourd'hui."},
  {"id": "FR_T005", "source": "We're so glad you're here.", "translation_a": "Nous sommes si heureux que vous soyez là.", "translation_b": "On est tellement contents que tu sois là."},
  {"id": "FR_I001", "source": "You bring it up and... crickets.", "translation_a": "Vous abordez le sujet et... silence radio.", "translation_b": "Tu en parles et... c'est le calme plat."},
  {"id": "FR_I002", "source": "Move past smalltalk.", "translation_a": "Allez au-delà des banalités.", "translation_b": "Dépasse les conversations de la pluie et du beau temps."},
  {"id": "FR_I003", "source": "Word of mouth still works.", "translation_a": "Le bouche-à-oreille fonctionne toujours.", "translation_b": "Le bouche à oreille, ça marche encore."},
  {"id": "FR_I004", "source": "It doesn't have to be a big deal.", "translation_a": "Pas besoin d'en faire toute une histoire.", "translation_b": "Pas besoin d'en faire tout un plat."},
  {"id": "FR_S001", "source": "Drop a 🙋", "translation_a": "Laissez un 🙋", "translation_b": "Mets un 🙋 en commentaire"},
  {"id": "FR_S002", "source": "Tag someone who needs this.", "translation_a": "Identifiez quelqu\u2019un qui a besoin de voir ça.", "translation_b": "Tague quelqu'un qui a besoin de lire ça."},
  {"id": "FR_S003", "source": "Share this.", "translation_a": "Partagez ceci.", "translation_b": "Partage ça."},
  {"id": "FR_G001", "source": "Tag a friend who needs to hear this.", "translation_a": "Identifiez un ami qui a besoin d\u2019entendre cela.", "translation_b": "Tague un ami qui a besoin d'entendre ça."},
  {"id": "FR_G002", "source": "Invite someone to check out the yesHEis App.", "translation_a": "Invitez quelqu\u2019un à découvrir l\u2019application yesHEis.", "translation_b": "Invite quelqu'un à découvrir la yesHEis App."},
  {"id": "FR_G003", "source": "Share this with a neighbour.", "translation_a": "Partagez ceci avec un voisin.", "translation_b": "Partage ça avec un voisin."},
  {"id": "FR_G004", "source": "Find someone who is searching for answers.", "translation_a": "Trouvez quelqu\u2019un qui cherche des réponses.", "translation_b": "Trouve quelqu'un qui cherche des réponses."},
  {"id": "FR_C001", "source": "Tag a friend who needs to hear this. \"Come to me, all you who are weary and burdened, and I will give you rest.\" Matthew 11:28", "translation_a": "Identifiez un ami qui a besoin d\u2019entendre cela. \"Venez à moi, vous tous qui êtes fatigués et chargés, et je vous donnerai du repos.\" Matthieu 11:28", "translation_b": "Tague un ami qui a besoin d'entendre ça. « Venez à moi, vous tous qui êtes accablés sous le poids d'un lourd fardeau, et je vous donnerai du repos. » Matthieu 11:28"},
  {"id": "FR_C002", "source": "You bring it up and... crickets. Start the conversation. Word of mouth still works.", "translation_a": "Vous abordez le sujet et... silence radio. Lancez la conversation. Le bouche-à-oreille fonctionne toujours.", "translation_b": "Tu en parles et... c'est le calme plat. Lance la conversation. Le bouche à oreille, ça marche encore."},
  {"id": "FR_C003", "source": "Download the yesHEis App 📲 Tag a friend who needs this. #follow", "translation_a": "Téléchargez l\u2019application yesHEis 📲 Identifiez un ami qui a besoin de cela. #follow", "translation_b": "Télécharge la yesHEis App 📲 Tague un ami qui en a besoin. #follow"},
  {"id": "FR_C004", "source": "Have you ever felt like you didn't belong? You are not alone in this. \"And we know that in all things God works for the good of those who love him.\" Romans 8:28", "translation_a": "Vous est-il déjà arrivé de vous sentir à votre place ? Vous n\u2019êtes pas seul dans cette situation. \"Et nous savons que, dans toutes choses, Dieu travaille pour le bien de ceux qui l\u2019aiment.\" Romains 8:28", "translation_b": "Vous avez déjà eu l'impression de ne pas être à votre place ? Vous n'êtes pas seuls dans tout ça. « Nous savons, du reste, que Dieu fait concourir toutes choses au bien de ceux qui l'aiment. » Romains 8:28"},
  {"id": "FR_C005", "source": "What if you were the answer someone has been waiting for? Find a friend and start that conversation today.", "translation_a": "Et si vous étiez la réponse que quelqu\u2019un attendait ? Trouvez un ami et commencez cette conversation aujourd\u2019hui.", "translation_b": "Et si tu étais la réponse que quelqu'un attendait ? Trouve un ami et lance cette conversation aujourd'hui."},
  {"id": "FR_C006", "source": "Sarah shared this with you. 🙌", "translation_a": "Sarah a partagé ceci avec vous. 🙌", "translation_b": "Sarah a partagé ça avec toi. 🙌"},
  {"id": "FR_C007", "source": "You have 3 friends who use the yesHEis App.", "translation_a": "Vous avez 3 amis qui utilisent l\u2019application yesHEis.", "translation_b": "Tu as 3 amis qui utilisent la yesHEis App."},
];
```

- [ ] **Step 5.3: Update the page `<title>` and header**

Find:
```html
<title>Translation Review — French Prompt Comparison</title>
```
Replace with:
```html
<title>Translation Review — French</title>
```

Find:
```html
      <h1>French Translation Review</h1>
```
Confirm it already reads "French Translation Review" — leave as-is.

- [ ] **Step 5.4: Commit french.html**

```bash
git add french.html
git commit -m "feat: french.html — Neither option, note field, updated data from XLSX"
```

---

## Task 6: Create `spanish.html`

**Files:**
- Create: `spanish.html`

- [ ] **Step 6.1: Copy french.html as the base**

```bash
cp french.html spanish.html
```

- [ ] **Step 6.2: Update META block**

In `spanish.html`, find:
```js
const META = {
  language: "French",
  date: "2026-03-24",
  label_a: "Current workflow",
  label_b: "Customised French prompt"
};
```

Replace with:
```js
const META = {
  language: "Spanish",
  date: "2026-04-02",
  label_a: "Current workflow",
  label_b: "Customised Spanish prompt"
};
```

- [ ] **Step 6.3: Replace STRINGS with Spanish data**

Find the entire `const STRINGS = [` block and replace with:

```js
const STRINGS = [
  {"id": "ES_B001", "source": "For God so loved the world that he gave his one and only Son, that whoever believes in him shall not perish but have eternal life. (John 3:16)", "translation_a": "Porque tanto amó Dios al mundo que dio a su Hijo unigénito, para que todo el que cree en él no se pierda, sino que tenga vida eterna. (Juan 3:16)", "translation_b": "Porque tanto amó Dios al mundo que dio a su Hijo unigénito, para que todo el que cree en él no perezca, sino que tenga vida eterna. (Juan 3:16)"},
  {"id": "ES_B002", "source": "I can do all this through him who gives me strength. (Philippians 4:13)", "translation_a": "Todo lo puedo en Cristo que me fortalece. (Filipenses 4:13)", "translation_b": "Todo esto lo puedo en aquel que me da fuerzas. (Filipenses 4:13)"},
  {"id": "ES_B003", "source": "The Lord is my shepherd, I lack nothing. (Psalm 23:1)", "translation_a": "El Señor es mi pastor, nada me falta. (Salmo 23:1)", "translation_b": "El Señor es mi pastor, nada me falta. (Salmo 23:1)"},
  {"id": "ES_B004", "source": "And we know that in all things God works for the good of those who love him. (Romans 8:28)", "translation_a": "Y sabemos que en todas las cosas Dios obra para el bien de quienes lo aman. (Romanos 8:28)", "translation_b": "Y sabemos que en todas las cosas Dios obra para el bien de los que lo aman. (Romanos 8:28)"},
  {"id": "ES_B005", "source": "Come to me, all you who are weary and burdened, and I will give you rest. (Matthew 11:28)", "translation_a": "Vengan a mí todos ustedes que están cansados y agobiados, y yo les daré descanso. (Mateo 11:28)", "translation_b": "Venid a mí, todos los que estáis cansados y agobiados, y yo os daré descanso. (Mateo 11:28)"},
  {"id": "ES_T001", "source": "What if you were the answer someone has been waiting for?", "translation_a": "¿Y si fueras la respuesta que alguien ha estado esperando?", "translation_b": "¿Y si fueras la respuesta que alguien ha estado esperando?"},
  {"id": "ES_T002", "source": "You are not alone in this.", "translation_a": "No estás solo en esto.", "translation_b": "No estás solo en esto."},
  {"id": "ES_T003", "source": "Have you ever felt like you didn't belong?", "translation_a": "¿Alguna vez has sentido que no encajabas?", "translation_b": "¿Alguna vez has sentido que no encajabas?"},
  {"id": "ES_T004", "source": "Start a conversation today.", "translation_a": "Empieza una conversación hoy.", "translation_b": "Comienza una conversación hoy."},
  {"id": "ES_T005", "source": "We're so glad you're here.", "translation_a": "Nos alegra mucho que estés aquí.", "translation_b": "Estamos muy contentos de que estés aquí."},
  {"id": "ES_I001", "source": "You bring it up and... crickets.", "translation_a": "Sacas el tema y... grillos.", "translation_b": "Sacas el tema y... silencio total."},
  {"id": "ES_I002", "source": "Move past smalltalk.", "translation_a": "Ve más allá de la charla superficial.", "translation_b": "Deja atrás la charla superficial."},
  {"id": "ES_I003", "source": "Word of mouth still works.", "translation_a": "El boca a boca todavía funciona.", "translation_b": "El boca a boca sigue funcionando."},
  {"id": "ES_I004", "source": "It doesn't have to be a big deal.", "translation_a": "No tiene que ser algo enorme.", "translation_b": "No tiene por qué ser para tanto."},
  {"id": "ES_S001", "source": "Drop a 🙋", "translation_a": "Deja un 🙋", "translation_b": "Deja un 🙋"},
  {"id": "ES_S002", "source": "Tag someone who needs this.", "translation_a": "Etiqueta a alguien que necesite esto.", "translation_b": "Etiqueta a alguien que necesita esto."},
  {"id": "ES_S003", "source": "Share this.", "translation_a": "Comparte esto.", "translation_b": "Comparte esto."},
  {"id": "ES_G001", "source": "Tag a friend who needs to hear this.", "translation_a": "Etiqueta a un amigo que necesite escuchar esto.", "translation_b": "Etiqueta a un amigo que necesita escuchar esto."},
  {"id": "ES_G002", "source": "Invite someone to check out the yesHEis App.", "translation_a": "Invita a alguien a conocer la app yesHEis.", "translation_b": "Invita a alguien a conocer la App de yesHEis."},
  {"id": "ES_G003", "source": "Share this with a neighbour.", "translation_a": "Comparte esto con un vecino.", "translation_b": "Comparte esto con un vecino."},
  {"id": "ES_G004", "source": "Find someone who is searching for answers.", "translation_a": "Encuentra a alguien que esté buscando respuestas.", "translation_b": "Encuentra a alguien que busque respuestas."},
  {"id": "ES_C001", "source": "Tag a friend who needs to hear this. \"Come to me, all you who are weary and burdened, and I will give you rest.\" Matthew 11:28", "translation_a": "Etiqueta a un amigo que necesite escuchar esto. \"Vengan a mí todos ustedes que están cansados y agobiados, y yo les daré descanso.\" Mateo 11:28", "translation_b": "Etiqueta a un amigo que necesita escuchar esto. \"Venid a mí todos los que estáis cansados y agobiados, y yo os daré descanso.\" Mateo 11:28"},
  {"id": "ES_C002", "source": "You bring it up and... crickets. Start the conversation. Word of mouth still works.", "translation_a": "Sacas el tema y... grillos. Empieza la conversación. El boca a boca todavía funciona.", "translation_b": "Sacas el tema y... silencio total. Comienza la conversación. El boca a boca sigue funcionando."},
  {"id": "ES_C003", "source": "Download the yesHEis App 📲 Tag a friend who needs this. #follow", "translation_a": "Descarga la app yesHEis 📲 Etiqueta a un amigo que necesite esto. #follow", "translation_b": "Descarga la App de yesHEis 📲 Etiqueta a un amigo que necesita esto. #follow"},
  {"id": "ES_C004", "source": "Have you ever felt like you didn't belong? You are not alone in this. \"And we know that in all things God works for the good of those who love him.\" Romans 8:28", "translation_a": "¿Alguna vez has sentido que no encajabas? No estás solo en esto. \"Y sabemos que en todas las cosas Dios obra para el bien de quienes lo aman.\" Romanos 8:28", "translation_b": "¿Alguna vez has sentido que no encajabas? No estás solo en esto. \"Y sabemos que en todas las cosas Dios obra para el bien de los que lo aman.\" Romanos 8:28"},
  {"id": "ES_C005", "source": "What if you were the answer someone has been waiting for? Find a friend and start that conversation today.", "translation_a": "¿Y si tú fueras la respuesta que alguien ha estado esperando? Busca a un amigo y empieza esa conversación hoy.", "translation_b": "¿Y si fueras la respuesta que alguien ha estado esperando? Encuentra a un amigo y comienza esa conversación hoy."},
  {"id": "ES_C006", "source": "Sarah shared this with you. 🙌", "translation_a": "Sarah compartió esto contigo. 🙌", "translation_b": "Sarah compartió esto contigo. 🙌"},
  {"id": "ES_C007", "source": "You have 3 friends who use the yesHEis App.", "translation_a": "Tienes 3 amigos que usan la app yesHEis.", "translation_b": "Tienes 3 amigos que usan la App de yesHEis."},
];
```

- [ ] **Step 6.4: Update title and header**

Find:
```html
<title>Translation Review — French</title>
```
Replace with:
```html
<title>Translation Review — Spanish</title>
```

Find:
```html
      <h1>French Translation Review</h1>
```
Replace with:
```html
      <h1>Spanish Translation Review</h1>
```

- [ ] **Step 6.5: Commit spanish.html**

```bash
git add spanish.html
git commit -m "feat: add spanish.html with ES_ strings from XLSX"
```

---

## Task 7: Create `german.html`

**Files:**
- Create: `german.html`

- [ ] **Step 7.1: Copy french.html as the base**

```bash
cp french.html german.html
```

- [ ] **Step 7.2: Update META block**

Find:
```js
const META = {
  language: "French",
  date: "2026-03-24",
  label_a: "Current workflow",
  label_b: "Customised French prompt"
};
```

Replace with:
```js
const META = {
  language: "German",
  date: "2026-04-02",
  label_a: "Current workflow",
  label_b: "Customised German prompt"
};
```

- [ ] **Step 7.3: Replace STRINGS with German data**

Find the entire `const STRINGS = [` block and replace with:

```js
const STRINGS = [
  {"id": "DE_B001", "source": "For God so loved the world that he gave his one and only Son, that whoever believes in him shall not perish but have eternal life. (John 3:16)", "translation_a": "Denn Gott hat die Welt so sehr geliebt, dass er seinen einzigen Sohn gab, damit jeder, der an ihn glaubt, nicht verloren geht, sondern ewiges Leben hat. (Johannes 3,16)", "translation_b": "Denn so sehr hat Gott die Welt geliebt, dass er seinen einzigen Sohn gab, damit jeder, der an ihn glaubt, nicht verloren geht, sondern ewiges Leben hat. (Johannes 3:16)"},
  {"id": "DE_B002", "source": "I can do all this through him who gives me strength. (Philippians 4:13)", "translation_a": "Ich vermag alles durch den, der mir Kraft gibt. (Philipper 4,13)", "translation_b": "Ich kann das alles durch ihn, der mir Kraft gibt. (Philipper 4:13)"},
  {"id": "DE_B003", "source": "The Lord is my shepherd, I lack nothing. (Psalm 23:1)", "translation_a": "Der Herr ist mein Hirte, mir fehlt nichts. (Psalm 23,1)", "translation_b": "Der Herr ist mein Hirte, mir fehlt nichts. (Psalm 23:1)"},
  {"id": "DE_B004", "source": "And we know that in all things God works for the good of those who love him. (Romans 8:28)", "translation_a": "Wir wissen aber, dass denen, die Gott lieben, alle Dinge zum Besten dienen. (Römer 8,28)", "translation_b": "Und wir wissen, dass Gott alles für die, die ihn lieben, zum Guten führt. (Römer 8:28)"},
  {"id": "DE_B005", "source": "Come to me, all you who are weary and burdened, and I will give you rest. (Matthew 11:28)", "translation_a": "Kommt her zu mir, alle, die ihr mühselig und beladen seid; ich will euch erquicken. (Matthäus 11,28)", "translation_b": "Kommt zu mir, alle, die ihr müde und beladen seid, und ich werde euch Ruhe geben. (Matthäus 11:28)"},
  {"id": "DE_T001", "source": "What if you were the answer someone has been waiting for?", "translation_a": "Was wäre, wenn du die Antwort wärst, auf die jemand gewartet hat?", "translation_b": "Was, wenn du die Antwort wärst, auf die jemand gewartet hat?"},
  {"id": "DE_T002", "source": "You are not alone in this.", "translation_a": "Du bist damit nicht allein.", "translation_b": "Du bist nicht allein damit."},
  {"id": "DE_T003", "source": "Have you ever felt like you didn't belong?", "translation_a": "Hast du dich jemals so gefühlt, als würdest du nicht dazugehören?", "translation_b": "Hattest du schon mal das Gefühl, nicht dazuzugehören?"},
  {"id": "DE_T004", "source": "Start a conversation today.", "translation_a": "Beginne noch heute ein Gespräch.", "translation_b": "Fang noch heute ein Gespräch an."},
  {"id": "DE_T005", "source": "We're so glad you're here.", "translation_a": "Wir freuen uns so, dass du hier bist.", "translation_b": "Wie schön, dass du da bist."},
  {"id": "DE_I001", "source": "You bring it up and... crickets.", "translation_a": "Du sprichst es an und ... Grillen zirpen.", "translation_b": "Du sprichst es an und ... Stille."},
  {"id": "DE_I002", "source": "Move past smalltalk.", "translation_a": "Geh über Smalltalk hinaus.", "translation_b": "Geh über Smalltalk hinaus."},
  {"id": "DE_I003", "source": "Word of mouth still works.", "translation_a": "Mundpropaganda funktioniert immer noch.", "translation_b": "Mundpropaganda funktioniert immer noch."},
  {"id": "DE_I004", "source": "It doesn't have to be a big deal.", "translation_a": "Es muss keine große Sache sein.", "translation_b": "Es muss nichts Großes sein."},
  {"id": "DE_S001", "source": "Drop a 🙋", "translation_a": "Schick ein 🙋", "translation_b": "Lass ein 🙋 da"},
  {"id": "DE_S002", "source": "Tag someone who needs this.", "translation_a": "Markiere jemanden, der das braucht.", "translation_b": "Markiere jemanden, der das braucht."},
  {"id": "DE_S003", "source": "Share this.", "translation_a": "Teile das.", "translation_b": "Teile das."},
  {"id": "DE_G001", "source": "Tag a friend who needs to hear this.", "translation_a": "Markiere einen Freund, der das hören muss.", "translation_b": "Markiere einen Freund, der das hören muss."},
  {"id": "DE_G002", "source": "Invite someone to check out the yesHEis App.", "translation_a": "Lade jemanden ein, sich die yesHEis App anzusehen.", "translation_b": "Lade jemanden ein, sich die yesHEis App anzusehen."},
  {"id": "DE_G003", "source": "Share this with a neighbour.", "translation_a": "Teile das mit einem Nachbarn.", "translation_b": "Teile das mit einem Nachbarn."},
  {"id": "DE_G004", "source": "Find someone who is searching for answers.", "translation_a": "Finde jemanden, der nach Antworten sucht.", "translation_b": "Finde jemanden, der nach Antworten sucht."},
  {"id": "DE_C001", "source": "Tag a friend who needs to hear this. \"Come to me, all you who are weary and burdened, and I will give you rest.\" Matthew 11:28", "translation_a": "Markiere einen Freund, der das hören muss. \"Kommt her zu mir, alle, die ihr mühselig und beladen seid; ich will euch erquicken.\" Matthäus 11,28", "translation_b": "Markiere einen Freund, der das hören muss. \"Kommt zu mir, alle, die ihr müde und belastet seid, und ich will euch Ruhe geben.\" Matthäus 11:28"},
  {"id": "DE_C002", "source": "You bring it up and... crickets. Start the conversation. Word of mouth still works.", "translation_a": "Du sprichst es an und ... Grillen zirpen. Fang das Gespräch an. Mundpropaganda funktioniert immer noch.", "translation_b": "Du sprichst es an und ... Stille. Fang das Gespräch an. Mundpropaganda funktioniert immer noch."},
  {"id": "DE_C003", "source": "Download the yesHEis App 📲 Tag a friend who needs this. #follow", "translation_a": "Lade die yesHEis App herunter 📲 Markiere einen Freund, der das braucht. #follow", "translation_b": "Lade die yesHEis App herunter 📲 Markiere einen Freund, der das braucht. #follow"},
  {"id": "DE_C004", "source": "Have you ever felt like you didn't belong? You are not alone in this. \"And we know that in all things God works for the good of those who love him.\" Romans 8:28", "translation_a": "Hast du dich jemals gefühlt, als würdest du nicht dazugehören? Du bist damit nicht allein. \"Wir wissen aber, dass denen, die Gott lieben, alle Dinge zum Besten dienen.\" Römer 8,28", "translation_b": "Hast du dich jemals so gefühlt, als würdest du nicht dazugehören? Du bist damit nicht allein. \"Und wir wissen, dass Gott für alle, die ihn lieben, in allem zum Guten wirkt.\" Römer 8:28"},
  {"id": "DE_C005", "source": "What if you were the answer someone has been waiting for? Find a friend and start that conversation today.", "translation_a": "Was wäre, wenn du die Antwort wärst, auf die jemand gewartet hat? Finde einen Freund und beginne dieses Gespräch noch heute.", "translation_b": "Was, wenn du die Antwort wärst, auf die jemand gewartet hat? Such dir einen Freund und fang das Gespräch noch heute an."},
  {"id": "DE_C006", "source": "Sarah shared this with you. 🙌", "translation_a": "Sarah hat das mit dir geteilt. 🙌", "translation_b": "Sarah hat das mit dir geteilt. 🙌"},
  {"id": "DE_C007", "source": "You have 3 friends who use the yesHEis App.", "translation_a": "Du hast 3 Freunde, die die yesHEis App nutzen.", "translation_b": "Du hast 3 Freunde, die die yesHEis App nutzen."},
];
```

- [ ] **Step 7.4: Update title and header**

Find:
```html
<title>Translation Review — French</title>
```
Replace with:
```html
<title>Translation Review — German</title>
```

Find:
```html
      <h1>French Translation Review</h1>
```
Replace with:
```html
      <h1>German Translation Review</h1>
```

- [ ] **Step 7.5: Commit german.html**

```bash
git add german.html
git commit -m "feat: add german.html with DE_ strings from XLSX"
```

---

## Task 8: Cleanup

**Files:**
- Delete: `index.html`
- Modify: `.gitignore`

- [ ] **Step 8.1: Add .superpowers to .gitignore**

Check if `.gitignore` exists. If not, create it. Add:
```
.superpowers/
```

- [ ] **Step 8.2: Remove index.html**

```bash
git rm index.html
```

- [ ] **Step 8.3: Final commit**

```bash
git add .gitignore
git commit -m "chore: remove index.html (replaced by french.html), ignore .superpowers/"
```

---

## Verification Checklist

After all tasks, open each file in a browser and confirm:

**Translator view (`french.html`, `spanish.html`, `german.html`):**
- [ ] Cards show A, B, and "Neither" as three equal columns
- [ ] Selecting A or B highlights it; Neither highlights in grey
- [ ] A note textarea appears after any selection
- [ ] Switching A→B preserves the note; switching to/from Neither clears it
- [ ] Progress bar updates correctly
- [ ] "Copy results link" works (copies a URL with `?tech#...`)
- [ ] No rule tags visible anywhere on cards

**Tech Summary (open the results link):**
- [ ] Overall score shows pipeline wins vs basic wins (Neither shown separately if any)
- [ ] Results table shows ID / Source / Chosen / Note columns
- [ ] Notes appear in the Note column
- [ ] No rule breakdown section
- [ ] Old-format results links (without notes) still decode correctly
