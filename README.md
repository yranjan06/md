# DevRadar — End-to-End Testing Prompt
### Local testing · Groq API (free) · Claude Browser navigation

---

## PART 1 — BEFORE TESTING: MISSING FEATURES TO BUILD

These are missing right now. Build them first, then test.

---

### MISSING FEATURE 1 — Skill Selection UI

Currently: User types skills manually in a text input.
Problem: No visual skill picker, no categories, no "currently learning" vs "know well".

Build this in OpeningScreen.jsx:

```
SKILL SELECTION — THREE STATES:

State 1: Know well (solid filled)
State 2: Currently learning (half filled / dashed border)
State 3: Not selected (empty)

Click once → Know well (blue filled)
Click twice → Currently learning (blue dashed)
Click thrice → Deselect

Show skills in categories:

CATEGORY: Frontend
  React, Vue.js, Next.js, TypeScript, JavaScript, Tailwind CSS, HTML/CSS

CATEGORY: Backend
  Node.js, Python, Java, Go, Django, FastAPI, Express.js

CATEGORY: Database
  PostgreSQL, MongoDB, MySQL, Redis, Firebase

CATEGORY: DevOps & Cloud
  Docker, AWS, Linux, Git, Kubernetes, CI/CD

CATEGORY: Other
  DSA, System Design, REST APIs, GraphQL, Machine Learning

UI for each skill button:
  Inactive:
    border: 1px solid var(--ctp-surface2)
    background: transparent
    color: var(--ctp-overlay1)
    padding: 6px 14px, border-radius: 8px, font-size: 12px

  Know well (state 1):
    border: 1px solid var(--ctp-blue)
    background: rgba(137,180,250,0.15)
    color: var(--ctp-blue)
    Small ✓ icon left

  Currently learning (state 2):
    border: 1px dashed var(--ctp-yellow)
    background: rgba(249,226,175,0.1)
    color: var(--ctp-yellow)
    Small ⏳ icon left

Store in state:
  { know_well: ["React", "Node.js"], learning: ["TypeScript", "Docker"] }

Send both to backend on submit.
```

---

### MISSING FEATURE 2 — What User Is Looking For

Currently: Simple goal buttons (Internship, Job, Hackathon, Freelance).
Problem: Too generic. Need more context.

Add these fields to OpeningScreen after goal selection:

```
FIELD 1 — Target Role (text input, optional):
  Placeholder: "e.g. Frontend Developer, Full Stack, ML Engineer"
  This helps Claude give more specific gap analysis

FIELD 2 — Target Companies (multi-select chips, optional):
  Show pre-loaded list: Razorpay, Groww, CRED, Zepto, Meesho, CRED, etc.
  User can select up to 3
  These become priority in gap analysis and roadmap

FIELD 3 — Timeline (slider or buttons):
  How soon are you looking?
  [ Immediately ] [ 1-3 months ] [ 3-6 months ] [ Just exploring ]

FIELD 4 — Learning style preference:
  [ Video courses ] [ Documentation ] [ Projects ] [ All of the above ]
  This affects what resources Claude recommends in roadmap

Store all in user profile and send to backend.
Add to /api/user/init body:
  {
    stack: skills.know_well,
    learning_stack: skills.learning,
    experience,
    goals,
    target_role: "Frontend Developer",
    target_companies: ["razorpay", "groww"],
    timeline: "1-3 months",
    learning_style: "video"
  }
```

---

### MISSING FEATURE 3 — Groq API Integration

Replace Claude API with Groq (free) for local testing.

Create backend/groq.js:

