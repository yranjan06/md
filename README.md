Fix the core UX problem in DevRadar:

PROBLEM:
App shows career graph and startups without knowing who the user is.
User never told the app their stack, goals, or what they want.
seed-demo.js pre-fills fake data — judges will notice this is fake.

SOLUTION:
Multi-step onboarding wizard that runs ONCE on first visit.
User tells the app everything. App stores in HydraDB. Then shows graph.
On return visit: HydraDB remembers — skip onboarding, go straight to graph.

---

STEP 1 — FIX App.jsx LOGIC

Current broken flow:
  App loads → shows graph (with fake seed data)

Correct flow:
  App loads
    ↓
  Check localStorage for "devradar_userId"
    ↓ found               ↓ not found
  Fetch return context   Show OnboardingWizard
    ↓
  Show ReturningScreen
    ↓
  User clicks Continue
    ↓
  Show main graph

Update App.jsx:

const [appState, setAppState] = useState("checking")
// States: checking | onboarding | returning | app

useEffect(() => {
  const existingId = localStorage.getItem("devradar_userId")
  if (!existingId) {
    setAppState("onboarding")
    return
  }
  // Has userId — fetch return context
  axios.get(`${API}/api/return-context/${existingId}`)
    .then(res => {
      setUserId(existingId)
      setReturnContext(res.data)
      setAppState(res.data.hasHistory ? "returning" : "app")
    })
    .catch(() => {
      // If fetch fails — still go to app
      setUserId(existingId)
      setAppState("app")
    })
}, [])

// Render based on state:
if (appState === "checking") return <LoadingScreen />
if (appState === "onboarding") return <OnboardingWizard onComplete={handleOnboardComplete} />
if (appState === "returning") return <ReturningScreen returnContext={returnContext} onContinue={() => setAppState("app")} />
if (appState === "app") return <MainAppLayout ... />

function handleOnboardComplete(userData) {
  setUserId(userData.userId)
  setUserStack(userData.stack)
  localStorage.setItem("devradar_userId", userData.userId)
  setAppState("app")
}

---

STEP 2 — CREATE frontend/src/components/OnboardingWizard.jsx

Four-step wizard. User goes through all four before seeing the app.
Progress bar at top shows which step they are on.
Cannot skip. Each step validates before Next.

Props: onComplete(userData)

