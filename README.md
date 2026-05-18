# DevRadar — Necto Mono Theme · Full Implementation Prompt

> Copy-paste this entire document into Claude (or your AI of choice) and say:
> **"Implement this theme system across my DevRadar React + Tailwind codebase."**
> It covers every file you need to touch — config, globals, and all 6 components.

---

## CONTEXT — what this codebase is

- **Framework:** React 18 + Vite
- **Styling:** Tailwind CSS (utility-first)
- **Components to retheme:**
  - `src/App.jsx` — root, nav bar, layout shell
  - `src/components/StackInput.jsx` — skill chip selector + name input
  - `src/components/Dashboard.jsx` — tab shell + stat cards
  - `src/components/StartupCard.jsx` — startup match card with score bar
  - `src/components/HackathonCard.jsx` — hackathon listing with deadline badge
  - `src/components/GapAnalysis.jsx` — Claude AI panel + skills chart
  - `src/components/MemoryBadge.jsx` — HydraDB session indicator in nav
- **Font:** monospace everywhere — use `'JetBrains Mono', 'Courier New', monospace`
- **Theme name:** Necto Mono (dark, monospace, lime accent)

---

## STEP 1 — Install the font

Add to `index.html` inside `<head>`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
```

---

## STEP 2 — `tailwind.config.js`

Replace the entire config with this:

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {
      fontFamily: {
        mono: ['"JetBrains Mono"', '"Courier New"', 'monospace'],
      },
      colors: {
        // ── Necto Mono palette ──────────────────────
        bg:      '#0f0f0f',   // page canvas
        s1:      '#1c1c1c',   // card surface
        s2:      '#252525',   // raised / input bg
        s3:      '#2e2e2e',   // progress track, dividers
        // Text
        'txt':   '#f0f0ee',   // primary text
        'muted': '#888880',   // secondary text, labels
        'dim':   '#555550',   // placeholders, disabled
        // Accent
        lime:    '#d4f53c',   // primary accent — CTAs, top scores, active chips
        'lime-bg':   '#1a2200',  // lime tinted background
        'lime-dim':  '#8aaa18',  // lime text on lime-bg
        // Semantic
        warn:    '#ea9d34',   // mid scores, deadlines
        'warn-bg':   '#2a1500',
        err:     '#e08080',   // low scores, critical gaps
        'err-bg':    '#2a0808',
        // Borders
        'bd':    'rgba(255,255,255,0.07)',
        'bd2':   'rgba(255,255,255,0.12)',
      },
      borderRadius: {
        card: '16px',
        pill: '999px',
        btn:  '30px',
      },
      fontSize: {
        'label': ['10px', { letterSpacing: '0.1em', textTransform: 'uppercase' }],
      },
    },
  },
  plugins: [],
}
```

---

## STEP 3 — `src/index.css`

Replace entirely:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ── Global resets ───────────────────────────── */
*, *::before, *::after { box-sizing: border-box; }

html, body, #root {
  background-color: #0f0f0f;
  color: #f0f0ee;
  font-family: 'JetBrains Mono', 'Courier New', monospace;
  min-height: 100vh;
}