```javascript
const fetch = require('node-fetch');

const GROQ_API_KEY = process.env.GROQ_API_KEY;
const GROQ_BASE_URL = 'https://api.groq.com/openai/v1/chat/completions';
const MODEL = 'llama-3.3-70b-versatile'; // Free, fast, good quality

async function groqChat(systemPrompt, userMessage, maxTokens = 1000) {
  try {
    const response = await fetch(GROQ_BASE_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${GROQ_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: MODEL,
        messages: [
          { role: 'system', content: systemPrompt },
          { role: 'user', content: userMessage }
        ],
        max_tokens: maxTokens,
        temperature: 0.3
      })
    });
    const data = await response.json();
    return data.choices?.[0]?.message?.content || '';
  } catch (error) {
    console.error('[Groq] Error:', error.message);
    return null;
  }
}

async function ingestStep1(rawInput, existingWikiSummary, userStack) {
  const system = `You are analyzing career data for an Indian developer.
Student stack: ${JSON.stringify(userStack)}.
Respond ONLY with valid JSON. No extra text. No markdown fences.`;

  const user = `Analyze this career data:
${rawInput}

Return JSON:
{
  "companies_found": [{"name":"","role":"","skills_required":[],"salary_range":"","location":"","interview_topics":[],"match_with_student":0,"missing_skills":[]}],
  "hackathons_found": [{"name":"","deadline":"","skills_needed":[],"prize":"","platform":"","match_with_student":0}],
  "skills_identified": [{"name":"","student_has":false,"reason":""}],
  "gaps_identified": [{"skill":"","companies_needing_it":[],"learning_time_weeks":0,"difficulty":""}],
  "wiki_pages_to_create": [{"key":"","title":"","type":""}],
  "summary": ""
}`;

  const result = await groqChat(system, user, 1500);
  try {
    return JSON.parse(result.replace(/```json|```/g, '').trim());
  } catch {
    return { companies_found: [], hackathons_found: [], skills_identified: [], gaps_identified: [], wiki_pages_to_create: [], summary: 'Parse failed' };
  }
}

async function ingestStep2GeneratePage(pageInfo, analysis, userStack) {
  const system = `Generate a career wiki page in markdown with YAML frontmatter.
Include [[wikilinks]] to related pages.
Respond ONLY with markdown. No extra text.`;

  const user = `Generate wiki page:
Key: ${pageInfo.key}
Type: ${pageInfo.type}
Title: ${pageInfo.title}
Data: ${JSON.stringify(analysis).slice(0, 1000)}
Student stack: ${JSON.stringify(userStack)}`;

  return await groqChat(system, user, 800) ||
    `---\ntype: ${pageInfo.type}\ntitle: ${pageInfo.title}\n---\n\n# ${pageInfo.title}`;
}

async function queryWiki(question, wikiPages, userContext) {
  const pagesContext = wikiPages.slice(0, 8).map((p, i) =>
    `[${i+1}] ${p.key}:\n${(p.content || '').slice(0, 400)}`
  ).join('\n\n---\n\n');

  const system = `You are DevRadar AI for Indian developers.
User: ${JSON.stringify(userContext)}.
Answer from wiki. Cite sources as [1][2]. Be specific and actionable.`;

  const answer = await groqChat(system,
    `Wiki:\n${pagesContext}\n\nQuestion: ${question}`, 1000);

  const citations = wikiPages.slice(0, 8).map((p, i) => ({
    index: i + 1, key: p.key, preview: (p.content || '').slice(0, 80)
  }));

  return { answer: answer || 'Could not process.', citations };
}

async function generateRoadmap(userStack, gapPages, companyPages, hackathonPages) {
  const system = `Generate a week-by-week career roadmap for Indian developer.
Include real URLs for resources. Respond ONLY with valid JSON. No markdown fences.`;

  const user = `Stack: ${JSON.stringify(userStack)}
Gaps: ${JSON.stringify(gapPages.slice(0,3).map(p=>p.key))}
Companies: ${companyPages.slice(0,3).map(p=>p.key).join(', ')}
Hackathons: ${hackathonPages.slice(0,3).map(p=>p.key).join(', ')}

Return JSON:
{
  "weeks": [{"week_range":"Week 1-2","focus_skill":"TypeScript","why":"","daily_time_hours":2,"resources":[{"type":"video","title":"","url":""}],"hackathon_to_target":"","milestone":""}],
  "immediate_hackathons": [{"name":"","deadline":"","match":0,"register_url":""}],
  "company_readiness": [{"company":"","ready_in":"","after_skill":""}],
  "overall_timeline_weeks": 8
}`;

  const result = await groqChat(system, user, 1500);
  try {
    return JSON.parse(result.replace(/```json|```/g, '').trim());
  } catch {
    return { weeks: [], immediate_hackathons: [], company_readiness: [], overall_timeline_weeks: 8 };
  }
}