STATE:
  const [step, setStep] = useState(1)          // 1, 2, 3, 4
  const [knowWell, setKnowWell] = useState([]) // skills user knows
  const [learning, setLearning] = useState([]) // skills user is learning
  const [experience, setExperience] = useState(null)
  const [targetRole, setTargetRole] = useState("")
  const [lookingFor, setLookingFor] = useState([]) // internship, job, hackathon
  const [targetCompanies, setTargetCompanies] = useState([])
  const [timeline, setTimeline] = useState(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState("")

LAYOUT:
  Full screen, bg: var(--bg-base)
  Center card: max-width 540px, bg: var(--bg-mantle)
  
  TOP of card:
    DevRadar wordmark left
    Step indicator right: "Step 2 of 4"
    Progress bar below: fills 25% per step
    Color: var(--accent)

  BOTTOM of card:
    Back button (left) — only on steps 2,3,4
    Next/Finish button (right) — disabled until step is valid

---

STEP 2a — STEP 1: Who are you?

Title: "Let's start with you"
Subtitle: "This helps DevRadar personalize everything"

Fields:

  Name (optional):
    Label: "What should we call you?"
    Input: text, placeholder "Arjun, Priya, your name..."
    Not required — can skip

  Experience level (REQUIRED):
    Label: "Where are you right now?"
    Four options as cards (not buttons):
    
    [ 🎓 Student ]
      "Currently in college, learning to code"
    
    [ 🌱 Fresher ]  
      "Just graduated, looking for first job"
    
    [ 💼 Working ]
      "1-3 years experience, want to grow"
    
    [ 🚀 Switching ]
      "Coming from another domain"
    
    Card style:
      width: 100%, padding: 12px 16px
      background: var(--bg-surface0)
      border: 2px solid var(--border)
      border-radius: 10px
      text-align: left
      emoji large left, text right
      
      Active selected card:
        border-color: var(--border-active)
        background: var(--accent-bg)
        Emoji and title in var(--accent)

  VALIDATION: Experience must be selected to proceed.

---

STEP 2b — STEP 2: Your current skills

Title: "What do you know right now?"
Subtitle: "Be honest — this is just for you"

Instruction text:
  "Click once — you know it well ✓
   Click twice — you're currently learning it ⏳
   Click again — remove it"

Skill grid by category.
Each category is collapsible (open by default):

CATEGORY: Frontend
  React, Vue.js, Next.js, Angular, TypeScript,
  JavaScript, HTML/CSS, Tailwind CSS, Redux

CATEGORY: Backend  
  Node.js, Python, Java, Go, Rust, Django,
  FastAPI, Express.js, Spring Boot, PHP

CATEGORY: Database
  PostgreSQL, MongoDB, MySQL, Redis,
  Firebase, SQLite, Supabase

CATEGORY: Cloud & DevOps
  AWS, Docker, Kubernetes, Git,
  Linux, CI/CD, GCP, Azure

CATEGORY: CS Fundamentals
  DSA, System Design, REST APIs,
  GraphQL, Microservices, OS basics

CATEGORY: Other / Emerging
  Machine Learning, TensorFlow, Solidity,
  Flutter, React Native, Web3

Each skill button:
  Default (not selected):
    bg: var(--bg-surface0)
    border: 1px solid var(--border)
    color: var(--text-muted)
    padding: 6px 14px, border-radius: 8px, font-size: 13px
  
  State 1 — Know well (click once):
    bg: var(--info-bg)
    border: 1px solid var(--info-border)
    color: var(--info)
    Left icon: "✓" small

  State 2 — Currently learning (click twice):
    bg: var(--warning-bg)
    border: 1px dashed var(--warning-border)
    color: var(--warning)
    Left icon: "⏳" small

  State 3 — Click again deselects back to default

Live summary below grid:
  "You know: React, Node.js, Python"     (info color)
  "Learning: TypeScript, Docker"         (warning color)

VALIDATION: At least 1 skill selected to proceed.

---

STEP 2c — STEP 3: What are you looking for?

Title: "What's your goal right now?"
Subtitle: "Pick everything that applies"

Section 1 — Looking for (REQUIRED, multi-select):
  
  Four option cards in 2x2 grid:
  
  [ 🎓 Internship ]         [ 💼 Full-time Job ]
  "First industry           "Permanent role at
   experience"               a company"
  
  [ 🏆 Hackathons ]         [ 🔄 Freelance ]
  "Win prizes, build        "Project-based work,
   portfolio, network"       independence"

  Selected: accent border + accent bg

Section 2 — Target role (optional):
  Label: "Any specific role in mind?"
  Input text: placeholder "e.g. Frontend Developer, Backend Engineer, Full Stack, ML Engineer"
  Helper text below: "Leave blank if unsure — DevRadar will suggest based on your skills"

Section 3 — Dream companies (optional):
  Label: "Any companies you have in mind?"
  Helper: "We'll prioritize these in your gap analysis"
  
  Chips from pre-loaded list:
    Razorpay, Groww, CRED, Zepto, Meesho, PhonePe,
    Swiggy, Zomato, Ola, BrowserStack, Postman,
    Freshworks, Unacademy, Flipkart, Amazon, Google
  
  Click chip to select (max 3):
    Selected: accent bg + accent border + "×" to remove
    Unselected: surface bg + border
  
  Or type custom company name:
    Small input below chips: "Other company..."

VALIDATION: At least 1 goal selected to proceed.

---

STEP 2d — STEP 4: Timeline + Confirm

Title: "Almost done!"
Subtitle: "Last two things"

Section 1 — Timeline (REQUIRED):
  Label: "When are you looking to land something?"
  
  Four option buttons horizontal row:
  [ Right now ] [ 1-3 months ] [ 3-6 months ] [ Just exploring ]
  
  Style same as experience cards but smaller

Section 2 — Summary preview (auto-generated):
  Show a preview card of what will be stored:
  
  Card: bg var(--bg-surface0), border var(--border), rounded-xl, padding 16px
  
  Title: "Your DevRadar Profile"
  
  Rows:
    Experience:  [selected experience]
    Know well:   React, Node.js, Python (max 4, "+N more")
    Learning:    TypeScript (max 2, "+N more")
    Looking for: Internship, Hackathons
    Timeline:    1-3 months
    Companies:   Razorpay, Groww
  
  Below card:
    Small text: "This is saved privately in HydraDB.
                 Only you can see it. You can update it anytime."

Section 3 — Finish button:
  Full width, large, bg var(--accent), color white
  Text: "Build my career graph →"
  
  On click:
    setLoading(true)
    POST /api/user/init with all collected data
    On success: call onComplete(userData)
    On error: show error message, setLoading(false)

LOADING STATE while submitting:
  Button shows spinner + "Setting up your career wiki..."
  Disable all inputs

---

STEP 3 — UPDATE /api/user/init TO ACCEPT FULL DATA

Update backend/server.js:

app.post('/api/user/init', async (req, res) => {
  try {
    const {
      name,
      experience,
      stack,           // know_well skills array
      learning_stack,  // learning skills array
      goals,           // ["internship", "hackathon"]
      target_role,     // "Frontend Developer"
      target_companies, // ["razorpay", "groww"]
      timeline         // "1-3 months"
    } = req.body

    // Validate required fields
    if (!experience) {
      return res.status(400).json({ error: "Experience level is required" })
    }
    if (!stack || stack.length === 0) {
      return res.status(400).json({ error: "At least one skill is required" })
    }
    if (!goals || goals.length === 0) {
      return res.status(400).json({ error: "At least one goal is required" })
    }
    if (!timeline) {
      return res.status(400).json({ error: "Timeline is required" })
    }

    const userId = require('uuid').v4()

    // Store full profile in HydraDB
    await initUser({
      userId,
      name: name || "Developer",
      experience,
      stack,
      learning_stack: learning_stack || [],
      goals,
      target_role: target_role || "",
      target_companies: target_companies || [],
      timeline,
      created_at: new Date().toISOString()
    })

    // Create initial career wiki overview page
    const overviewContent = `---
type: overview
userId: ${userId}
name: ${name || "Developer"}
experience: ${experience}
created: ${new Date().toISOString().split("T")[0]}
---

# Career Overview

**Experience Level:** ${experience}
**Target Role:** ${target_role || "Not specified yet"}
**Timeline:** ${timeline}

## Current Stack
${stack.map(s => `- [[skills/${s.toLowerCase().replace(/ /g, "-")}]] ✓`).join("\n")}

## Currently Learning
${(learning_stack || []).map(s => `- [[skills/${s.toLowerCase().replace(/ /g, "-")}]] ⏳`).join("\n")}

## Goals
${goals.map(g => `- ${g}`).join("\n")}

## Target Companies
${(target_companies || []).map(c => `- [[companies/${c}]]`).join("\n")}

_Feed job descriptions, URLs, or screenshots to build your career wiki._`

    await saveWikiPage(userId, "overview", overviewContent)
    await appendToLog(userId, {
      action: "onboarding_complete",
      detail: `Profile created. Stack: ${stack.join(", ")}. Goals: ${goals.join(", ")}.`
    })

    console.log(`[Init] New user: ${userId}, stack: ${stack.join(", ")}`)

    res.json({
      userId,
      message: "Profile created",
      name: name || "Developer",
      stack,
      learning_stack,
      goals,
      target_companies
    })

  } catch (error) {
    console.error("[/api/user/init] error:", error)
    res.status(500).json({ error: "Failed to create profile" })
  }
})

---

STEP 4 — REMOVE seed-demo.js FROM AUTO-RUN

In package.json, remove any seed script from "start" or "dev".
seed-demo.js should only run manually: node seed-demo.js

Also: clear any pre-seeded HydraDB data to test clean onboarding.

---

STEP 5 — UPDATE /api/analyze TO USE USER PROFILE

Currently analyze pulls from JSON files generically.
Now it should filter based on user's actual target companies and goals.

app.post('/api/analyze', async (req, res) => {
  const { userId } = req.body
  const user = await getUser(userId)
  
  // user.stack = their actual skills
  // user.target_companies = their preferred companies
  // user.experience = beginner/fresher/working/switching
  // user.goals = ["internship", "hackathon"]
  
  const startups = require('./data/startups.json')
  
  // Filter: if user has target companies, show those first
  let ordered = [...startups]
  if (user.target_companies?.length > 0) {
    ordered.sort((a, b) => {
      const aTarget = user.target_companies.includes(a.id) ? -1 : 0
      const bTarget = user.target_companies.includes(b.id) ? -1 : 0
      return aTarget - bTarget
    })
  }
  
  // Filter by experience: beginner only sees 0-1yr roles
  if (user.experience === "student" || user.experience === "fresher") {
    ordered = ordered.filter(s =>
      s.min_experience === "0-1 years" || s.min_experience === "Fresher"
    )
  }
  
  // Match stack
  const withMatch = ordered.map(startup => {
    const userSkills = [...(user.stack || []), ...(user.learning_stack || [])]
    const matching = startup.skills_required.filter(s => userSkills.includes(s))
    const missing = startup.skills_required.filter(s => !userSkills.includes(s))
    const match_percentage = Math.round((matching.length / startup.skills_required.length) * 100)
    return { ...startup, match_percentage, matching_skills: matching, missing_skills: missing }
  })
  .sort((a, b) => b.match_percentage - a.match_percentage)
  
  res.json({ startups: withMatch, user_profile: { stack: user.stack, goals: user.goals } })
})

---

STEP 6 — SHOW PERSONALIZED EMPTY STATE

When user first completes onboarding and graph loads:

Graph is empty (no wiki pages yet).
Show a helpful empty state — NOT a generic "no data" message.

EmptyGraphState component:

Show inside graph canvas when graphData.nodes.length <= 1:

  Icon: 🗺️ large
  
  Title: "Hey {user.name}! Your career graph is ready to build."
  
  Message: "You've told us your stack. Now feed DevRadar
            some career data to grow your graph."
  
  Three action cards in a row:
  
  Card 1 — Paste a job description:
    Icon: 📋
    Title: "Job Description"
    Text: "Copy-paste from LinkedIn, Naukri, or any job board"
    Button: "Paste JD →" → navigates to Feed Data
  
  Card 2 — Share a company URL:
    Icon: 🔗
    Title: "Company URL"
    Text: "LinkedIn job post, company careers page, Devfolio hackathon"
    Button: "Add URL →" → navigates to Feed Data
  
  Card 3 — Upload a screenshot:
    Icon: 📸
    Title: "Screenshot"
    Text: "Screenshot of a job post, tweet, or anything career-related"
    Button: "Upload →" → navigates to Feed Data
  
  Below the three cards:
    "Or start by exploring" text
    Two pill links:
      "Browse matching startups →"
      "Find hackathons →"

Style:
  Cards: bg var(--bg-mantle), border var(--border), rounded-xl, padding 16px
  Action buttons: var(--accent) color text, no bg, underline hover
  Position: absolute center of graph canvas

---

STEP 7 — ONBOARDING COMPLETION ANIMATION

When user clicks "Build my career graph →":

While loading:
  Wizard card stays visible
  Button spins: "Setting up your career wiki..."
  Small progress steps appear below button:
    ✓ Creating your profile...
    ✓ Saving to HydraDB...
    ✓ Building your wiki overview...
    ⏳ Preparing your career graph...

After success:
  Card fades out
  Graph canvas fades in
  Empty state shows with personalized greeting
  Smooth 400ms transition

---

STEP 8 — RETURNING USER KNOWS THEIR STACK

On return visit, ReturningScreen shows user's own stack:

"Welcome back, Arjun!

Your stack: React · Node.js · Python
Still learning: TypeScript · Docker

Last visit: 3 days ago
Progress since then: 1 wiki page added

[Continue to your graph →]"

This is pulled from HydraDB — the actual data the user entered.
NOT demo seed data.

---

COMPLETE USER JOURNEY — FIRST TIME

Step 1: User opens http://localhost:5173
  → OnboardingWizard Step 1 (Who are you?)
  → User selects "Student" + types name "Arjun"
  → Clicks Next

Step 2: OnboardingWizard Step 2 (Skills)
  → Arjun clicks React once (know well)
  → Clicks Node.js once (know well)
  → Clicks Python once (know well)
  → Clicks TypeScript twice (learning)
  → Summary shows: "You know: React, Node.js, Python | Learning: TypeScript"
  → Clicks Next

Step 3: OnboardingWizard Step 3 (Goals)
  → Clicks "Internship" card
  → Clicks "Hackathons" card
  → Types "Frontend Developer" in role field
  → Clicks Razorpay chip
  → Clicks Groww chip
  → Clicks Next

Step 4: OnboardingWizard Step 4 (Timeline + Confirm)
  → Clicks "1-3 months"
  → Sees profile preview card
  → Clicks "Build my career graph →"

After submit:
  → POST /api/user/init called with full data
  → userId stored in localStorage
  → HydraDB stores full profile
  → Wiki overview page created
  → Graph loads with empty state
  → "Hey Arjun! Your career graph is ready to build."
  → Arjun feeds a job description
  → Graph populates with his actual data

COMPLETE USER JOURNEY — RETURN VISIT

User opens http://localhost:5173 3 days later
  → localStorage has userId
  → Return context fetched from HydraDB
  → ReturningScreen shows:
      "Welcome back, Arjun!
       You added Razorpay wiki page last session.
       TypeScript is still in your learning list."
  → User clicks Continue
  → Graph loads with Razorpay node visible from last session
  → MemoryBadge shows same context

---

GIT COMMITS

"fix app state machine for proper onboarding flow"
"create OnboardingWizard step 1 who are you"
"create OnboardingWizard step 2 skill selection grid"
"create OnboardingWizard step 3 goals and companies"
"create OnboardingWizard step 4 timeline confirm"
"update user init api to accept full onboarding data"
"filter analyze results by user experience and goals"
"add empty graph state with feed prompts"
"remove seed demo from auto-run"
"test full first-time user flow end to end"
"test return visit shows actual user data not demo"
