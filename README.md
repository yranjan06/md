# DevRadar — Soft Light Theme System
### Catppuccin Latte + Rosé Pine Dawn + Neutral Soft
### Eye-friendly · Warm · Not harsh white

---

```
Replace ALL dark theme CSS with soft light eye-friendly themes.
Three light options:
  1. Rosé Pine Dawn  — warm cream, most eye-friendly (DEFAULT)
  2. Catppuccin Latte — soft lavender-gray
  3. Neutral Soft     — pure clean light with blue accent

Default: Rosé Pine Dawn
Switcher in sidebar. Persists in localStorage.
Graph node colors are vibrant on light background — like LLM Wiki screenshot.

---

COMPLETE index.css — REPLACE ENTIRELY

@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ─────────────────────────────────────────
   ROSÉ PINE DAWN — Warm cream, eye-friendly
   ───────────────────────────────────────── */
:root,
:root[data-theme="rosepine-dawn"] {

  /* Backgrounds — warm cream tones */
  --bg-base:        #faf4ed;
  --bg-mantle:      #f2e9e1;
  --bg-crust:       #ede5da;
  --bg-surface0:    #f4ede8;
  --bg-surface1:    #dfdad9;
  --bg-surface2:    #cecacd;

  /* Borders */
  --border:         #dfdad9;
  --border-active:  #907aa9;

  /* Text — warm dark purple-gray, not black */
  --text:           #575279;
  --text-sub:       #797593;
  --text-muted:     #9893a5;
  --text-faint:     #b2afa9;

  /* Accent — soft iris/purple */
  --accent:         #907aa9;
  --accent-alt:     #d7827a;
  --accent-bg:      rgba(144,122,169,0.10);
  --accent-border:  rgba(144,122,169,0.30);

  /* Status */
  --success:        #286983;
  --success-bg:     rgba(40,105,131,0.08);
  --success-border: rgba(40,105,131,0.25);
  --warning:        #ea9d34;
  --warning-bg:     rgba(234,157,52,0.10);
  --warning-border: rgba(234,157,52,0.30);
  --danger:         #b4637a;
  --danger-bg:      rgba(180,99,122,0.08);
  --danger-border:  rgba(180,99,122,0.25);
  --info:           #56949f;
  --info-bg:        rgba(86,148,159,0.08);
  --info-border:    rgba(86,148,159,0.25);

  /* Graph nodes — vibrant on light bg */
  --node-user:      #ea9d34;
  --node-company:   #d7827a;
  --node-skill:     #56949f;
  --node-gap:       #b4637a;
  --node-hackathon: #907aa9;
  --node-edge:      #cecacd;

  /* Shadows — very soft */
  --shadow-card:    0 1px 4px rgba(87,82,121,0.08), 0 1px 2px rgba(87,82,121,0.05);
  --shadow-panel:   0 4px 16px rgba(87,82,121,0.12);
  --shadow-input:   0 0 0 2px rgba(144,122,169,0.20);

  /* Floating circles on opening screen */
  --circle-1:       #d7827a;
  --circle-2:       #907aa9;
  --circle-3:       #56949f;
  --circle-4:       #ea9d34;

  /* Message bubbles */
  --bubble-user-bg:      #907aa9;
  --bubble-user-text:    #faf4ed;
  --bubble-bot-bg:       #f4ede8;
  --bubble-bot-text:     #575279;
  --bubble-bot-border:   #dfdad9;
}

/* ─────────────────────────────────────────
   CATPPUCCIN LATTE — Soft lavender-gray
   ───────────────────────────────────────── */
:root[data-theme="catppuccin-latte"] {

  /* Backgrounds — cool light lavender */
  --bg-base:        #eff1f5;
  --bg-mantle:      #e6e9ef;
  --bg-crust:       #dce0e8;
  --bg-surface0:    #ccd0da;
  --bg-surface1:    #bcc0cc;
  --bg-surface2:    #acb0be;

  /* Borders */
  --border:         #ccd0da;
  --border-active:  #8839ef;

  /* Text — dark blue-gray */
  --text:           #4c4f69;
  --text-sub:       #5c5f77;
  --text-muted:     #6c6f85;
  --text-faint:     #8c8fa1;

  /* Accent — mauve purple */
  --accent:         #8839ef;
  --accent-alt:     #7287fd;
  --accent-bg:      rgba(136,57,239,0.09);
  --accent-border:  rgba(136,57,239,0.28);

  /* Status */
  --success:        #40a02b;
  --success-bg:     rgba(64,160,43,0.08);
  --success-border: rgba(64,160,43,0.25);
  --warning:        #df8e1d;
  --warning-bg:     rgba(223,142,29,0.09);
  --warning-border: rgba(223,142,29,0.28);
  --danger:         #d20f39;
  --danger-bg:      rgba(210,15,57,0.08);
  --danger-border:  rgba(210,15,57,0.25);
  --info:           #1e66f5;
  --info-bg:        rgba(30,102,245,0.08);
  --info-border:    rgba(30,102,245,0.25);

  /* Graph nodes */
  --node-user:      #df8e1d;
  --node-company:   #fe640b;
  --node-skill:     #1e66f5;
  --node-gap:       #d20f39;
  --node-hackathon: #8839ef;
  --node-edge:      #acb0be;

  /* Shadows */
  --shadow-card:    0 1px 4px rgba(76,79,105,0.10), 0 1px 2px rgba(76,79,105,0.06);
  --shadow-panel:   0 4px 16px rgba(76,79,105,0.14);
  --shadow-input:   0 0 0 2px rgba(136,57,239,0.18);

  /* Floating circles */
  --circle-1:       #fe640b;
  --circle-2:       #8839ef;
  --circle-3:       #1e66f5;
  --circle-4:       #40a02b;

  /* Message bubbles */
  --bubble-user-bg:      #8839ef;
  --bubble-user-text:    #eff1f5;
  --bubble-bot-bg:       #e6e9ef;
  --bubble-bot-text:     #4c4f69;
  --bubble-bot-border:   #ccd0da;
}

/* ─────────────────────────────────────────
   NEUTRAL SOFT — Clean blue-accent light
   ───────────────────────────────────────── */
:root[data-theme="neutral-soft"] {

  /* Backgrounds — pure clean light */
  --bg-base:        #f8f9fa;
  --bg-mantle:      #ffffff;
  --bg-crust:       #f1f3f5;
  --bg-surface0:    #f1f3f5;
  --bg-surface1:    #e9ecef;
  --bg-surface2:    #dee2e6;

  /* Borders */
  --border:         #e9ecef;
  --border-active:  #3b82f6;

  /* Text */
  --text:           #1a1a2e;
  --text-sub:       #495057;
  --text-muted:     #6c757d;
  --text-faint:     #adb5bd;

  /* Accent — blue */
  --accent:         #3b82f6;
  --accent-alt:     #6366f1;
  --accent-bg:      rgba(59,130,246,0.08);
  --accent-border:  rgba(59,130,246,0.25);

  /* Status */
  --success:        #059669;
  --success-bg:     rgba(5,150,105,0.08);
  --success-border: rgba(5,150,105,0.25);
  --warning:        #d97706;
  --warning-bg:     rgba(217,119,6,0.09);
  --warning-border: rgba(217,119,6,0.28);
  --danger:         #dc2626;
  --danger-bg:      rgba(220,38,38,0.08);
  --danger-border:  rgba(220,38,38,0.25);
  --info:           #0284c7;
  --info-bg:        rgba(2,132,199,0.08);
  --info-border:    rgba(2,132,199,0.25);

  /* Graph nodes */
  --node-user:      #d97706;
  --node-company:   #ea580c;
  --node-skill:     #3b82f6;
  --node-gap:       #dc2626;
  --node-hackathon: #7c3aed;
  --node-edge:      #dee2e6;

  /* Shadows */
  --shadow-card:    0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.04);
  --shadow-panel:   0 4px 16px rgba(0,0,0,0.10);
  --shadow-input:   0 0 0 2px rgba(59,130,246,0.20);

  /* Floating circles */
  --circle-1:       #ea580c;
  --circle-2:       #7c3aed;
  --circle-3:       #3b82f6;
  --circle-4:       #059669;

  /* Message bubbles */
  --bubble-user-bg:      #3b82f6;
  --bubble-user-text:    #ffffff;
  --bubble-bot-bg:       #ffffff;
  --bubble-bot-text:     #1a1a2e;
  --bubble-bot-border:   #e9ecef;
}

/* ─────────────────────────────────────────
   BASE STYLES — Same for all themes
   ───────────────────────────────────────── */
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  background: var(--bg-base);
  color: var(--text);
  -webkit-font-smoothing: antialiased;
  line-height: 1.6;
  transition: background 200ms ease, color 200ms ease;
}

::selection {
  background: var(--accent-bg);
  color: var(--text);
}

::-webkit-scrollbar { width: 5px; height: 5px; }
::-webkit-scrollbar-track { background: var(--bg-base); }
::-webkit-scrollbar-thumb {
  background: var(--bg-surface2);
  border-radius: 999px;
}
::-webkit-scrollbar-thumb:hover { background: var(--text-muted); }

/* Override vis-network tooltip for light theme */
.vis-tooltip {
  background: var(--bg-mantle) !important;
  border: 1px solid var(--border) !important;
  border-radius: 10px !important;
  color: var(--text) !important;
  font-family: 'Inter', sans-serif !important;
  font-size: 12px !important;
  padding: 8px 12px !important;
  box-shadow: var(--shadow-panel) !important;
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-18px); }
}
@keyframes pulse-dot {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.3; }
}
@keyframes fade-up {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes slide-right {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
@keyframes slide-bottom {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}

.anim-fade-up { animation: fade-up 350ms cubic-bezier(0.16,1,0.3,1) forwards; }
.anim-pulse { animation: pulse-dot 2s ease-in-out infinite; }
.anim-slide-right { animation: slide-right 250ms ease forwards; }
.anim-slide-bottom { animation: slide-bottom 280ms ease forwards; }

---

UPDATE useTheme.js

const THEMES = {
  "rosepine-dawn": {
    id: "rosepine-dawn",
    label: "Rosé Pine Dawn",
    emoji: "🌸",
    preview: ["#d7827a", "#907aa9", "#56949f"]
  },
  "catppuccin-latte": {
    id: "catppuccin-latte",
    label: "Catppuccin Latte",
    emoji: "☕",
    preview: ["#fe640b", "#8839ef", "#1e66f5"]
  },
  "neutral-soft": {
    id: "neutral-soft",
    label: "Neutral Soft",
    emoji: "🪨",
    preview: ["#ea580c", "#7c3aed", "#3b82f6"]
  }
};

const DEFAULT_THEME = "rosepine-dawn";

---

FULL COMPONENT STYLE GUIDE
Apply these to every component using CSS variables only:

CARDS:
  background: var(--bg-mantle)
  border: 1px solid var(--border)
  border-radius: 10px
  box-shadow: var(--shadow-card)

INPUTS:
  background: var(--bg-surface0)
  border: 1px solid var(--border)
  color: var(--text)
  placeholder color: var(--text-faint)
  on focus:
    border-color: var(--border-active)
    box-shadow: var(--shadow-input)
    outline: none

BUTTONS — Primary:
  background: var(--accent)
  color: var(--bg-base)
  border: none
  hover: filter brightness(1.08)
  disabled: opacity 0.4, cursor not-allowed

BUTTONS — Secondary/Outlined:
  background: transparent
  border: 1px solid var(--border)
  color: var(--text-sub)
  hover: background var(--bg-surface0)

BUTTONS — Active/Selected:
  background: var(--accent-bg)
  border: 1px solid var(--accent-border)
  color: var(--accent)

BADGES:
  Success: bg var(--success-bg), border var(--success-border), color var(--success)
  Warning: bg var(--warning-bg), border var(--warning-border), color var(--warning)
  Danger: bg var(--danger-bg), border var(--danger-border), color var(--danger)
  Info: bg var(--info-bg), border var(--info-border), color var(--info)

SIDEBAR:
  background: var(--bg-mantle)
  border-right: 1px solid var(--border)

TOPBAR:
  background: var(--bg-mantle)
  border-bottom: 1px solid var(--border)
  box-shadow: var(--shadow-card)

CHAT bubbles:
  User: bg var(--bubble-user-bg), color var(--bubble-user-text)
  Bot: bg var(--bubble-bot-bg), border var(--bubble-bot-border), color var(--bubble-bot-text)

---

UPDATED ThemeSwitcher.jsx FOR LIGHT THEMES

export default function ThemeSwitcher() {
  const { theme, setTheme, themes } = useTheme();

  return (
    <div className="px-3 py-3"
         style={{ borderTop: "1px solid var(--border)" }}>
      <div style={{
        fontSize: "10px",
        textTransform: "uppercase",
        letterSpacing: "0.7px",
        color: "var(--text-faint)",
        marginBottom: "8px"
      }}>
        Theme
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: "4px" }}>
        {Object.values(themes).map(t => (
          <button key={t.id} onClick={() => setTheme(t.id)}
                  style={{
                    display: "flex",
                    alignItems: "center",
                    gap: "10px",
                    padding: "7px 10px",
                    borderRadius: "8px",
                    textAlign: "left",
                    width: "100%",
                    cursor: "pointer",
                    transition: "all 150ms",
                    background: theme === t.id ? "var(--accent-bg)" : "transparent",
                    border: `1px solid ${theme === t.id ? "var(--accent-border)" : "transparent"}`,
                    color: theme === t.id ? "var(--accent)" : "var(--text-muted)"
                  }}>
            {/* Preview dots */}
            <div style={{ display: "flex", gap: "3px", flexShrink: 0 }}>
              {t.preview.map((color, i) => (
                <div key={i} style={{
                  width: "10px", height: "10px",
                  borderRadius: "50%", background: color
                }} />
              ))}
            </div>
            <span style={{ fontSize: "12px", fontWeight: 500, flex: 1 }}>
              {t.label}
            </span>
            {theme === t.id && (
              <span style={{ fontSize: "11px", color: "var(--accent)" }}>✓</span>
            )}
          </button>
        ))}
      </div>
    </div>
  );
}

---

UPDATED OpeningScreen.jsx — LIGHT THEME CARD

The opening card on light theme:

background: var(--bg-mantle)
border: 1px solid var(--border)
box-shadow: var(--shadow-panel)
border-radius: 16px

NO glow effect — glow is for dark themes.
Instead: very soft shadow.

Tag input container:
  background: var(--bg-surface0)
  border: 1px solid var(--border)
  Focused: border var(--border-active), box-shadow var(--shadow-input)

Skill tags:
  background: var(--accent-bg)
  border: 1px solid var(--accent-border)
  color: var(--accent)

CTA button:
  background: var(--accent)
  color: white (or var(--bg-base) for Rosé Pine Dawn)
  no border

---

UPDATED MemoryBadge FOR LIGHT THEME

On light theme, memory badge should be warm — not dark:

background: var(--warning-bg)        ← soft warm yellow/gold tint
border-bottom: 1px solid var(--warning-border)
color: var(--warning)
Message text: var(--text)

Urgent pills:
  Deadline < 3 days: bg var(--danger-bg), border var(--danger-border), color var(--danger)
  Deadline < 7 days: bg var(--warning-bg), border var(--warning-border), color var(--warning)

HydraDB status dot: var(--success)

---

SIGMA.JS GRAPH CONFIG FOR LIGHT THEME

Light theme needs different sigma config —
nodes must be vibrant to pop on light canvas:

const options = {
  physics enabled,
  nodes: {
    font: {
      color: "var(--text)" read via getComputedStyle,
      strokeWidth: 4,
      strokeColor: getComputedStyle bg-base value
    },
    shadow: {
      enabled: true,
      color: "rgba(0,0,0,0.15)",
      size: 6, x: 0, y: 2
    }
  },
  edges: {
    color: { color: getComputedStyle node-edge value, opacity: 0.5 }
  }
}

Canvas background: var(--bg-base)
The graph container div:
  style={{ background: "var(--bg-base)" }}

On light: nodes look more vibrant — Rosé Pine Dawn rose + iris + foam
are beautiful on the cream bg exactly like LLM Wiki screenshot.

---

THEME PREVIEW COMPARISON

Visual feel of each:

Rosé Pine Dawn (DEFAULT):
  Like an old book, warm parchment
  Cream bg #faf4ed + rose/iris/foam accents
  Most eye-friendly for long sessions
  Warm golden yellow user node pops beautifully

Catppuccin Latte:
  Like a clear overcast morning
  Gray-lavender bg #eff1f5 + mauve/orange/blue nodes
  Clean, professional, slightly cooler tone
  High contrast nodes — orange company, blue skill pop

Neutral Soft:
  Like white paper, cleanest
  Near-white bg #f8f9fa + standard blue/purple accents
  Closest to GitHub/Notion feel
  Most conventional, safest for demos

---

TESTING FLOW 17 — LIGHT THEME VERIFICATION

Add to E2ETest.md:

FLOW 17 — LIGHT THEME

Open app on desktop.

VERIFY default theme is Rosé Pine Dawn:
  Page bg is warm cream — NOT white, NOT dark
  Sidebar bg slightly darker cream
  Text is dark purple-gray (#575279) — not black, not harsh
  Accent color is soft iris purple

VERIFY eye-friendliness:
  No pure white (#ffffff) backgrounds
  No pure black (#000000) text
  All contrasts readable but soft
  Feels warm, like reading on paper

Click Catppuccin Latte:
  Page bg changes to cool gray-lavender
  Text becomes blue-gray (#4c4f69)
  Accent becomes mauve (#8839ef)
  Graph nodes: orange company, blue skill — high contrast on gray

Click Neutral Soft:
  Page bg near-white
  Blue accent replaces purple
  Most neutral look

In ALL three themes verify:
  Graph canvas uses the bg-base color — matches page
  Nodes are vibrant colored circles on light background
  Exactly like LLM Wiki screenshot — colorful nodes on light bg
  Node legend readable
  Sidebar readable
  Chat readable
  Roadmap readable
  No dark panels anywhere

Screenshot: Rosé Pine Dawn — full app.
Screenshot: Catppuccin Latte — full app.
Screenshot: graph closeup in each theme showing vibrant nodes.

FLOW 17 — LIGHT THEME: PASS/FAIL

---

GIT COMMITS

"replace dark theme vars with rosepine dawn light theme"
"add catppuccin latte light theme vars"
"add neutral soft light theme vars"
"update useTheme default to rosepine-dawn"
"update ThemeSwitcher for light theme options"
"update graph canvas bg to use theme variable"
"update vis-network tooltip for light theme"
"update MemoryBadge warm colors for light theme"
"update OpeningScreen card shadow for light no glow"
"verify no dark colors remain in any component"
"all three light themes tested passing"
```