module.exports = { groqChat, ingestStep1, ingestStep2GeneratePage, queryWiki, generateRoadmap };
```

Add to .env:
```
GROQ_API_KEY=your_groq_key_here
```

Get free key from: console.groq.com (free, no credit card needed)

In server.js replace claude requires with groq:
```javascript
const { ingestStep1, ingestStep2GeneratePage, queryWiki, generateRoadmap } = require('./groq');
```

---

## PART 2 — DEMO DATA FOR TESTING

Use this exact data throughout all tests.

```
TEST USER — ARJUN:
  name: Arjun Sharma
  stack know_well: ["React", "Node.js", "Python"]
  stack learning: ["TypeScript", "Docker"]
  experience: beginner
  goals: ["internship", "hackathon"]
  target_role: Frontend Developer
  target_companies: ["razorpay", "groww"]
  timeline: 1-3 months
  learning_style: video
  userId: test_user_arjun_001 (store in localStorage)

DEMO JOB DESCRIPTION TO PASTE (Razorpay):
---
Senior Frontend Engineer — Razorpay
Location: Bangalore (Hybrid)
Experience: 1-2 years

We are looking for a frontend engineer to join our payments team.

Requirements:
  - Strong React experience (hooks, context, performance optimization)
  - TypeScript proficiency — strict mode preferred
  - GraphQL basics
  - Understanding of system design
  - REST API integration experience

Nice to have:
  - PostgreSQL basics
  - Redis knowledge
  - Docker for local dev

Salary: 15-25 LPA
Interview process: 4 rounds
  Round 1: DSA (Arrays, Trees — Leetcode medium)
  Round 2: System Design (for freshers — basic)
  Round 3: React deep dive (custom hooks, optimization)
  Round 4: HR + culture fit

About Razorpay: India's leading payment gateway. Fast paced fintech.
Apply: careers.razorpay.com
---

DEMO URL TO FETCH:
  https://devfolio.co/hackathons (Devfolio hackathon listing)
  OR
  https://unstop.com/hackathons (Unstop listing)

DEMO SCREENSHOT:
  Take screenshot of any LinkedIn job post visible on screen
  Or use a saved image of a job description

DEMO CHAT QUESTIONS:
  Q1: "Which hackathon should I register for right now?"
  Q2: "Am I ready for Razorpay? What's missing?"
  Q3: "What should I learn this week?"
  Q4: "Compare Groww vs Razorpay for my profile"
  Q5: "How long before I can apply to CRED?"
```

---

## PART 3 — LOCAL SETUP BEFORE TESTING

Run these commands before starting tests:

```bash
# Terminal 1 — Backend
cd devradar/backend
npm install
cp .env.example .env
# Add GROQ_API_KEY to .env
node server.js
# Should see: DevRadar Backend running on port 3001

# Terminal 2 — Frontend
cd devradar/frontend
npm install
npm run dev
# Should see: Local: http://localhost:5173

# Terminal 3 — Seed demo data
cd devradar/backend
node seed-demo.js
# Should see: Demo user seeded
```

Verify backend is working:
```bash
curl http://localhost:3001/api/health
# Expected: { status: "ok", hydradb: "active", ai: "groq" }
```

---

## PART 4 — CLAUDE BROWSER TESTING PROMPT

Paste this into Claude with browser access enabled.
Browser should be pointed at: http://localhost:5173

---

```
You are testing DevRadar end-to-end — a personal career knowledge base
for Indian developers built with React frontend, Node.js backend,
Groq AI (Llama 3.3), and HydraDB memory.

