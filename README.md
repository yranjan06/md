# DevRadar — Catppuccin Theme + Opening Screen Prompt
### Mocha flavor · LLM Wiki inspired opening · Career knowledge graph

Paste this into Claude to apply Catppuccin Mocha theme throughout DevRadar.

---

```
Apply Catppuccin Mocha theme to all DevRadar frontend files.
Also create a new polished opening/onboarding screen
inspired by LLM Wiki's landing pattern, adapted for DevRadar's career niche.

---

CATPPUCCIN MOCHA — FULL PALETTE

Paste this as CSS variables at the top of index.css:

:root {
  /* Catppuccin Mocha — Base */
  --ctp-rosewater: #F5E0DC;
  --ctp-flamingo:  #F2CDCD;
  --ctp-pink:      #F5C2E7;
  --ctp-mauve:     #CBA6F7;
  --ctp-red:       #F38BA8;
  --ctp-maroon:    #EBA0AC;
  --ctp-peach:     #FAB387;
  --ctp-yellow:    #F9E2AF;
  --ctp-green:     #A6E3A1;
  --ctp-teal:      #94E2D5;
  --ctp-sky:       #89DCEB;
  --ctp-sapphire:  #74C7EC;
  --ctp-blue:      #89B4FA;
  --ctp-lavender:  #B4BEFE;

  /* Catppuccin Mocha — Text */
  --ctp-text:      #CDD6F4;
  --ctp-subtext1:  #BAC2DE;
  --ctp-subtext0:  #A6ADC8;

  /* Catppuccin Mocha — Overlay */
  --ctp-overlay2:  #9399B2;
  --ctp-overlay1:  #7F849C;
  --ctp-overlay0:  #6C7086;

  /* Catppuccin Mocha — Surface */
  --ctp-surface2:  #585B70;
  --ctp-surface1:  #45475A;
  --ctp-surface0:  #313244;

  /* Catppuccin Mocha — Base */
  --ctp-base:      #1E1E2E;
  --ctp-mantle:    #181825;
  --ctp-crust:     #11111B;
}

---

DESIGN SYSTEM USING CATPPUCCIN MOCHA

Page background:       var(--ctp-base)       #1E1E2E
Sidebar background:    var(--ctp-mantle)     #181825
Card background:       var(--ctp-surface0)   #313244
Card hover:            var(--ctp-surface1)   #45475A
Input background:      var(--ctp-surface0)   #313244
Border default:        var(--ctp-surface1)   #45475A
Border active:         var(--ctp-mauve)      #CBA6F7
Border subtle:         var(--ctp-surface0)   #313244

Text primary:          var(--ctp-text)       #CDD6F4
Text secondary:        var(--ctp-subtext1)   #BAC2DE
Text muted:            var(--ctp-overlay1)   #7F849C
Text placeholder:      var(--ctp-overlay0)   #6C7086

Accent primary:        var(--ctp-mauve)      #CBA6F7
Accent hover:          var(--ctp-lavender)   #B4BEFE
Accent light bg:       rgba(203,166,247,0.1)

Success:               var(--ctp-green)      #A6E3A1
Success bg:            rgba(166,227,161,0.1)
Warning:               var(--ctp-yellow)     #F9E2AF
Warning bg:            rgba(249,226,175,0.1)
Danger:                var(--ctp-red)        #F38BA8
Danger bg:             rgba(243,139,168,0.1)
Info:                  var(--ctp-blue)       #89B4FA
Info bg:               rgba(137,180,250,0.1)

Shadow:
  card:   0 2px 8px rgba(17,17,27,0.4)
  panel:  0 4px 20px rgba(17,17,27,0.6)
  glow:   0 0 20px rgba(203,166,247,0.15)

---

GRAPH NODE COLORS (Catppuccin palette)

User center node:      var(--ctp-yellow)     #F9E2AF  ← warm, stands out
Companies:             var(--ctp-peach)      #FAB387  ← orange warmth
Skills known:          var(--ctp-blue)       #89B4FA  ← calm blue
Skills gap:            var(--ctp-red)        #F38BA8  ← soft red not harsh
Hackathons:            var(--ctp-mauve)      #CBA6F7  ← signature catppuccin
Concepts:              var(--ctp-teal)       #94E2D5  ← teal accent

Edge colors:
Known skill edge:      var(--ctp-blue)       opacity 0.5
Gap edge dashed:       var(--ctp-red)        opacity 0.4
Hackathon edge:        var(--ctp-mauve)      opacity 0.4
Default edge:          var(--ctp-surface2)   opacity 0.6

---

OPENING SCREEN — LLM WIKI INSPIRED

Create a new file: frontend/src/components/OpeningScreen.jsx

This is shown BEFORE the user enters their stack.
It replaces the plain StackInput as the very first thing they see.

LLM Wiki inspiration:
  - Clean centered content
  - Logo prominent at top
  - Short punchy description
  - Visual preview hint (graph nodes floating in bg)
  - One clear CTA

DevRadar adaptation:
  - Career focused copy
  - Show the three outputs (graph, chat, roadmap) as feature pills
  - Indian developer context in the copy

OPENING SCREEN LAYOUT:

Full viewport, background: var(--ctp-base) #1E1E2E

Subtle background effect:
  Floating colored circles, very low opacity, absolute positioned
  Like graph nodes floating behind the content
  Use CSS animation: slow float up and down (10-15s loop)
  Colors from catppuccin palette, opacity 0.06

  Example circles (position absolute, pointer-events none):
    circle 1: 320px, ctp-peach, top 10%, left 5%, opacity 0.05
    circle 2: 240px, ctp-mauve, top 60%, left 80%, opacity 0.06
    circle 3: 180px, ctp-blue, top 30%, right 10%, opacity 0.05
    circle 4: 280px, ctp-teal, bottom 15%, left 30%, opacity 0.04

Center card (max-width 480px, centered):
  Background: var(--ctp-mantle) #181825
  Border: 1px solid var(--ctp-surface1)
  Border-radius: 16px
  Padding: 48px 40px
  Box-shadow: panel shadow + subtle mauve glow

  TOP SECTION:
    Logo row:
      Small colored squares stacked (like graph nodes) — decorative
      Three squares: ctp-peach 8px, ctp-blue 8px, ctp-mauve 8px
      Gap 4px, inline
    
    App name: "DevRadar"
      Font: 28px, weight 700, color: var(--ctp-text)
      Letter-spacing: -0.5px
    
    Tagline: "Your personal career knowledge base"
      Font: 14px, color: var(--ctp-subtext1)
      Margin-top: 4px
    
    Hackathon badge:
      "WikiThon 2025"
      Background: rgba(203,166,247,0.15)
      Border: 1px solid rgba(203,166,247,0.3)
      Color: var(--ctp-mauve)
      Padding: 3px 10px, border-radius: 999px, font-size: 11px
      Margin-top: 12px, display inline-block

  DIVIDER:
    1px solid var(--ctp-surface1), margin: 24px 0

  MIDDLE SECTION — What it does:
    Heading: "Build your career graph"
      Font: 20px, weight 600, color: var(--ctp-text)
    
    Subtext: "Feed job descriptions, LinkedIn posts, or screenshots.
              DevRadar extracts career insights and builds a
              personal knowledge graph — powered by HydraDB memory."
      Font: 13px, color: var(--ctp-subtext0)
      Line-height: 1.6, margin-top: 8px

    Feature pills row (margin-top 16px):
      Three small pills in a row:
      
      Pill 1: "🗺  Career Graph"
        Background: rgba(137,180,250,0.1)
        Border: 1px solid rgba(137,180,250,0.25)
        Color: var(--ctp-blue)
      
      Pill 2: "💬  Ask Anything"
        Background: rgba(166,227,161,0.1)
        Border: 1px solid rgba(166,227,161,0.25)
        Color: var(--ctp-green)
      
      Pill 3: "📍  Roadmap"
        Background: rgba(250,179,135,0.1)
        Border: 1px solid rgba(250,179,135,0.25)
        Color: var(--ctp-peach)
      
      Each pill: padding 5px 12px, border-radius 999px, font-size 12px

  DIVIDER: same as above

  BOTTOM SECTION — Stack Input (inline, no separate page):
    
    Label: "What's your current tech stack?"
      Font: 13px, weight 500, color: var(--ctp-subtext1)
      Margin-bottom: 8px

    Tag input container:
      Background: var(--ctp-surface0)
      Border: 1px solid var(--ctp-surface1)
      Border-radius: 10px
      Padding: 8px
      Min-height: 72px
      On focus: border-color var(--ctp-mauve)
      Transition: 150ms

      Skill tags inside:
        Background: rgba(203,166,247,0.2)
        Border: 1px solid rgba(203,166,247,0.4)
        Color: var(--ctp-mauve)
        Padding: 3px 10px, border-radius: 999px, font-size: 12px
        × button: color var(--ctp-overlay1), hover var(--ctp-red)

      Input inside container:
        Background: transparent, border: none
        Color: var(--ctp-text), font-size: 13px
        Placeholder color: var(--ctp-overlay0)

    Experience row (margin-top 12px):
      Label: "Experience" 12px var(--ctp-overlay1)
      Three buttons: Beginner · Intermediate · Advanced
      
      Inactive button:
        Border: 1px solid var(--ctp-surface2)
        Background: transparent
        Color: var(--ctp-subtext0)
        Padding: 6px 14px, border-radius: 6px, font-size: 12px
      
      Active button:
        Border: 1px solid var(--ctp-mauve)
        Background: rgba(203,166,247,0.15)
        Color: var(--ctp-mauve)

    Goal row (margin-top 8px):
      Label: "Looking for" 12px var(--ctp-overlay1)
      Four toggle buttons: Internship · Job · Hackathon · Freelance
      Same inactive/active style as experience

    CTA button (margin-top 20px):
      Full width, height: 44px
      Background: var(--ctp-mauve)
      Color: var(--ctp-base) — dark text on mauve button
      Border-radius: 10px, font-size: 14px, font-weight: 600
      Letter-spacing: -0.2px
      Hover: background var(--ctp-lavender), slight scale(1.01)
      Loading: "Building your graph..." with subtle spinner
      Disabled: opacity 0.4, cursor not-allowed
      No border

    Footer text below button:
      "Powered by HydraDB · Claude AI · WikiThon 2025"
      Font: 11px, color: var(--ctp-overlay0), text-align: center
      Margin-top: 12px

ANIMATION for opening screen:
  Card fades in from translateY(8px) opacity 0 to translateY(0) opacity 1
  Duration: 400ms, easing: cubic-bezier(0.16, 1, 0.3, 1)
  Background circles float with:
    @keyframes float {
      0%, 100% { transform: translateY(0px); }
      50% { transform: translateY(-20px); }
    }
    Each circle has different animation-duration (12s, 15s, 10s, 18s)
    and animation-delay (0s, 2s, 4s, 1s)

---

SIDEBAR — CATPPUCCIN MOCHA

Background: var(--ctp-mantle) #181825
Right border: 1px solid var(--ctp-surface0)

Logo section:
  "DevRadar" — font 15px weight 700 color var(--ctp-text)
  "WikiThon 2025" badge:
    Background: rgba(203,166,247,0.15)
    Color: var(--ctp-mauve), font-size: 10px
    Border: 1px solid rgba(203,166,247,0.25)
    Padding: 2px 8px, border-radius: 999px

Nav items:
  Height: 36px, padding: 0 12px, border-radius: 8px
  Font: 13px, color: var(--ctp-subtext0)
  Icon + label gap: 8px
  
  Hover: background var(--ctp-surface0), color var(--ctp-text)
  
  Active:
    Background: rgba(203,166,247,0.12)
    Color: var(--ctp-mauve)
    Left border: 2px solid var(--ctp-mauve)
    Font-weight: 500

Wiki section headers:
  Font: 10px uppercase, letter-spacing: 0.8px
  Color: var(--ctp-overlay0)
  Padding: 0 12px

Wiki entity items:
  Same as nav items but smaller (font 12px, height 28px)
  Color: var(--ctp-subtext0)
  Hover: background var(--ctp-surface0), color var(--ctp-text)

Count badges:
  Companies: background rgba(250,179,135,0.15) color var(--ctp-peach)
  Skills: background rgba(137,180,250,0.15) color var(--ctp-blue)
  Hackathons: background rgba(203,166,247,0.15) color var(--ctp-mauve)
  Gaps: background rgba(243,139,168,0.15) color var(--ctp-red)

Bottom status:
  Text: var(--ctp-overlay0), font 11px
  Green dot: #A6E3A1 (ctp-green), pulse animation

---

TOPBAR — CATPPUCCIN MOCHA

Background: var(--ctp-mantle)
Bottom border: 1px solid var(--ctp-surface0)
Height: 48px

Current view title: var(--ctp-text), 14px weight 600

Stack pills:
  Background: rgba(137,180,250,0.12)
  Border: 1px solid rgba(137,180,250,0.2)
  Color: var(--ctp-blue)
  Padding: 2px 10px, border-radius: 999px, font-size: 11px

---

CAREER GRAPH — CATPPUCCIN MOCHA

Canvas background: var(--ctp-base) #1E1E2E

vis-network / sigma config:
  Font color: var(--ctp-text) #CDD6F4
  Font stroke: var(--ctp-base) #1E1E2E (for readability)
  Edge default: var(--ctp-surface2) #585B70, opacity 0.7
  
  Node shadow: color #11111B, enabled true
  
  Node groups:
    user:      background #F9E2AF (ctp-yellow), border #F5E0DC
    companies: background #FAB387 (ctp-peach), border #EBA0AC
    skills:    background #89B4FA (ctp-blue), border #74C7EC
    gaps:      background #F38BA8 (ctp-red), border #EBA0AC, dashes [4,2]
    hackathons:background #CBA6F7 (ctp-mauve), border #B4BEFE

Tooltip (vis-network override):
  .vis-tooltip {
    background: var(--ctp-surface0) !important;
    border: 1px solid var(--ctp-surface2) !important;
    border-radius: 8px !important;
    color: var(--ctp-text) !important;
    font-family: Inter, sans-serif !important;
    font-size: 12px !important;
    padding: 8px 12px !important;
    box-shadow: 0 4px 20px rgba(17,17,27,0.6) !important;
  }

Node legend:
  Background: var(--ctp-surface0)
  Border: 1px solid var(--ctp-surface1)
  Border-radius: 8px
  Color: var(--ctp-subtext1)
  Title: var(--ctp-overlay1) 10px uppercase

Minimap:
  Background: var(--ctp-mantle)
  Border: 1px solid var(--ctp-surface0)

---

DETAIL PANEL — CATPPUCCIN MOCHA

Background: var(--ctp-mantle)
Left border: 1px solid var(--ctp-surface0)
Box-shadow: -4px 0 20px rgba(17,17,27,0.5)

Header:
  Border-bottom: 1px solid var(--ctp-surface0)
  Type badge colors:
    companies: bg rgba(250,179,135,0.15) color ctp-peach border rgba(250,179,135,0.25)
    skills:    bg rgba(137,180,250,0.15) color ctp-blue border rgba(137,180,250,0.25)
    gaps:      bg rgba(243,139,168,0.15) color ctp-red border rgba(243,139,168,0.25)
    hackathons:bg rgba(203,166,247,0.15) color ctp-mauve border rgba(203,166,247,0.25)

Close button: color var(--ctp-overlay1), hover ctp-red

Section dividers: 1px solid var(--ctp-surface0)
Section headers: var(--ctp-overlay0), 10px uppercase

Content text: var(--ctp-text)
Secondary text: var(--ctp-subtext1)

Skill pills:
  Known: bg rgba(166,227,161,0.15) border rgba(166,227,161,0.3) color ctp-green
  Missing: bg rgba(243,139,168,0.15) border rgba(243,139,168,0.3) color ctp-red

Company chips:
  bg ctp-surface0, border ctp-surface1, color ctp-text
  hover: bg ctp-surface1

Action buttons:
  Primary: bg ctp-mauve, color ctp-base, hover bg ctp-lavender
  Secondary: border ctp-surface2, color ctp-subtext1, hover bg ctp-surface0

---

INGEST PANEL — CATPPUCCIN MOCHA

Background: var(--ctp-base)
Max-width: 640px, centered

Title: var(--ctp-text) 22px weight 700
Subtitle: var(--ctp-subtext0) 13px

Tab switcher:
  Container: bg ctp-surface0, border-radius 10px, padding 4px
  Active tab: bg ctp-surface1, color ctp-text, border-radius 8px, shadow
  Inactive tab: color ctp-overlay1, hover color ctp-subtext0

Textarea / URL input:
  Background: var(--ctp-surface0)
  Border: 1px solid var(--ctp-surface1)
  Color: var(--ctp-text)
  Placeholder: var(--ctp-overlay0)
  Border-radius: 10px, padding: 12px
  Focus: border-color var(--ctp-mauve), outline none

Upload area:
  Border: 2px dashed var(--ctp-surface2)
  Color: var(--ctp-overlay1)
  Border-radius: 10px
  Hover: border-color var(--ctp-mauve)

Process button: same as CTA — ctp-mauve background, ctp-base text

Activity log:
  Background: var(--ctp-surface0)
  Border: 1px solid var(--ctp-surface1)
  Border-radius: 8px
  Time text: var(--ctp-overlay0)
  Message text: var(--ctp-subtext1)

Success result:
  Background: rgba(166,227,161,0.08)
  Border: 1px solid rgba(166,227,161,0.25)
  Color: var(--ctp-green)

Error:
  Background: rgba(243,139,168,0.08)
  Border: 1px solid rgba(243,139,168,0.25)
  Color: var(--ctp-red)

---

CHAT INTERFACE — CATPPUCCIN MOCHA

Background: var(--ctp-base)

User message bubble:
  Background: var(--ctp-mauve)
  Color: var(--ctp-base)
  Border-radius: 16px 16px 4px 16px

Assistant message bubble:
  Background: var(--ctp-surface0)
  Border: 1px solid var(--ctp-surface1)
  Color: var(--ctp-text)
  Border-radius: 16px 16px 16px 4px

Citation chips:
  Background: var(--ctp-surface1)
  Border: 1px solid var(--ctp-surface2)
  Color: var(--ctp-subtext1)
  Font: 11px mono

Suggestion buttons:
  Border: 1px solid var(--ctp-surface2)
  Color: var(--ctp-blue)
  Background: rgba(137,180,250,0.05)
  Hover: background rgba(137,180,250,0.12)

Input bar:
  Background: var(--ctp-mantle)
  Border-top: 1px solid var(--ctp-surface0)
  Input field: bg ctp-surface0, border ctp-surface1, color ctp-text
  Focus border: ctp-mauve
  Send button: bg ctp-mauve, color ctp-base

Typing dots: color var(--ctp-mauve)

---

ROADMAP VIEW — CATPPUCCIN MOCHA

Background: var(--ctp-base)

Immediate hackathon banner:
  Background: rgba(250,179,135,0.08)
  Border: 1px solid rgba(250,179,135,0.25)
  Title: var(--ctp-peach)

Week cards:
  Border: 1px solid var(--ctp-surface1)
  Border-radius: 12px
  
  Header section:
    Background: var(--ctp-surface0)
    Color: var(--ctp-text)
  
  Body section:
    Background: var(--ctp-mantle)
  
  Resource icons:
    📖 docs: var(--ctp-sapphire)
    🎥 video: var(--ctp-red)
    🛠 practice: var(--ctp-green)
    📝 blog: var(--ctp-yellow)
  
  Resource links: color var(--ctp-blue), hover underline
  
  Hackathon target pill:
    Background: rgba(203,166,247,0.1)
    Border: 1px solid rgba(203,166,247,0.2)
    Color: var(--ctp-mauve)
  
  Milestone text: var(--ctp-overlay1), border-top ctp-surface0

Company readiness section:
  Background: var(--ctp-surface0)
  Border: 1px solid var(--ctp-surface1)
  Color: var(--ctp-text)

---

JOURNEY VIEW — CATPPUCCIN MOCHA

Background: var(--ctp-base)

Stats grid cards:
  Background: var(--ctp-surface0)
  Border: 1px solid var(--ctp-surface1)
  Number: var(--ctp-text) 24px bold
  Label: var(--ctp-overlay0) 11px

Timeline vertical line: var(--ctp-surface1)

Timeline dots:
  companies: var(--ctp-peach) #FAB387
  skills: var(--ctp-blue) #89B4FA
  gaps: var(--ctp-red) #F38BA8
  hackathons: var(--ctp-mauve) #CBA6F7
  default: var(--ctp-overlay0)

Event label: var(--ctp-text) 13px
Event time: var(--ctp-overlay0) 11px

---

MEMORY BADGE — CATPPUCCIN MOCHA

Background: rgba(249,226,175,0.06)
Border-bottom: 1px solid rgba(249,226,175,0.15)
Left section icon + text: var(--ctp-yellow) #F9E2AF

Message text: var(--ctp-subtext1)

Urgent pills:
  Red (< 3 days): bg rgba(243,139,168,0.15) color ctp-red border rgba(243,139,168,0.3)
  Orange (< 7 days): bg rgba(250,179,135,0.15) color ctp-peach border rgba(250,179,135,0.3)

Bottom bar text: var(--ctp-overlay0) 11px
HydraDB active dot: var(--ctp-green) pulse animation

---

SCROLLBAR STYLING

::-webkit-scrollbar { width: 6px; height: 6px; }
::-webkit-scrollbar-track { background: var(--ctp-base); }
::-webkit-scrollbar-thumb {
  background: var(--ctp-surface2);
  border-radius: 999px;
}
::-webkit-scrollbar-thumb:hover { background: var(--ctp-overlay0); }

---

SELECTION COLOR

::selection {
  background: rgba(203,166,247,0.3);
  color: var(--ctp-text);
}

---

TYPOGRAPHY

Font: Inter from Google Fonts (already installed)
All text uses catppuccin variables — no hardcoded colors
Line-height: 1.6 for body text
Monospace: JetBrains Mono or system-mono for code/citations

---

RESPONSIVE CATPPUCCIN ADJUSTMENTS

Mobile bottom tab bar:
  Background: var(--ctp-mantle)
  Top border: 1px solid var(--ctp-surface0)
  Active tab: color var(--ctp-mauve)
  Inactive tab: color var(--ctp-overlay1)

Mobile detail panel (bottom sheet):
  Background: var(--ctp-mantle)
  Drag handle: var(--ctp-surface2)
  Border-radius top: 16px

---

COMPLETE index.css FILE

Replace existing index.css with:

@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --ctp-rosewater: #F5E0DC;
  --ctp-flamingo:  #F2CDCD;
  --ctp-pink:      #F5C2E7;
  --ctp-mauve:     #CBA6F7;
  --ctp-red:       #F38BA8;
  --ctp-maroon:    #EBA0AC;
  --ctp-peach:     #FAB387;
  --ctp-yellow:    #F9E2AF;
  --ctp-green:     #A6E3A1;
  --ctp-teal:      #94E2D5;
  --ctp-sky:       #89DCEB;
  --ctp-sapphire:  #74C7EC;
  --ctp-blue:      #89B4FA;
  --ctp-lavender:  #B4BEFE;
  --ctp-text:      #CDD6F4;
  --ctp-subtext1:  #BAC2DE;
  --ctp-subtext0:  #A6ADC8;
  --ctp-overlay2:  #9399B2;
  --ctp-overlay1:  #7F849C;
  --ctp-overlay0:  #6C7086;
  --ctp-surface2:  #585B70;
  --ctp-surface1:  #45475A;
  --ctp-surface0:  #313244;
  --ctp-base:      #1E1E2E;
  --ctp-mantle:    #181825;
  --ctp-crust:     #11111B;
}

* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  background: var(--ctp-base);
  color: var(--ctp-text);
  -webkit-font-smoothing: antialiased;
  line-height: 1.6;
}

::selection { background: rgba(203,166,247,0.3); color: var(--ctp-text); }

::-webkit-scrollbar { width: 5px; height: 5px; }
::-webkit-scrollbar-track { background: var(--ctp-base); }
::-webkit-scrollbar-thumb { background: var(--ctp-surface2); border-radius: 999px; }
::-webkit-scrollbar-thumb:hover { background: var(--ctp-overlay0); }

.vis-tooltip {
  background: var(--ctp-surface0) !important;
  border: 1px solid var(--ctp-surface2) !important;
  border-radius: 8px !important;
  color: var(--ctp-text) !important;
  font-family: 'Inter', sans-serif !important;
  font-size: 12px !important;
  padding: 8px 12px !important;
  box-shadow: 0 4px 20px rgba(17,17,27,0.6) !important;
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-20px); }
}

@keyframes pulse-dot {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

@keyframes fade-up {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-fade-up { animation: fade-up 400ms cubic-bezier(0.16,1,0.3,1) forwards; }
.animate-pulse-dot { animation: pulse-dot 2s ease-in-out infinite; }

---

OPENING SCREEN COMPONENT

Create frontend/src/components/OpeningScreen.jsx:

import { useState, useRef, useEffect } from "react";
import axios from "axios";

const API = import.meta.env.VITE_API_URL || "http://localhost:3001";

const SKILL_SUGGESTIONS = [
  "React", "Node.js", "Python", "TypeScript", "JavaScript",
  "Go", "Java", "Docker", "PostgreSQL", "MongoDB",
  "AWS", "GraphQL", "Next.js", "Vue.js", "Django"
];

export default function OpeningScreen({ onComplete }) {
  const [skills, setSkills] = useState([]);
  const [input, setInput] = useState("");
  const [suggestions, setSuggestions] = useState([]);
  const [experience, setExperience] = useState("beginner");
  const [goals, setGoals] = useState([]);
  const [loading, setLoading] = useState(false);
  const inputRef = useRef(null);

  const handleInput = (val) => {
    setInput(val);
    if (val.trim().length > 0) {
      const matches = SKILL_SUGGESTIONS.filter(s =>
        s.toLowerCase().includes(val.toLowerCase()) && !skills.includes(s)
      );
      setSuggestions(matches.slice(0, 5));
    } else {
      setSuggestions([]);
    }
  };

  const addSkill = (skill) => {
    if (skill && !skills.includes(skill) && skills.length < 15) {
      setSkills([...skills, skill]);
    }
    setInput("");
    setSuggestions([]);
    inputRef.current?.focus();
  };

  const removeSkill = (skill) => setSkills(skills.filter(s => s !== skill));

  const toggleGoal = (goal) => {
    setGoals(prev => prev.includes(goal) ? prev.filter(g => g !== goal) : [...prev, goal]);
  };

  const handleKeyDown = (e) => {
    if ((e.key === "Enter" || e.key === ",") && input.trim()) {
      e.preventDefault();
      addSkill(input.trim().replace(",", ""));
    }
  };

  const handleSubmit = async () => {
    if (skills.length === 0) return;
    setLoading(true);
    try {
      const res = await axios.post(`${API}/api/user/init`, {
        stack: skills,
        experience,
        goals
      });
      localStorage.setItem("devradar_userId", res.data.userId);
      onComplete({ userId: res.data.userId, stack: skills, experience, goals });
    } catch (err) {
      console.error("Init failed:", err);
    } finally {
      setLoading(false);
    }
  };

  const EXPERIENCES = ["Beginner", "Intermediate", "Advanced"];
  const GOALS = ["Internship", "Job", "Hackathon", "Freelance"];

  return (
    <div className="min-h-screen flex items-center justify-center p-4 relative overflow-hidden"
         style={{ background: "var(--ctp-base)" }}>

      {/* Floating background circles */}
      {[
        { size: 320, color: "var(--ctp-peach)", top: "10%", left: "5%", delay: "0s", dur: "12s" },
        { size: 240, color: "var(--ctp-mauve)", top: "60%", left: "80%", delay: "2s", dur: "15s" },
        { size: 180, color: "var(--ctp-blue)", top: "30%", right: "8%", delay: "4s", dur: "10s" },
        { size: 280, color: "var(--ctp-teal)", bottom: "15%", left: "30%", delay: "1s", dur: "18s" },
      ].map((c, i) => (
        <div key={i} className="absolute rounded-full pointer-events-none"
             style={{
               width: c.size, height: c.size,
               background: c.color, opacity: 0.05,
               top: c.top, left: c.left, right: c.right, bottom: c.bottom,
               animation: `float ${c.dur} ease-in-out ${c.delay} infinite`,
               filter: "blur(40px)"
             }} />
      ))}

      {/* Main card */}
      <div className="animate-fade-up relative z-10 w-full max-w-md rounded-2xl p-10"
           style={{
             background: "var(--ctp-mantle)",
             border: "1px solid var(--ctp-surface1)",
             boxShadow: "0 4px 20px rgba(17,17,27,0.6), 0 0 40px rgba(203,166,247,0.05)"
           }}>

        {/* Logo */}
        <div className="flex items-center gap-3 mb-1">
          <div className="flex gap-1">
            {["var(--ctp-peach)", "var(--ctp-blue)", "var(--ctp-mauve)"].map((c, i) => (
              <div key={i} className="w-2.5 h-2.5 rounded-sm"
                   style={{ background: c, opacity: 0.9 }} />
            ))}
          </div>
          <span className="text-xl font-bold tracking-tight"
                style={{ color: "var(--ctp-text)" }}>DevRadar</span>
        </div>

        <p className="text-sm mb-3" style={{ color: "var(--ctp-subtext0)" }}>
          Your personal career knowledge base
        </p>

        <div className="inline-block text-xs px-2.5 py-1 rounded-full mb-6"
             style={{
               background: "rgba(203,166,247,0.12)",
               border: "1px solid rgba(203,166,247,0.25)",
               color: "var(--ctp-mauve)"
             }}>
          WikiThon 2025
        </div>

        {/* Divider */}
        <div className="mb-6" style={{ borderTop: "1px solid var(--ctp-surface0)" }} />

        {/* Description */}
        <h2 className="text-lg font-semibold mb-2" style={{ color: "var(--ctp-text)" }}>
          Build your career graph
        </h2>
        <p className="text-xs mb-4 leading-relaxed" style={{ color: "var(--ctp-subtext0)" }}>
          Feed job descriptions, LinkedIn posts, or screenshots.
          DevRadar extracts insights and builds a personal knowledge graph
          powered by HydraDB memory.
        </p>

        {/* Feature pills */}
        <div className="flex gap-2 mb-6 flex-wrap">
          {[
            { label: "🗺  Career Graph", color: "var(--ctp-blue)", bg: "rgba(137,180,250,0.1)", border: "rgba(137,180,250,0.25)" },
            { label: "💬  Ask Anything", color: "var(--ctp-green)", bg: "rgba(166,227,161,0.1)", border: "rgba(166,227,161,0.25)" },
            { label: "📍  Roadmap", color: "var(--ctp-peach)", bg: "rgba(250,179,135,0.1)", border: "rgba(250,179,135,0.25)" },
          ].map(p => (
            <span key={p.label} className="text-xs px-3 py-1.5 rounded-full"
                  style={{ color: p.color, background: p.bg, border: `1px solid ${p.border}` }}>
              {p.label}
            </span>
          ))}
        </div>

        {/* Divider */}
        <div className="mb-5" style={{ borderTop: "1px solid var(--ctp-surface0)" }} />

        {/* Stack input */}
        <label className="block text-xs font-medium mb-2"
               style={{ color: "var(--ctp-subtext1)" }}>
          What's your current tech stack?
        </label>

        <div className="relative">
          <div className="flex flex-wrap gap-1.5 p-2 rounded-xl min-h-16 cursor-text"
               style={{
                 background: "var(--ctp-surface0)",
                 border: `1px solid ${skills.length > 0 ? "var(--ctp-mauve)" : "var(--ctp-surface1)"}`,
                 transition: "border-color 150ms"
               }}
               onClick={() => inputRef.current?.focus()}>
            {skills.map(skill => (
              <span key={skill} className="flex items-center gap-1 text-xs px-2.5 py-1 rounded-full"
                    style={{
                      background: "rgba(203,166,247,0.18)",
                      border: "1px solid rgba(203,166,247,0.35)",
                      color: "var(--ctp-mauve)"
                    }}>
                {skill}
                <button onClick={() => removeSkill(skill)}
                        className="ml-0.5 hover:opacity-70 leading-none"
                        style={{ color: "var(--ctp-overlay1)" }}>×</button>
              </span>
            ))}
            <input ref={inputRef} value={input}
                   onChange={e => handleInput(e.target.value)}
                   onKeyDown={handleKeyDown}
                   placeholder={skills.length === 0 ? "Type a skill and press Enter..." : ""}
                   className="flex-1 min-w-24 bg-transparent border-none outline-none text-xs"
                   style={{ color: "var(--ctp-text)" }} />
          </div>

          {/* Autocomplete dropdown */}
          {suggestions.length > 0 && (
            <div className="absolute top-full left-0 right-0 mt-1 rounded-xl overflow-hidden z-20"
                 style={{
                   background: "var(--ctp-surface0)",
                   border: "1px solid var(--ctp-surface1)",
                   boxShadow: "0 4px 20px rgba(17,17,27,0.6)"
                 }}>
              {suggestions.map(s => (
                <button key={s} onClick={() => addSkill(s)}
                        className="w-full text-left text-xs px-3 py-2.5 hover:opacity-80 transition-opacity"
                        style={{ color: "var(--ctp-text)", background: "transparent" }}>
                  {s}
                </button>
              ))}
            </div>
          )}
        </div>

        {/* Experience */}
        <div className="mt-4">
          <label className="text-xs mb-2 block" style={{ color: "var(--ctp-overlay1)" }}>
            Experience level
          </label>
          <div className="flex gap-2">
            {EXPERIENCES.map(e => (
              <button key={e} onClick={() => setExperience(e.toLowerCase())}
                      className="flex-1 text-xs py-1.5 rounded-lg transition-all"
                      style={{
                        background: experience === e.toLowerCase()
                          ? "rgba(203,166,247,0.15)" : "transparent",
                        border: `1px solid ${experience === e.toLowerCase()
                          ? "var(--ctp-mauve)" : "var(--ctp-surface2)"}`,
                        color: experience === e.toLowerCase()
                          ? "var(--ctp-mauve)" : "var(--ctp-subtext0)"
                      }}>
                {e}
              </button>
            ))}
          </div>
        </div>

        {/* Goals */}
        <div className="mt-3">
          <label className="text-xs mb-2 block" style={{ color: "var(--ctp-overlay1)" }}>
            Looking for
          </label>
          <div className="flex gap-2 flex-wrap">
            {GOALS.map(g => (
              <button key={g} onClick={() => toggleGoal(g.toLowerCase())}
                      className="text-xs px-3 py-1.5 rounded-lg transition-all"
                      style={{
                        background: goals.includes(g.toLowerCase())
                          ? "rgba(203,166,247,0.15)" : "transparent",
                        border: `1px solid ${goals.includes(g.toLowerCase())
                          ? "var(--ctp-mauve)" : "var(--ctp-surface2)"}`,
                        color: goals.includes(g.toLowerCase())
                          ? "var(--ctp-mauve)" : "var(--ctp-subtext0)"
                      }}>
                {g}
              </button>
            ))}
          </div>
        </div>

        {/* CTA */}
        <button onClick={handleSubmit}
                disabled={skills.length === 0 || loading}
                className="w-full mt-6 py-3 rounded-xl text-sm font-semibold transition-all"
                style={{
                  background: skills.length === 0 ? "var(--ctp-surface1)" : "var(--ctp-mauve)",
                  color: skills.length === 0 ? "var(--ctp-overlay0)" : "var(--ctp-base)",
                  cursor: skills.length === 0 ? "not-allowed" : "pointer",
                  opacity: loading ? 0.7 : 1,
                  letterSpacing: "-0.2px"
                }}>
          {loading ? "Building your graph..." : "Start → Build Career Graph"}
        </button>

        <p className="text-center mt-3 text-xs" style={{ color: "var(--ctp-overlay0)" }}>
          Powered by HydraDB · Claude AI · WikiThon 2025
        </p>
      </div>
    </div>
  );
}

---

RETURNING USER SCREEN

If userId exists in localStorage, show a different opening:

Create frontend/src/components/ReturningScreen.jsx:

export default function ReturningScreen({ returnContext, onContinue }) {
  return (
    <div className="min-h-screen flex items-center justify-center p-4"
         style={{ background: "var(--ctp-base)" }}>

      {/* Same floating circles as OpeningScreen */}

      <div className="animate-fade-up relative z-10 w-full max-w-sm rounded-2xl p-8 text-center"
           style={{
             background: "var(--ctp-mantle)",
             border: "1px solid var(--ctp-surface1)",
             boxShadow: "0 4px 20px rgba(17,17,27,0.6)"
           }}>

        <div className="text-3xl mb-4">🧠</div>

        <h2 className="text-lg font-semibold mb-2"
            style={{ color: "var(--ctp-text)" }}>
          Welcome back
        </h2>

        <p className="text-sm mb-4 leading-relaxed"
           style={{ color: "var(--ctp-subtext0)" }}>
          {returnContext?.message || "HydraDB remembers your career journey."}
        </p>

        {returnContext?.urgentItems?.length > 0 && (
          <div className="flex flex-wrap gap-2 justify-center mb-4">
            {returnContext.urgentItems.map((item, i) => (
              <span key={i} className="text-xs px-3 py-1 rounded-full"
                    style={{
                      background: item.daysLeft < 3
                        ? "rgba(243,139,168,0.15)" : "rgba(250,179,135,0.15)",
                      border: `1px solid ${item.daysLeft < 3
                        ? "rgba(243,139,168,0.3)" : "rgba(250,179,135,0.3)"}`,
                      color: item.daysLeft < 3
                        ? "var(--ctp-red)" : "var(--ctp-peach)"
                    }}>
                {item.name} · {item.daysLeft}d left
              </span>
            ))}
          </div>
        )}

        <button onClick={onContinue}
                className="w-full py-3 rounded-xl text-sm font-semibold"
                style={{ background: "var(--ctp-mauve)", color: "var(--ctp-base)" }}>
          Continue →
        </button>

        <div className="mt-3 flex items-center justify-center gap-1.5">
          <div className="w-1.5 h-1.5 rounded-full animate-pulse-dot"
               style={{ background: "var(--ctp-green)" }} />
          <span className="text-xs" style={{ color: "var(--ctp-overlay0)" }}>
            HydraDB memory loaded
          </span>
        </div>
      </div>
    </div>
  );
}

---

APP.JSX FLOW UPDATE

On app load:
  Check localStorage for userId
  If found: fetch returnContext → show ReturningScreen
  If not: show OpeningScreen
  After either: show main three-zone layout

const existingUserId = localStorage.getItem("devradar_userId");

if (!userId) {
  if (existingUserId) {
    // Fetch return context then show ReturningScreen
    return <ReturningScreen returnContext={returnContext} onContinue={...} />;
  } else {
    return <OpeningScreen onComplete={...} />;
  }
}
// else show main layout

---

GIT COMMITS FOR THIS WORK

"add catppuccin mocha css variables to index.css"
"restyle sidebar with catppuccin colors"
"restyle topbar with catppuccin"
"apply catppuccin to detail panel"
"apply catppuccin to ingest panel"
"apply catppuccin to chat interface"
"apply catppuccin to roadmap view"
"apply catppuccin to journey view"
"apply catppuccin to memory badge"
"create opening screen with floating nodes animation"
"create returning user screen"
"wire opening and returning screens in app jsx"
"fix graph tooltip catppuccin override"
"final catppuccin polish pass"
```