/* ── Scrollbar ───────────────────────────────── */
::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: #1c1c1c; }
::-webkit-scrollbar-thumb { background: #2e2e2e; border-radius: 3px; }
::-webkit-scrollbar-thumb:hover { background: #3a3a3a; }

/* ── Input reset ─────────────────────────────── */
input, button, select, textarea {
  font-family: inherit;
  outline: none;
}

input:focus {
  border-color: #d4f53c !important;
}

input::placeholder {
  color: #555550;
}

/* ── Transition defaults ─────────────────────── */
.card-hover {
  transition: border-color 0.15s ease;
}
.card-hover:hover {
  border-color: rgba(255,255,255,0.18) !important;
}

/* ── Score bar animation ─────────────────────── */
.bar-fill {
  transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

/* ── Chip transition ─────────────────────────── */
.chip-toggle {
  transition: all 0.15s ease;
}
```

---

## STEP 4 — Design tokens (quick reference card)

| Token | Value | Use |
|---|---|---|
| `bg-bg` | `#0f0f0f` | Page background |
| `bg-s1` | `#1c1c1c` | All cards, nav |
| `bg-s2` | `#252525` | Inputs, raised panels |
| `bg-s3` | `#2e2e2e` | Progress tracks, dividers |
| `text-txt` | `#f0f0ee` | Primary text, headings |
| `text-muted` | `#888880` | Labels, metadata |
| `text-dim` | `#555550` | Placeholders |
| `bg-lime` / `text-lime` | `#d4f53c` | CTAs, top match, active |
| `bg-lime-bg` / `text-lime-dim` | `#1a2200` / `#8aaa18` | Tag fills |
| `text-warn` / `bg-warn-bg` | `#ea9d34` / `#2a1500` | Mid scores, urgency |
| `text-err` / `bg-err-bg` | `#e08080` / `#2a0808` | Low scores, gaps |
| `border-bd` | `rgba(255,255,255,0.07)` | All card borders |
| `border-bd2` | `rgba(255,255,255,0.12)` | Hover borders, ghost btns |

---

## STEP 5 — Component implementation

### `src/App.jsx` — Root shell + Nav

```jsx
// Nav bar classes:
// Outer nav:   bg-s1 border-b border-bd h-[52px] px-5 flex items-center justify-between
// Logo mark:   w-7 h-7 rounded-lg bg-lime flex items-center justify-center
//              text-[11px] font-bold text-bg
// Logo text:   text-[15px] font-bold text-txt tracking-tight font-mono
// Page shell:  min-h-screen bg-bg font-mono

// Example:
<nav className="bg-s1 border-b border-bd h-[52px] px-5 flex items-center justify-between">
  <div className="flex items-center gap-2.5">
    <div className="w-7 h-7 rounded-lg bg-lime flex items-center justify-center text-[11px] font-bold text-bg">
      DR
    </div>
    <span className="text-[15px] font-bold text-txt tracking-tight">devradar</span>
  </div>
  <MemoryBadge />
</nav>
```

---

### `src/components/MemoryBadge.jsx`

```jsx
// Active (HydraDB connected):
// bg-lime-bg text-lime border border-lime/20 rounded-pill px-3 py-1 text-[10px] font-bold tracking-widest
// Shows: ● HydraDB / on

// Inactive / guest:
// bg-s2 text-muted border border-bd rounded-pill px-3 py-1 text-[10px] font-bold tracking-widest
// Shows: ○ guest mode

export default function MemoryBadge({ active = true }) {
  return (
    <span className={`
      rounded-pill px-3 py-1 text-[10px] font-bold tracking-widest border
      ${active
        ? 'bg-lime-bg text-lime border-lime/20'
        : 'bg-s2 text-muted border-bd'}
    `}>
      {active ? '● HydraDB / on' : '○ guest mode'}
    </span>
  )
}
```

---

### `src/components/StackInput.jsx`

```jsx
// Section label pattern:  text-[10px] text-muted tracking-[0.1em] uppercase mb-2.5
// "// label //" style:    use this exact prefix + suffix pattern for all section labels

// Heading:    text-[26px] font-bold text-txt tracking-tight leading-[1.15] mb-3.5
// Body text:  text-[12px] text-muted leading-[1.75] mb-5

// Input wrapper:   w-full bg-s2 border border-bd rounded-[10px] px-3.5 py-2.5 text-[12px] text-txt
//                  focus:border-lime transition-colors

// Chip OFF:   bg-transparent text-muted border border-bd2 rounded-pill px-3 py-1.5
//             text-[11px] font-bold cursor-pointer chip-toggle
// Chip ON:    bg-lime text-bg border-lime rounded-pill px-3 py-1.5
//             text-[11px] font-bold cursor-pointer chip-toggle

// Primary button:  w-full bg-txt text-bg rounded-btn px-5 py-3 text-[12px] font-bold
//                  hover:opacity-85 transition-opacity cursor-pointer border-none
// Ghost button:    bg-transparent text-txt border-[1.5px] border-bd2 rounded-btn px-5 py-3
//                  text-[12px] font-bold hover:border-white/35 transition-colors

// Info blocks at bottom — two-column grid:
// grid grid-cols-2 gap-3 mt-3.5
// Each block: bg-s1 border border-bd rounded-card p-5

// Big stat number:  text-[40px] font-bold text-txt leading-none mb-1.5
// Big stat label:   text-[11px] text-muted leading-relaxed
```

---

### `src/components/Dashboard.jsx`

```jsx
// Return/memory banner:
// bg-s2 border border-lime/15 rounded-card p-4 mb-4 flex gap-2.5 items-start
// Text inside: text-[12px] text-muted leading-[1.75]
// Highlighted words: text-lime font-bold (Claude-recalled data, urgent deadlines)

// Stat cards row:  grid grid-cols-3 gap-2.5 mb-4
// Each stat card:  bg-s1 border border-bd rounded-[14px] p-3.5 text-center
// Stat number:     text-[26px] font-bold text-txt leading-none
// Stat label:      text-[9px] text-muted mt-1.5 tracking-[0.08em] uppercase

// Tab bar:
// flex border border-bd rounded-[12px] overflow-hidden mb-4.5
// Each tab:         flex-1 py-2.5 text-[11px] font-bold text-muted text-center
//                   cursor-pointer border-r border-bd last:border-r-0
//                   transition-all bg-transparent hover:bg-s2
// Active tab:       bg-s2 text-txt
```

---

### `src/components/StartupCard.jsx`

```jsx
// Card wrapper:
// bg-s1 border border-bd rounded-card p-[18px] mb-2.5 card-hover cursor-pointer

// Card header:  flex justify-between items-start mb-3
// Company name: text-[15px] font-bold text-txt tracking-tight
// Meta line:    text-[10px] text-muted mt-0.5 tracking-[0.02em]

// Score badge — HIGH (≥70%):
//   bg-lime text-bg rounded-pill px-2.5 py-1 text-[11px] font-bold
// Score badge — MID (40–69%):
//   bg-warn-bg text-warn border border-warn/30 rounded-pill px-2.5 py-1 text-[11px] font-bold
// Score badge — LOW (<40%):
//   bg-err-bg text-err border border-err/25 rounded-pill px-2.5 py-1 text-[11px] font-bold

// Progress track:  h-[3px] bg-s3 rounded-full overflow-hidden mb-1.5
// Progress fill — HIGH:  h-full bg-lime bar-fill
// Progress fill — MID:   h-full bg-warn bar-fill
// Progress fill — LOW:   h-full bg-err bar-fill

// Bar labels:  flex justify-between text-[9px] text-dim tracking-[0.04em] uppercase

// Skill tags row:  flex flex-wrap gap-1.5 mt-3
// Tag — known:     bg-lime-bg text-lime-dim border border-lime/15 rounded-pill
//                  px-2.5 py-1 text-[10px] font-bold
// Tag — gap:       bg-warn-bg text-warn border border-warn/20 rounded-pill
//                  px-2.5 py-1 text-[10px] font-bold  (append " ↑")
// Tag — big gap:   bg-err-bg text-err border border-err/20 rounded-pill
//                  px-2.5 py-1 text-[10px] font-bold  (append " ↑ gap")
```

---

### `src/components/HackathonCard.jsx`

```jsx
// Card:   bg-s1 border border-bd rounded-card px-[18px] py-3.5 mb-2.5 flex items-center gap-3.5
//         card-hover cursor-pointer
// URGENT card border override:  border-warn/30

// Date block:  min-w-[40px] text-center
// Day number:  text-[22px] font-bold text-txt leading-none
// Month:       text-[9px] text-muted uppercase tracking-[0.1em] mt-0.5

// Separator:   w-px bg-s3 self-stretch

// Info block:  flex-1 min-w-0
// Name:        text-[12px] font-bold text-txt
// Org line:    text-[10px] text-muted mt-0.5

// Right column:  flex flex-col items-end gap-1.5

// Urgency badge:
//   bg-[#1a1000] text-warn border border-warn/30 rounded-pill
//   px-2 py-1 text-[9px] font-bold tracking-[0.06em] uppercase
//   Prefix with: ⚡

// Match score badge:
//   same as StartupCard score badge (HIGH/MID/LOW) but text-[10px] px-2 py-1
```

---

### `src/components/GapAnalysis.jsx`

```jsx
// Claude recall box:
// bg-s2 border border-lime/12 rounded-[12px] p-3.5 mb-3.5
// Label:   text-[10px] font-bold text-lime tracking-[0.1em] uppercase mb-2
//          Prefix: "● claude · context-aware"
// Text:    text-[11px] text-muted leading-[1.75]
// Bold highlights inside: text-txt font-bold (company names, skill names)

// Chart card wrapper:
// bg-s1 border border-bd rounded-card p-[18px] mb-3

// Chart section label:
// text-[10px] text-muted tracking-[0.1em] uppercase mb-4
// Pattern: "// skill gap analysis //"

// ── Bar chart (startup match scores) ──────────
// Container:   flex items-end gap-2 h-[110px] pb-5 relative
// Each bar column: flex flex-col items-center flex-1 justify-end relative
// Bar fill:    rounded-t-[4px] w-full  (height set dynamically as inline style)
//   HIGH bars:  bg-lime
//   MID bars:   bg-warn
//   LOW bars:   bg-err
// Val above bar:  text-[10px] font-bold absolute top-[-18px]
//   HIGH: text-lime  |  MID: text-warn  |  LOW: text-err
// Label below bar: text-[9px] text-muted text-center absolute bottom-[-18px]
//                  w-full overflow-hidden text-ellipsis whitespace-nowrap

// ── Horizontal skill bars (learn priority) ────
// Row:          flex items-center gap-2.5 py-2.5 border-b border-bd last:border-b-0
// Skill name:   text-[11px] font-bold text-txt min-w-[100px] tracking-[0.01em]
// Track:        flex-1 h-1 bg-s3 rounded-full overflow-hidden
// Fill:         h-full rounded-full
//   Priority 1: bg-lime
//   Priority 2: bg-lime/60
//   Priority 3: bg-warn
//   Priority 4: bg-err
// Time label:   text-[10px] text-muted
// Salary badge: text-[10px] font-bold rounded-[10px] px-2 py-0.5
//   Priority 1: bg-lime-bg text-lime
//   Priority 2: bg-lime-bg text-lime-dim
//   Priority 3: bg-warn-bg text-warn
//   Priority 4: bg-err-bg text-err

// ── Donut chart (stack coverage) ──────────────
// Use an <svg> donut built with <circle> stroke-dasharray
// Colors: lime (strong), lime/45 (learning), err (gaps)
// Centre text: text-[14px] font-bold fill-txt percentage
// Legend rows: flex items-center gap-2 text-[11px] text-muted
// Dot:  w-2 h-2 rounded-[2px] flex-shrink-0
```

---

## STEP 6 — Typography rules (apply everywhere)

```
ALL text uses font-mono (JetBrains Mono).
Never use font-sans or font-serif.

Heading sizes:
  Hero / page title:    text-[26px] font-bold tracking-tight   (letter-spacing: -0.02em)
  Card title:           text-[15px] font-bold tracking-tight
  Sub-card title:       text-[12px] font-bold
  Section label:        text-[10px] uppercase tracking-[0.1em]  ← // pattern //

Body sizes:
  Body text:            text-[12px] leading-[1.75]
  Metadata / captions:  text-[10px] leading-relaxed
  Micro labels:         text-[9px] uppercase tracking-[0.06em–0.1em]

Text colours:
  Primary:    text-txt    (#f0f0ee)
  Secondary:  text-muted  (#888880)
  Disabled:   text-dim    (#555550)
  Accent:     text-lime   (#d4f53c)  — interactive elements only
  Warning:    text-warn   (#ea9d34)
  Error:      text-err    (#e08080)

Section label pattern — use this EVERYWHERE for section headings:
  <p className="text-[10px] text-muted tracking-[0.1em] uppercase mb-3.5">
    // startup matches //
  </p>
```

---

## STEP 7 — Spacing system

| Name | Value | Tailwind class | Use |
|---|---|---|---|
| XS | 4px | `gap-1` / `p-1` | Icon-to-text gap |
| SM | 8px | `gap-2` / `p-2` | Chip gap, tight rows |
| MD | 12px | `gap-3` / `p-3` | Card inner gap |
| LG | 16px | `gap-4` / `p-4` | Card padding |
| XL | 24px | `gap-6` / `p-6` | Between sections |
| 2XL | 40px | `gap-10` / `p-10` | Section separators |

Card padding: always `p-[18px]` (not a Tailwind default, use inline or extend config).

---

## STEP 8 — Page title fix

In `public/index.html`, change:
```html
<!-- FROM: -->
<title>React App</title>

<!-- TO: -->
<title>devradar — your career. one screen.</title>
<meta name="description" content="AI-powered career intelligence for Indian developers. Match your stack to top startups, surface hackathons, and identify skill gaps — remembered across sessions by HydraDB." />
<meta name="theme-color" content="#0f0f0f" />
<meta property="og:title" content="devradar — your career. one screen." />
<meta property="og:description" content="Stack matching · Hackathon radar · Gap analysis · Persistent memory" />
```

---

## STEP 9 — Render cold-start UX (add to App.jsx)

```jsx
// Add this state and banner — prevents "is this broken?" when Render spins up
const [backendReady, setBackendReady] = useState(false)
const [waking, setWaking] = useState(true)

useEffect(() => {
  fetch(import.meta.env.VITE_API_URL + '/api/health')
    .then(() => { setBackendReady(true); setWaking(false) })
    .catch(() => setWaking(false))
}, [])

// Banner JSX (show when waking === true):
{waking && (
  <div className="bg-s2 border border-warn/20 rounded-card px-4 py-3 mb-4 flex items-center gap-2.5">
    <span className="text-warn text-[12px]">⏳</span>
    <p className="text-[11px] text-muted">
      backend warming up — <span className="text-txt font-bold">give it ~30 seconds</span> on first load (Render free tier)
    </p>
  </div>
)}
```

---

## STEP 10 — Final checklist before pushing

- [ ] `tailwind.config.js` has all custom colors + mono font
- [ ] `index.html` has JetBrains Mono link + correct title + OG tags
- [ ] `index.css` sets `font-family: 'JetBrains Mono'` on `body`
- [ ] All 6 components use `font-mono` — zero `font-sans` anywhere
- [ ] Every section label uses `// label //` pattern
- [ ] Score badges: lime ≥70%, warn 40–69%, err <40%
- [ ] Chip ON state: `bg-lime text-bg` — chip OFF: `bg-transparent text-muted border-bd2`
- [ ] MemoryBadge shown in nav at all times
- [ ] Render cold-start banner added to App.jsx
- [ ] `bg-bg` (#0f0f0f) set on `<body>` — no white flash on load

---

*Theme: Necto Mono — dark near-black · monospace-first · lime accent #d4f53c*
*Applied to: DevRadar — WikiThon 2025 · React 18 + Tailwind CSS*