App is running locally at http://localhost:5173
Backend is at http://localhost:3001

Test user profile:
  Name: Arjun
  Stack: React, Node.js, Python (know well) + TypeScript, Docker (learning)
  Goal: Frontend Developer internship + hackathons
  Target: Razorpay, Groww

After each flow write exactly:
  FLOW [N] — [name]: PASS or FAIL
  If fail: what exactly failed, what was expected vs actual

Screenshot after each major screen change.

---

FLOW 1 — OPENING SCREEN

Navigate to http://localhost:5173

CHECK THESE ELEMENTS EXIST:
  DevRadar logo/wordmark visible
  WikiThon 2025 badge in mauve/purple
  Floating colored circles in background
  "Build your career graph" heading
  Three feature pills: Career Graph, Ask Anything, Roadmap
  Skill category sections: Frontend, Backend, Database, DevOps, Other
  Experience selector: Beginner, Intermediate, Advanced
  Goal buttons: Internship, Job, Hackathon, Freelance
  Target role text input
  Target companies multi-select
  Timeline selector
  CTA button: "Start → Build Career Graph" (disabled initially)

ACTIONS:
  Click "React" in Frontend category — verify it turns blue (know well)
  Click "React" again — verify it turns yellow dashed (learning)
  Click "React" again — verify it deselects
  Click "React" once more — set to know well (blue)
  
  Now select these skills:
    Know well (click once): React, Node.js, Python
    Learning (click twice): TypeScript, Docker
  
  Click "Beginner" experience
  Click "Internship" goal
  Click "Hackathon" goal (both should be active)
  
  Type "Frontend Developer" in target role field
  Click "Razorpay" in target companies
  Click "Groww" in target companies
  
  Click "1-3 months" timeline
  
  Verify CTA button is now enabled (not grayed out)
  
  Click "Start → Build Career Graph"
  Verify button shows loading state "Building your graph..."
  Wait for navigation to main app
  
VERIFY after submit:
  localStorage has devradar_userId set
  No console errors
  Navigation to graph view happens within 5 seconds

Screenshot: opening screen with all skills selected.
Screenshot: loading state.

---

FLOW 2 — MAIN LAYOUT LOAD

After navigation, verify three-zone layout:

SIDEBAR (left, ~240px wide):
  DevRadar wordmark visible
  WikiThon 2025 badge
  Nav items: Career Graph, Feed Data, Ask Anything, Roadmap, My Journey
  Career Graph is active (mauve color, left border)
  Wiki sections: Companies (0), Skills (0), Hackathons (0), Gaps (0)
  All wiki sections show "Feed data to populate" (empty)
  Bottom: HydraDB active green dot

TOPBAR (top, ~48px):
  "Career Graph" title
  Stack pills: React, Node.js, Python in blue

GRAPH AREA (center):
  Graph canvas visible (dark catppuccin background)
  One yellow center node "You" visible
  Empty graph message visible
  Node legend visible in bottom left:
    Yellow dot: You
    Blue dot: Skills you know
    Red dot: Skills to learn
    Orange dot: Companies
    Purple dot: Hackathons

VERIFY:
  No console errors
  All three zones render properly
  Colors match Catppuccin Mocha palette

Screenshot: full main layout.

---

FLOW 3 — INGEST: TEXT PASTE

Click "Feed Data" in sidebar.
Verify IngestPanel loads.

CHECK UI:
  Three tabs: Paste Text, Enter URL, Upload Image
  "Paste Text" tab is active by default
  Large textarea visible
  "Process & Add to Career Wiki" button visible

ACTION — Paste the Razorpay job description:
  Click in textarea
  Paste this exact text:
  
  "Senior Frontend Engineer — Razorpay
  Location: Bangalore (Hybrid)
  Requirements: React, TypeScript, GraphQL, System Design
  Salary: 15-25 LPA
  Interview: 4 rounds — DSA, System Design, React deep dive, HR"
  
  Click "Process & Add to Career Wiki"

