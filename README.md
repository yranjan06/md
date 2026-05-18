# DevRadar — UI Changes Implementation Prompt

> Paste this entire prompt into Claude Code or your AI editor.
> Apply all changes across the DevRadar React 18 + Tailwind CSS codebase.

---

## CONTEXT

- **Repo:** https://github.com/iamabhaydawar/devradar
- **Frontend:** React 18 + Vite + Tailwind CSS → `frontend/src/`
- **Theme:** Necto Mono dark (#0f0f0f canvas, #d4f53c lime accent, monospace everywhere)
- **Components:** App.jsx, StackInput.jsx, Dashboard.jsx, StartupCard.jsx,
  HackathonCard.jsx, GapAnalysis.jsx, MemoryBadge.jsx

---

## CHANGE 1 — Sidebar navigation order

**File:** `frontend/src/components/Sidebar.jsx`
(or wherever the sidebar nav items are defined — find the component
that renders the sidebar menu)

**Rule:** Move "Roadmap Journey" nav item to appear **directly below**
"Your Profile" in the sidebar. It should be the second item in the list.

**Before (current order — example):**
```jsx
<NavItem icon="home"     label="Dashboard"      />
<NavItem icon="radar"    label="Roadmap Journey" />  ← currently here
<NavItem icon="user"     label="Your Profile"   />
<NavItem icon="chart"    label="Gap Analysis"   />
```

**After (correct order):**
```jsx
<NavItem icon="home"     label="Dashboard"      />
<NavItem icon="user"     label="Your Profile"   />
<NavItem icon="radar"    label="Roadmap Journey" />  ← moved here, below Profile
<NavItem icon="chart"    label="Gap Analysis"   />
```

**Exact rule:** Find every array, list, or JSX block that defines sidebar
navigation items. Locate "Your Profile" and "Roadmap Journey" entries.
Ensure "Roadmap Journey" always comes immediately after "Your Profile"
regardless of how many items are in the list.

---

## CHANGE 2 — Logo: shared component used on ALL pages

### 2a. Create a shared Logo component

Create `frontend/src/components/Logo.jsx` with TWO exports:

**Export 1 — `<LogoMark size={n} />` (animated dot-pixel radar icon)**

This is a `<canvas>` element rendering a 13×13 dot grid radar with:
- Near-black background (`#080808`)
- Dim dots for the grid (`#1e1e1c`)
- Ring dots at radii r=2, r=4, r=6 from center (`#383830`)
- Crosshair dots along center row/col (`#232320`)
- Lime sweep arm rotating continuously (`rgba(212,245,60,0.15)` line)
- Dots under the sweep flare to lime (`#d4f53c`) then fade
- Blip dot at grid position (9,3) pulsing in lime
- Center dot always lime

Canvas size = `size` prop (default 28). Animation runs via
`requestAnimationFrame`. Use a `useEffect` with cleanup to start/stop
the animation loop when the component mounts/unmounts.

```jsx
import { useEffect, useRef } from 'react'

export function LogoMark({ size = 28 }) {
  const ref = useRef(null)
  useEffect(() => {
    const canvas = ref.current
    // — paste the full drawRadar() logic here —
    // same algorithm as the landing page hero canvas
    let angle = 0, pulse = 0
    let raf
    const loop = () => {
      angle += 0.016
      pulse += 0.065
      drawRadar(canvas, size, angle, pulse, false)
      raf = requestAnimationFrame(loop)
    }
    raf = requestAnimationFrame(loop)
    return () => cancelAnimationFrame(raf)
  }, [size])
  return <canvas ref={ref} width={size} height={size} />
}
```

Full `drawRadar(canvas, size, angle, pulse, inverted)` algorithm:

```js
const GRID = 13, CX = 6, CY = 6
function drawRadar(canvas, size, angle, pulse, inv) {
  const ctx = canvas.getContext('2d')
  const PAD = size * 0.07
  const AREA = size - PAD * 2
  const CELL = AREA / (GRID - 1)
  const DR = Math.max(1, CELL * 0.32)
  ctx.fillStyle = inv ? '#e8e8e0' : '#080808'
  ctx.fillRect(0, 0, size, size)
  const ox = PAD, oy = PAD
  const sweep = ((angle % (Math.PI*2)) + Math.PI*2) % (Math.PI*2)
  for (let gy = 0; gy < GRID; gy++) {
    for (let gx = 0; gx < GRID; gx++) {
      const px = ox + gx * CELL, py = oy + gy * CELL
      const dx = gx - CX, dy = gy - CY
      const dist = Math.sqrt(dx*dx + dy*dy)
      if (dist > 6.6) continue
      const isC = gx === CX && gy === CY
      const isB = gx === 9 && gy === 3
      const isR = Math.abs(dist-2) < .58 || Math.abs(dist-4) < .58 || Math.abs(dist-6) < .58
      const isCr = (gx === CX || gy === CY) && dist <= 6.2
      const da = ((Math.atan2(dy,dx) % (Math.PI*2)) + Math.PI*2) % (Math.PI*2)
      const trail = (sweep - da + Math.PI*2) % (Math.PI*2)
      let sb = 0
      if (!isC && dist > .5) {
        if (trail < .1) sb = 1
        else if (trail < 1.4) sb = .9 * (1 - (trail - .1) / 1.3)
      }
      let color, alpha = 1
      if (isC) { color = '#d4f53c' }
      else if (isB) {
        const bp = Math.abs(Math.sin(pulse))
        color = '#d4f53c'; alpha = sb > .5 ? 1 : .25 + bp * .75
      } else if (sb > 0) {
        if (isR) {
          const t = sb
          color = `rgb(${Math.round(212*t+42*(1-t))},${Math.round(245*t+42*(1-t))},${Math.round(60*t+8*(1-t))})`
        } else {
          color = `rgba(212,245,60,${sb * .45})`
        }
      } else if (isR) { color = inv ? '#aaaaaa' : '#383830' }
      else if (isCr) { color = inv ? '#cccccc' : '#232320' }
      else { color = inv ? '#dddddd' : '#1e1e1c' }
      ctx.globalAlpha = alpha
      ctx.beginPath(); ctx.arc(px, py, DR, 0, Math.PI*2)
      ctx.fillStyle = color; ctx.fill()
      ctx.globalAlpha = 1
    }
  }
  if (size >= 48) {
    const ll = (GRID / 2 - .5) * CELL
    ctx.beginPath()
    ctx.moveTo(ox + CX*CELL, oy + CY*CELL)
    ctx.lineTo(ox + CX*CELL + Math.cos(sweep)*ll, oy + CY*CELL + Math.sin(sweep)*ll)
    ctx.strokeStyle = 'rgba(212,245,60,0.15)'
    ctx.lineWidth = Math.max(.5, size * .006)
    ctx.stroke()
  }
}
```

**Export 2 — `<LogoText size="sm"|"md"|"lg" />` (pixel dot wordmark)**

Renders "DEVRADAR" as dot-pixel art on a `<canvas>`.
Each letter is a 5-column × 7-row binary grid.

Pixel font data:
```js
const FONT = {
  D: [[1,1,1,0,0],[1,0,0,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,1,0],[1,1,1,0,0]],
  E: [[1,1,1,1,1],[1,0,0,0,0],[1,0,0,0,0],[1,1,1,1,0],[1,0,0,0,0],[1,0,0,0,0],[1,1,1,1,1]],
  V: [[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,0,1,0],[0,1,0,1,0],[0,0,1,0,0],[0,0,1,0,0]],
  R: [[1,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,0],[1,0,1,0,0],[1,0,0,1,0],[1,0,0,0,1]],
  A: [[0,0,1,0,0],[0,1,0,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,1],[1,0,0,0,1],[1,0,0,0,1]],
}
const WORD = 'DEVRADAR'.split('')
```

Size variants:
- `sm` → step = 6px, dot radius = 1.8px  (nav bar, footer)
- `md` → step = 9px, dot radius = 2.8px  (page headers)
- `lg` → step = 12px, dot radius = 3.8px (landing hero)

Compute canvas dimensions from step:
```js
const STEP = { sm: 6, md: 9, lg: 12 }[size] ?? 9
const CHAR_W = 5 * STEP
const GAP = STEP
const W = WORD.length * CHAR_W + (WORD.length - 1) * GAP
const H = 7 * STEP
```

Lit dots → `#d4f53c` (lime)
Unlit dots → `#1e1e1c` (visible grid, not invisible)

Animate: pass the shared `sweepAngle` so dots near the sweep arm
briefly brighten — same formula as `drawRadar`.

---

### 2b. Use `<LogoMark>` and `<LogoText>` on every page

Replace every existing logo/title reference across the app:

| Location | Replace with |
|---|---|
| `App.jsx` nav bar | `<LogoMark size={28} />` + `"devradar"` wordmark text |
| Landing page hero | `<LogoMark size={200} />` above `<LogoText size="lg" />` |
| Page `<title>` (index.html) | already fixed — keep as is |
| Sidebar header | `<LogoMark size={24} />` + `"devradar"` text |
| Auth / onboarding screen | `<LogoMark size={64} />` + `<LogoText size="md" />` |
| Footer | `<LogoMark size={22} />` + `"devradar"` text |
| Tab favicon | already set in index.html — keep as is |
| Any `<h1>` or `<title>` that says "DevRadar" or "devradar" as plain text | wrap with `<LogoText size="sm" />` where space allows, or keep text with monospace class |

**Import pattern everywhere:**
```jsx
import { LogoMark, LogoText } from '../components/Logo'
// or from wherever Logo.jsx lives relative to the file
```

---

## CHANGE 3 — Replace 2025 → 2026 everywhere

**Search and replace across the ENTIRE codebase:**

```
find: 2025
replace: 2026
```

**Files to check specifically:**
- `frontend/src/components/*.jsx` — any hardcoded year in UI text
- `frontend/index.html` — meta description, OG tags
- `backend/data/hackathons.json` — hackathon dates/years
- `backend/data/startups.json` — any year references
- `README.md` — WikiThon year, dates
- Any copyright notice or footer text

**Exceptions — do NOT change these:**
- npm package versions that happen to contain "2025"
- Git commit hashes
- Any URL that contains "2025" as a path segment belonging to an external service
- `claude-sonnet-4-20250514` model string — this is an API model ID, not a year reference

---

## CHANGE 4 — Ingest Chat / Roadmap Journey section

**File:** wherever the "Ingest Chat" or "Roadmap Journey" feature is rendered.

**Rule:** This section must render **below** the "Your Profile" section/component
in the sidebar. Confirm the DOM order, not just visual order — use
`order` CSS or reorder the JSX array directly.

If this is a route-based sidebar (React Router), ensure the route link
for Roadmap Journey is listed after the route link for Your Profile in
the nav items array.

---

## CHANGE 5 — Landing page typography — desktop optimised

**Problem:** All text on the landing page is too small for desktop viewport.
Body text at 11–12px, section headings at 13–16px — these sizes work on
mobile but feel microscopic on 1280px+ screens. Every font size needs
to scale up for desktop readability.

**File:** `frontend/src/pages/Landing.jsx`
(or wherever the landing page JSX lives — could also be `App.jsx`
if there is no separate landing page component)

**Also update:** `frontend/index.css` or `tailwind.config.js` if base
font sizes are set there globally.

---

### Desktop type scale — apply these exact sizes

Use Tailwind responsive prefixes (`md:` = 768px+, `lg:` = 1024px+).
Every size below has a mobile value and a desktop override.

| Element | Mobile | Desktop (`lg:`) | Tailwind classes |
|---|---|---|---|
| Nav wordmark | 14px | 16px | `text-sm lg:text-base` |
| Nav links / pills | 10px | 12px | `text-[10px] lg:text-xs` |
| Hero tagline (above DEVRADAR) | 11px | 13px | `text-[11px] lg:text-[13px]` |
| Hero subheading (below DEVRADAR) | 13px | 18px | `text-[13px] lg:text-lg` |
| Hero body paragraph | 13px | 16px | `text-[13px] lg:text-base` |
| CTA button text | 12px | 15px | `text-xs lg:text-[15px]` |
| Stats numbers | 36px | 52px | `text-4xl lg:text-[52px]` |
| Stats labels | 9px | 12px | `text-[9px] lg:text-xs` |
| Section label (// label //) | 9px | 11px | `text-[9px] lg:text-[11px]` |
| Section heading (h2) | 18px | 32px | `text-lg lg:text-[32px]` |
| Section body paragraph | 12px | 15px | `text-xs lg:text-[15px]` |
| Feature card title | 13px | 15px | `text-[13px] lg:text-[15px]` |
| Feature card body | 11px | 13px | `text-[11px] lg:text-[13px]` |
| Step number (01, 02...) | 24px | 36px | `text-2xl lg:text-4xl` |
| Step title | 11px | 14px | `text-[11px] lg:text-sm` |
| Step body | 10px | 13px | `text-[10px] lg:text-[13px]` |
| Memory/Claude quote text | 11px | 14px | `text-[11px] lg:text-sm` |
| Footer text | 10px | 13px | `text-[10px] lg:text-[13px]` |
| Skill/stack tags | 11px | 13px | `text-[11px] lg:text-[13px]` |
| URL / code snippets | 10px | 12px | `text-[10px] lg:text-xs` |

---

### Desktop spacing — increase breathing room at lg:

Landing page sections feel cramped at desktop widths.
Apply these padding/gap overrides:

```jsx
// Hero section
// mobile: py-14    desktop: py-24
className="py-14 lg:py-24"

// Section wrappers (features, how-it-works, memory)
// mobile: py-12    desktop: py-20
className="py-12 lg:py-20 px-7 lg:px-16"

// Feature card grid
// mobile: gap-3    desktop: gap-5
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 lg:gap-5 mt-8 lg:mt-12"

// Feature card internal padding
// mobile: p-5      desktop: p-7
className="... p-5 lg:p-7"

// Step grid
// mobile: gap-3    desktop: gap-5
className="grid grid-cols-2 lg:grid-cols-4 gap-3 lg:gap-5"

// Stat numbers — give them more vertical padding
// mobile: py-7     desktop: py-10
className="... py-7 lg:py-10"

// CTA button padding
// mobile: px-7 py-3   desktop: px-10 py-4
className="... px-7 py-3 lg:px-10 lg:py-4"

// Max content width — constrain to readable width on ultra-wide
// Wrap main content in: max-w-7xl mx-auto
```

---

### Desktop max-width container

Wrap every landing page section's inner content in a max-width
container to prevent lines stretching too wide on large monitors:

```jsx
// Add this wrapper inside every <section>:
<div className="max-w-6xl mx-auto px-6 lg:px-12">
  {/* section content */}
</div>
```

Hero section gets a narrower container for text (centred):
```jsx
<div className="max-w-3xl mx-auto text-center">
  {/* tagline, subheading, CTAs */}
</div>
```

---

### Line height and letter spacing — desktop overrides

```jsx
// Hero subheading — tighter tracking on large sizes
className="... lg:tracking-tight lg:leading-tight"

// Body paragraphs — more leading at larger sizes
className="... lg:leading-relaxed"

// Section headings — tighten tracking as size increases
className="... lg:tracking-[-0.03em]"

// // section label // — keep wide tracking at all sizes
className="... tracking-[0.1em] lg:tracking-[0.14em]"
```

---

### LogoMark size on desktop hero — make it bigger

The animated radar on the landing page hero should be larger on desktop:

```jsx
// mobile: 160px    desktop: 240px
<LogoMark size={isMobile ? 160 : 240} />

// Or use CSS transform to scale the canvas on desktop:
<div className="w-40 h-40 lg:w-60 lg:h-60 lg:scale-100">
  <LogoMark size={240} />
</div>
```

LogoText (DEVRADAR pixel art) — add an `xl` size variant:
- `xl` → step = 16px, dot radius = 5px (landing hero desktop)

```js
const STEP = { sm: 6, md: 9, lg: 12, xl: 16 }[size] ?? 9
```

Use `<LogoText size="xl" />` on desktop hero,
`<LogoText size="lg" />` on mobile hero.

---

## IMPLEMENTATION ORDER

Apply changes in this sequence to avoid merge conflicts:

```
1. Create frontend/src/components/Logo.jsx  (new file)
2. Update App.jsx                           (import + use LogoMark)
3. Update Sidebar.jsx                       (reorder nav items)
4. Update all other components              (import + use LogoMark/LogoText)
5. Update Landing page — all type sizes + spacing (Change 5)
6. Update tailwind.config.js                (add xl LogoText step)
7. Global find-replace 2025 → 2026
8. Verify backend/data/ JSON files
9. Run: npm run dev — check no console errors at 1280px+ viewport
10. Commit: git add . && git commit -m "feat: shared logo, sidebar reorder, desktop typography, 2026 update"
11. Push:  git push origin feat/necto-mono-theme
```

---

## VERIFICATION CHECKLIST

After implementing, confirm:

- [ ] `<LogoMark>` animates on every page — nav, sidebar, footer, landing
- [ ] `<LogoText>` shows DEVRADAR in dot pixels on landing hero
- [ ] Sidebar: "Your Profile" comes before "Roadmap Journey"
- [ ] No page shows plain text "DevRadar" as a heading where logo should be
- [ ] No instance of "2025" remains in UI-visible text
- [ ] `hackathons.json` dates updated to 2026
- [ ] Landing page at 1280px — all text is comfortably readable (no squinting)
- [ ] Landing page at 375px — text is not oversized, still fits on screen
- [ ] Hero DEVRADAR pixel text is large and crisp at desktop width
- [ ] Section headings hit 28–32px on desktop, not 16–18px
- [ ] CTA buttons have generous padding — not tiny on desktop
- [ ] `npm run build` completes with no errors
- [ ] `npm run dev` — all pages load, logo animates, no console errors

---

*Codebase: DevRadar · React 18 + Vite + Tailwind CSS*
*Theme: Necto Mono dark — #0f0f0f · #d4f53c · JetBrains Mono*
*WikiThon 2026*