VERIFY DURING PROCESSING:
  Button shows "Processing..."
  Activity log appears below button
  Log shows: "Reading text content..."
  Log shows: "Running analysis (Step 1)..."
  Log shows: "Extracting companies, skills, hackathons..."
  Log shows: "Generating wiki pages (Step 2)..."
  Log shows: "Created X wiki pages"
  Log shows: "Done!"

VERIFY SUCCESS RESULT:
  Green success box appears
  Shows: "Successfully processed"
  Shows: "1 companies found" (Razorpay)
  Shows: "2+ gaps identified" (TypeScript, GraphQL)
  Shows: "0 hackathons found"
  Shows wiki pages created list

VERIFY SIDEBAR UPDATED:
  Companies section: shows "razorpay" (count badge: 1)
  Skills section: new skills appeared
  Gaps section: TypeScript, GraphQL appeared

Click "Career Graph" in sidebar.
VERIFY GRAPH UPDATED:
  Orange node "Razorpay" appeared
  Red nodes for TypeScript, GraphQL appeared (gaps)
  Blue nodes for React, Node.js (skills Arjun knows)
  Edges connecting Razorpay to skills

Screenshot: ingest panel with success result.
Screenshot: graph after ingest with new nodes.

---

FLOW 4 — INGEST: URL

Go back to "Feed Data"
Click "Enter URL" tab

VERIFY:
  URL input field visible
  Platform hint badges: linkedin.com/jobs, devfolio.co, unstop.com, hackerearth.com

ACTION:
  Type: https://devfolio.co/hackathons
  Click "Process & Add to Career Wiki"

VERIFY:
  Processing activity log runs
  Success or graceful error if URL blocks fetch
  
  If success: hackathon nodes appear in graph
  If error (blocked): shows "Could not fetch URL. Try pasting the content instead."
  
  Either way — no crash, no console error

Screenshot: URL tab with URL entered.

---

FLOW 5 — INGEST: SCREENSHOT

Go back to "Feed Data"
Click "Upload Image" tab

VERIFY:
  Dashed upload area visible with camera icon
  "PNG, JPG, JPEG supported" text
  Click triggers file picker

ACTION:
  Take a screenshot of any job posting visible on screen
  Save as test-job.jpg
  Upload via the file picker

VERIFY:
  File name appears after selection
  Click "Process & Add to Career Wiki"
  Activity log shows: "Reading image..."
  Processing completes
  New wiki pages created from image content

Screenshot: upload tab with file selected.

---

FLOW 6 — GRAPH INTERACTION

Click "Career Graph" in sidebar.

VERIFY graph has nodes from previous ingests.

ACTION — Click Razorpay node (orange):
  Click the orange Razorpay node in graph

VERIFY:
  Razorpay node gets highlighted (white border)
  Other nodes fade to 20% opacity
  Razorpay's connected edges fully visible
  DetailPanel slides in from right
  
  DetailPanel shows:
    "STARTUP" badge in peach/orange
    "Razorpay" title
    Match percentage with progress bar
    Skills section: React ✓ (green), Node.js ✓ (green), TypeScript ✗ (red)
    Salary: 15-25 LPA
    Location: Bangalore
    Interview topics
    "Apply" button (mauve)
    "Save to Watchlist" button

ACTION — Click TypeScript red node:
  Click the red TypeScript node

VERIFY DetailPanel:
  "GAP SKILL" badge in red
  "TypeScript" title
  "Not in your stack" badge
  "WHY YOU NEED THIS" section with Razorpay chip
  Learning path: 2 weeks, Beginner friendly
  Resource link
  "Mark as Learned" button

ACTION — Click background (empty area):
  Click empty area of graph

VERIFY:
  All nodes return to full opacity
  DetailPanel closes
  No errors

ACTION — Click You (yellow center node):
  Click the amber/yellow center node

VERIFY DetailPanel:
  YOUR PROFILE section
  Stack pills: React, Node.js, Python
  HydraDB Memory section with timeline events
  Stats grid
  "Edit Stack" button

Screenshot: graph with Razorpay node selected and detail panel open.
Screenshot: graph with TypeScript gap node selected.

---

FLOW 7 — SIDEBAR NAVIGATION

Click "Razorpay" in Companies section of sidebar.

VERIFY:
  Graph view activates
  Razorpay node gets focused/highlighted
  DetailPanel opens with Razorpay details
  Same as clicking the node directly

Click "TypeScript" in Gaps section of sidebar.

VERIFY:
  TypeScript node focused in graph
  DetailPanel shows gap details

Click different nav items:
  "Ask Anything" — ChatInterface loads
  "Roadmap" — RoadmapView loads
  "My Journey" — JourneyView loads
  "Feed Data" — IngestPanel loads
  "Career Graph" — graph loads

VERIFY each view loads without errors.

Screenshot: sidebar with companies and gaps populated.

---

FLOW 8 — CHAT INTERFACE

Click "Ask Anything" in sidebar.

VERIFY:
  Chat interface loads
  Empty state: "Ask anything about your career"
  4 suggestion buttons visible with catppuccin styling
  Input bar at bottom

ACTION — Click suggestion "Which hackathon should I register for right now?":

VERIFY:
  Message appears in chat as user bubble (mauve background, dark text)
  Loading dots appear (mauve dots)
  Response arrives within 10 seconds
  Response is in assistant bubble (surface0 background)
  Response mentions actual hackathons if ingest found any
  OR says to feed more data first if wiki is empty
  Citation chips visible below response

ACTION — Type custom question:
  "Am I ready for Razorpay? What's missing?"
  Press Enter or click Send

VERIFY:
  Smooth chat flow
  Answer references Razorpay data from wiki
  Mentions TypeScript as gap (if ingest was done)
  Citations reference wiki pages

ACTION — Ask another:
  "What should I learn this week?"

VERIFY:
  Specific recommendation based on gaps found
  References time estimates
  Mentions relevant hackathon if available

Screenshot: chat with 2-3 messages and citations visible.

---

FLOW 9 — ROADMAP VIEW

Click "Roadmap" in sidebar.

VERIFY:
  "Generating your personalized roadmap..." loading state
  Wait up to 15 seconds for Groq to generate

  After load:
  "Your Career Roadmap" heading
  "Powered by Claude AI · Based on your career wiki" subtitle
  Total timeline weeks shown

  If wiki has data:
    Immediate hackathons section (orange/peach banner)
    At least 2-3 week cards
    Each week card has:
      Week range (Week 1-2)
      Focus skill name
      Why text
      Daily time hours
      Resource list with icons: 📖 📝 🎥 🛠
      At least one resource with real URL
      Hackathon to target (mauve pill)
      Milestone text
    Company readiness section

  If wiki empty:
    "Feed career data first to generate your roadmap"
    "Generate Roadmap" button

Click a resource URL (if available):
  Verify it opens in new tab

Screenshot: roadmap with week cards expanded.

---

FLOW 10 — JOURNEY VIEW

Click "My Journey" in sidebar.

VERIFY:
  "My Journey" heading
  "Everything DevRadar has learned about your career"
  Stats grid (4 cards):
    Wiki Pages count
    Companies count
    Skills Tracked count
    Gaps Found count

  Timeline of events (if ingest was done):
    Events listed newest first
    Each event has colored dot + label + timestamp
    Company events: peach dots
    Skill events: blue dots
    Gap events: red dots
    Hackathon events: mauve dots

  If no events: "No journey data yet" message

VERIFY counts match what was actually ingested:
  If 1 company ingested → Companies: 1
  Dots are correctly colored
  Timestamps are realistic

Screenshot: journey view with timeline events.

---

FLOW 11 — MARK AS LEARNED

Go to "Career Graph"
Click TypeScript red node
DetailPanel shows "Mark as Learned" button

ACTION — Click "Mark as Learned":

VERIFY:
  Button changes state (green check or grayed)
  Node color in graph changes from red to blue (live update)
  Edge between Razorpay and TypeScript changes from dashed to solid
  Sidebar "Gaps" count decreases by 1
  Sidebar "Skills" count increases by 1
  HydraDB saved (check console for log)

Screenshot: before marking learned (red node).
Screenshot: after marking learned (node turned blue).

---

FLOW 12 — RETURN VISIT (HydraDB Memory)

ACTION — Close the browser tab.
Open new tab, navigate to http://localhost:5173

VERIFY:
  App does NOT show OpeningScreen
  App shows ReturningScreen (new returning user component):
    Brain emoji 🧠
    "Welcome back" heading
    Memory message from HydraDB
    Urgent items if any hackathon deadlines close
    "Continue →" button
    Green pulsing "HydraDB memory loaded" indicator

Click "Continue →"

VERIFY:
  Main graph layout loads
  Graph has same nodes as before session ended
  Sidebar has same wiki pages
  MemoryBadge banner visible at top of graph area:
    Yellow/amber background
    "HydraDB recalls your journey" text
    The same memory message
    Urgent items as pills

Screenshot: ReturningScreen with memory message.
Screenshot: main layout with MemoryBadge visible.

---

FLOW 13 — MOBILE LAYOUT

Resize browser to 375px width.

VERIFY:
  Sidebar is hidden (not visible)
  Bottom tab bar appears at bottom (60px):
    5 tabs: 🗺 Career Graph, ➕ Feed Data, 💬 Ask, 📍 Roadmap, ⏱ Journey
    Active tab in mauve color
  Topbar shows view title on left, search icon right
  Content fills full width

Click each bottom tab:
  Verify correct view loads
  Verify no horizontal overflow

Click graph tab:
  Graph fills full screen
  Click a node
  Verify DetailPanel slides up from bottom as sheet (not from right)
  Sheet covers 70% of screen
  Drag handle at top of sheet

Screenshot: mobile view 375px with bottom tabs.
Screenshot: mobile with detail sheet open.

---

FLOW 14 — BACKEND API VERIFICATION

Open browser DevTools → Network tab
Clear all requests

Perform these actions and verify API calls:

Stack submission → POST /api/user/init
  Request body has: stack, learning_stack, experience, goals, target_role, target_companies
  Response has: userId, message

Ingest text → POST /api/ingest
  Request body has: userId, content, type: "text"
  Response has: success, pages_created, companies_found, gaps_identified

Graph load → GET /api/graph/:userId
  Response has: nodes array, edges array
  Nodes have: id, label, type properties

Chat question → POST /api/chat
  Request has: userId, question
  Response has: answer, citations array

Roadmap → GET /api/roadmap/:userId
  Response has: weeks array, immediate_hackathons, company_readiness

Wiki pages → GET /api/wiki-pages/:userId
  Response has: pages array, count

VERIFY no 500 errors on any endpoint.
VERIFY all responses are valid JSON.

Screenshot: Network tab showing successful API calls.

---

FLOW 15 — CONSOLE CHECK

Open DevTools → Console tab

VERIFY:
  Styled DevRadar log appears: colored background
  "Powered by HydraDB and Groq AI" log
  "WikiThon 2025" log
  No red errors
  HydraDB operation logs visible (blue colored)
  Groq API call logs visible

VERIFY no:
  Uncaught TypeError
  Network failed (besides expected CORS on external URLs)
  React key warnings (acceptable but note them)

Screenshot: clean console with styled logs.

---

END-TO-END TEST REPORT

After all 15 flows, write:

DEVRADAR E2E TEST REPORT
Date: [today]
Backend: Groq API (Llama 3.3-70b-versatile)
Frontend: React + Catppuccin Mocha

FLOW 1  Opening Screen:           PASS/FAIL
FLOW 2  Main Layout:              PASS/FAIL
FLOW 3  Ingest Text:              PASS/FAIL
FLOW 4  Ingest URL:               PASS/FAIL
FLOW 5  Ingest Screenshot:        PASS/FAIL
FLOW 6  Graph Interaction:        PASS/FAIL
FLOW 7  Sidebar Navigation:       PASS/FAIL
FLOW 8  Chat Interface:           PASS/FAIL
FLOW 9  Roadmap View:             PASS/FAIL
FLOW 10 Journey View:             PASS/FAIL
FLOW 11 Mark as Learned:          PASS/FAIL
FLOW 12 Return Visit Memory:      PASS/FAIL
FLOW 13 Mobile Layout:            PASS/FAIL
FLOW 14 API Verification:         PASS/FAIL
FLOW 15 Console Check:            PASS/FAIL

CRITICAL ISSUES (blocks demo):
  List anything broken that judges would immediately see

NON-CRITICAL ISSUES:
  List minor visual/UX issues

GROQ API STATUS:
  Response quality: Good/Acceptable/Poor
  Average response time: X seconds
  Any failures: Yes/No

HYDRADB STATUS:
  Memory persisted across session: Yes/No
  Return visit context loaded: Yes/No

MISSING FROM ORIGINAL PLAN:
  List features not yet implemented

READY FOR DEPLOYMENT: Yes/No
```

---

## PART 5 — QUICK GROQ SETUP (5 minutes)

```
1. Go to: console.groq.com
2. Sign up free (Google login works)
3. Go to: API Keys section
4. Create new key
5. Copy key
6. Add to backend/.env:
   GROQ_API_KEY=gsk_xxxxxxxxxxxx
7. Restart backend: node server.js
8. Test: curl http://localhost:3001/api/health
```

Free tier limits: 14,400 requests/day, 30 requests/minute.
More than enough for hackathon testing and demo.

---

## PART 6 — IF GROQ IS SLOW: FALLBACK DEMO DATA

If Groq is slow or fails during testing, inject this hardcoded response
in groq.js for ingestStep1 to test UI without waiting for API:

```javascript
// DEMO MODE — remove before submission
const DEMO_INGEST_RESPONSE = {
  companies_found: [{
    name: "Razorpay", role: "Frontend Engineer",
    skills_required: ["React", "TypeScript", "GraphQL"],
    salary_range: "15-25 LPA", location: "Bangalore",
    interview_topics: ["DSA", "System Design", "React"],
    match_with_student: 80, missing_skills: ["TypeScript", "GraphQL"]
  }],
  hackathons_found: [{
    name: "HackRx 6.0", deadline: "2025-08-05",
    skills_needed: ["React", "Node.js"], prize: "250000",
    platform: "Devfolio", match_with_student: 90
  }],
  skills_identified: [
    { name: "TypeScript", student_has: false, reason: "Required by Razorpay" },
    { name: "GraphQL", student_has: false, reason: "Required by Razorpay" }
  ],
  gaps_identified: [
    { skill: "TypeScript", companies_needing_it: ["Razorpay"], learning_time_weeks: 2, difficulty: "Beginner friendly" },
    { skill: "GraphQL", companies_needing_it: ["Razorpay"], learning_time_weeks: 3, difficulty: "Intermediate" }
  ],
  wiki_pages_to_create: [
    { key: "companies/razorpay", title: "Razorpay", type: "company" },
    { key: "gaps/typescript", title: "TypeScript Gap", type: "gap" },
    { key: "gaps/graphql", title: "GraphQL Gap", type: "gap" },
    { key: "hackathons/hackrx-6", title: "HackRx 6.0", type: "hackathon" }
  ],
  summary: "Found Razorpay frontend role. Main gaps: TypeScript and GraphQL. HackRx 6.0 is a strong immediate match."
};
```
