# DevRadar — End-to-End Modification Prompt
### Existing repo: https://github.com/iamabhaydawar/devradar

Paste this entire prompt into Claude Code or any AI coding tool.
Work on the cloned repo. Do not start fresh.

---

```
You are modifying an existing project called DevRadar.

Repo: https://github.com/iamabhaydawar/devradar
Clone it first, then make all changes described below.

DevRadar is already a working app with:
- Node.js + Express backend
- React 18 + Tailwind CSS frontend
- HydraDB for memory
- Claude API for AI
- 20 startups, 15 hackathons, 30 skills in JSON files
- 8 working API endpoints
- Deployed on Vercel + Render

We are upgrading it to implement the LLM Wiki pattern for careers.
The core idea: instead of a simple dashboard, DevRadar becomes a
personal career knowledge base that builds itself as the student
feeds it data — job descriptions, LinkedIn URLs, screenshots.

New features to add:
1. Multi-format ingest (text + URL + screenshot)
2. Two-step LLM Wiki ingest pipeline
3. Career wiki pages stored in HydraDB
4. sigma.js knowledge graph visualization
5. Wiki-based chat with citations
6. Structured roadmap with resources
7. Notion-style white light theme UI

---

IMPORTANT RULES

Never delete existing working code. Only extend it.
Keep all existing API endpoints exactly as they are.
Keep all existing data JSON files exactly as they are.
Keep seed-demo.js, vercel.json, render.yaml unchanged.
Keep StackInput.jsx working — just restyle it for light theme.
Keep MemoryBadge.jsx logic — just restyle for light theme.
Git commits: lowercase, no emoji, no conventional prefixes,
             commit every 30-60 minutes of work.

---

STEP 1 — INSTALL NEW PACKAGES

In backend directory:
npm install node-fetch turndown

In frontend directory:
npm install sigma graphology graphology-layout-forceatlas2

---

STEP 2 — CREATE backend/fetcher.js

New file. Handles external content fetching.

const fetch = require('node-fetch');
const TurndownService = require('turndown');
const turndown = new TurndownService();

async function fetchURL(url) {
  try {
    const response = await fetch(url, {
      headers: {
        'User-Agent': 'Mozilla/5.0 (compatible; DevRadar/1.0)'
      },
      timeout: 10000
    });
    const html = await response.text();
    const markdown = turndown.turndown(html);
    // Clean up: remove nav, footer noise, keep main content
    const lines = markdown.split('\n')
      .filter(line => line.trim().length > 10)
      .slice(0, 200); // max 200 lines
    return lines.join('\n');
  } catch (error) {
    console.error('fetchURL error:', error.message);
    return null;
  }
}

function detectInputType(input) {
  if (!input) return 'text';
  const trimmed = input.trim();
  if (trimmed.startsWith('http://') || trimmed.startsWith('https://')) {
    return 'url';
  }
  if (trimmed.startsWith('data:image') || trimmed.length > 1000 &&
      /^[A-Za-z0-9+/=]+$/.test(trimmed.replace(/\s/g, ''))) {
    return 'screenshot';
  }
  return 'text';
}

module.exports = { fetchURL, detectInputType };

---

STEP 3 — EXTEND backend/hydradb.js

Add these new functions to the existing file.
Do not remove any existing functions.

// --- WIKI STORAGE FUNCTIONS ---

const WIKI_PREFIX = 'wiki_';

async function saveWikiPage(userId, pageKey, content) {
  const key = WIKI_PREFIX + userId + '_' + pageKey.replace(/\//g, '_');
  try {
    await client.set(key, JSON.stringify({
      key: pageKey,
      content,
      updated_at: new Date().toISOString()
    }));
    console.log('[HydraDB] Wiki page saved:', pageKey);
    return true;
  } catch (error) {
    console.error('[HydraDB] saveWikiPage error:', error.message);
    LOCAL_FALLBACK.set(key, { key: pageKey, content,
      updated_at: new Date().toISOString() });
    return true;
  }
}

async function getWikiPage(userId, pageKey) {
  const key = WIKI_PREFIX + userId + '_' + pageKey.replace(/\//g, '_');
  try {
    const result = await client.get(key);
    return result ? JSON.parse(result) : null;
  } catch (error) {
    return LOCAL_FALLBACK.get(key) || null;
  }
}

async function getAllWikiPages(userId) {
  try {
    const prefix = WIKI_PREFIX + userId + '_';
    const keys = await client.list(prefix);
    const pages = [];
    for (const key of (keys || [])) {
      const page = await client.get(key);
      if (page) pages.push(JSON.parse(page));
    }
    return pages;
  } catch (error) {
    const pages = [];
    for (const [key, value] of LOCAL_FALLBACK.entries()) {
      if (key.startsWith(WIKI_PREFIX + userId)) pages.push(value);
    }
    return pages;
  }
}

async function updateWikiIndex(userId, newPageEntry) {
  let index = await getWikiPage(userId, 'index') || { content: '# Career Wiki Index\n', key: 'index' };
  if (!index.content.includes(newPageEntry.key)) {
    index.content += `\n- [[${newPageEntry.key}]] — ${newPageEntry.title || newPageEntry.key}`;
    await saveWikiPage(userId, 'index', index.content);
  }
}

async function appendToLog(userId, logEntry) {
  let log = await getWikiPage(userId, 'log') || { content: '# Career Log\n', key: 'log' };
  const entry = `\n[${new Date().toISOString()}] ${logEntry.action}: ${logEntry.detail}`;
  log.content += entry;
  await saveWikiPage(userId, 'log', log.content);
}

async function getGraphData(userId) {
  const pages = await getAllWikiPages(userId);
  const nodes = [];
  const edges = [];
  const nodeSet = new Set();

  for (const page of pages) {
    if (!page.key || page.key === 'index' || page.key === 'log' || page.key === 'overview') continue;
    
    const parts = page.key.split('/');
    const type = parts[0]; // companies, skills, hackathons, gaps
    const name = parts[1] || page.key;
    
    if (!nodeSet.has(page.key)) {
      nodeSet.add(page.key);
      nodes.push({
        id: page.key,
        label: name.replace(/-/g, ' '),
        type: type,
        content_preview: (page.content || '').slice(0, 200)
      });
    }

    // Extract [[wikilinks]] from content and create edges
    const wikilinks = (page.content || '').match(/\[\[([^\]]+)\]\]/g) || [];
    for (const link of wikilinks) {
      const target = link.replace('[[', '').replace(']]', '');
      if (nodeSet.has(target) || pages.find(p => p.key === target)) {
        edges.push({ from: page.key, to: target, type: 'wikilink' });
      }
    }
  }

  // Add user center node
  nodes.unshift({ id: 'user_center', label: 'You', type: 'user' });

  return { nodes, edges };
}

module.exports = {
  // ... existing exports
  saveWikiPage,
  getWikiPage,
  getAllWikiPages,
  updateWikiIndex,
  appendToLog,
  getGraphData
};

---

STEP 4 — EXTEND backend/claude.js

Add these new functions. Keep all existing ones.

const Anthropic = require('@anthropic-ai/sdk');
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

async function ingestStep1(rawInput, existingWikiSummary, userStack) {
  try {
    const response = await client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2000,
      system: `You are analyzing career data for an Indian developer student.
               The student's current stack: ${JSON.stringify(userStack)}.
               Existing wiki summary: ${existingWikiSummary || 'None yet'}.
               Respond ONLY with valid JSON. No extra text.`,
      messages: [{
        role: 'user',
        content: `Analyze this career data and extract structured information:

${rawInput}

Return JSON:
{
  "companies_found": [
    {
      "name": "Razorpay",
      "role": "Frontend Engineer",
      "skills_required": ["React", "TypeScript"],
      "salary_range": "15-25 LPA",
      "location": "Bangalore",
      "interview_topics": ["DSA", "System Design"],
      "match_with_student": 80,
      "missing_skills": ["TypeScript"]
    }
  ],
  "hackathons_found": [
    {
      "name": "ETHIndia 2025",
      "deadline": "2025-08-15",
      "skills_needed": ["React", "Solidity"],
      "prize": "500000",
      "platform": "Devfolio",
      "match_with_student": 72
    }
  ],
  "skills_identified": [
    { "name": "TypeScript", "student_has": false, "reason": "Required by Razorpay" }
  ],
  "gaps_identified": [
    {
      "skill": "TypeScript",
      "companies_needing_it": ["Razorpay", "CRED"],
      "learning_time_weeks": 2,
      "difficulty": "Beginner friendly"
    }
  ],
  "wiki_pages_to_create": [
    { "key": "companies/razorpay", "title": "Razorpay", "type": "company" },
    { "key": "gaps/typescript", "title": "TypeScript Gap", "type": "gap" }
  ],
  "summary": "One paragraph summary of what was found"
}`
      }]
    });
    
    const text = response.content[0].text;
    return JSON.parse(text.replace(/```json|```/g, '').trim());
  } catch (error) {
    console.error('[Claude] ingestStep1 error:', error.message);
    return { companies_found: [], hackathons_found: [], skills_identified: [],
             gaps_identified: [], wiki_pages_to_create: [], summary: 'Processing failed' };
  }
}

async function ingestStep2GeneratePage(pageInfo, analysis, userStack) {
  try {
    const response = await client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1000,
      system: 'Generate a career wiki page in markdown format with YAML frontmatter. Include [[wikilinks]] to connect related pages. Respond ONLY with the markdown content.',
      messages: [{
        role: 'user',
        content: `Generate a wiki page for: ${pageInfo.key}
Type: ${pageInfo.type}
Title: ${pageInfo.title}
Analysis data: ${JSON.stringify(analysis)}
Student stack: ${JSON.stringify(userStack)}

Format:
---
type: ${pageInfo.type}
title: ${pageInfo.title}
updated: ${new Date().toISOString().split('T')[0]}
---

# ${pageInfo.title}

[content with [[wikilinks]] to related pages]`
      }]
    });
    return response.content[0].text;
  } catch (error) {
    console.error('[Claude] ingestStep2 error:', error.message);
    return `---\ntype: ${pageInfo.type}\ntitle: ${pageInfo.title}\n---\n\n# ${pageInfo.title}\n\nProcessing failed.`;
  }
}

async function queryWiki(question, wikiPages, userContext) {
  try {
    const pagesContext = wikiPages.slice(0, 10).map((p, i) =>
      `[${i+1}] ${p.key}:\n${(p.content || '').slice(0, 500)}`
    ).join('\n\n---\n\n');

    const response = await client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1500,
      system: `You are DevRadar's AI assistant answering questions about a student's career.
               User context: ${JSON.stringify(userContext)}.
               Answer using the wiki pages provided. Always cite sources as [1], [2], etc.
               Be specific and actionable.`,
      messages: [{
        role: 'user',
        content: `Wiki pages available:\n${pagesContext}\n\nQuestion: ${question}`
      }]
    });

    const answer = response.content[0].text;
    const citations = wikiPages.slice(0, 10).map((p, i) => ({
      index: i + 1,
      key: p.key,
      preview: (p.content || '').slice(0, 100)
    }));

    return { answer, citations };
  } catch (error) {
    console.error('[Claude] queryWiki error:', error.message);
    return { answer: 'Could not process your question. Please try again.', citations: [] };
  }
}

async function generateRoadmap(userStack, gapPages, companyPages, hackathonPages) {
  try {
    const response = await client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2000,
      system: 'Generate a structured week-by-week career roadmap for an Indian developer student. Include real resources, YouTube videos, and relevant hackathons. Respond ONLY with valid JSON.',
      messages: [{
        role: 'user',
        content: `Student stack: ${JSON.stringify(userStack)}
Gaps: ${JSON.stringify(gapPages.slice(0, 5))}
Target companies: ${companyPages.slice(0, 3).map(p => p.key).join(', ')}
Available hackathons: ${hackathonPages.slice(0, 3).map(p => p.key).join(', ')}

Generate roadmap JSON:
{
  "weeks": [
    {
      "week_range": "Week 1-2",
      "focus_skill": "TypeScript",
      "why": "Required by Razorpay, CRED",
      "daily_time_hours": 2,
      "resources": [
        { "type": "docs", "title": "TypeScript Official Docs", "url": "https://typescriptlang.org/docs" },
        { "type": "video", "title": "TypeScript Full Course - Hitesh Choudhary", "url": "https://youtube.com/..." },
        { "type": "practice", "title": "Build: Convert todo app to TypeScript", "url": null },
        { "type": "blog", "title": "TypeScript for React Devs", "url": "https://ui.dev/typescript" }
      ],
      "hackathon_to_target": "HackRx 6.0 (90% match with current stack — register now)",
      "milestone": "Complete 10 TypeScript exercises on exercism.io"
    }
  ],
  "immediate_hackathons": [
    { "name": "HackRx 6.0", "deadline": "Aug 5", "match": 90, "register_url": "..." }
  ],
  "company_readiness": [
    { "company": "Razorpay", "ready_in": "3 weeks", "after_skill": "TypeScript" }
  ],
  "overall_timeline_weeks": 8
}`
      }]
    });

    const text = response.content[0].text;
    return JSON.parse(text.replace(/```json|```/g, '').trim());
  } catch (error) {
    console.error('[Claude] generateRoadmap error:', error.message);
    return { weeks: [], immediate_hackathons: [], company_readiness: [], overall_timeline_weeks: 8 };
  }
}

module.exports = {
  // ... existing exports
  ingestStep1,
  ingestStep2GeneratePage,
  queryWiki,
  generateRoadmap
};

---

STEP 5 — EXTEND backend/server.js

Add these new endpoints after existing ones.
Do not touch existing endpoints.

const { fetchURL, detectInputType } = require('./fetcher');
const {
  saveWikiPage, getWikiPage, getAllWikiPages,
  updateWikiIndex, appendToLog, getGraphData
} = require('./hydradb');
const {
  ingestStep1, ingestStep2GeneratePage,
  queryWiki, generateRoadmap
} = require('./claude');

// POST /api/ingest
app.post('/api/ingest', async (req, res) => {
  try {
    const { userId, content, type, imageBase64 } = req.body;
    if (!userId || !content) return res.status(400).json({ error: 'userId and content required' });

    const user = await getUser(userId);
    if (!user) return res.status(404).json({ error: 'User not found' });

    let rawContent = content;
    const inputType = type || detectInputType(content);
    console.log(`[Ingest] userId=${userId} type=${inputType}`);

    // Step 0: Fetch content if URL
    if (inputType === 'url') {
      const fetched = await fetchURL(content);
      if (!fetched) return res.status(400).json({ error: 'Could not fetch URL' });
      rawContent = fetched;
    }

    // Step 0b: Screenshot via Claude Vision
    if (inputType === 'screenshot' && imageBase64) {
      const visionResponse = await claude.messages.create({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 1000,
        messages: [{
          role: 'user',
          content: [
            { type: 'image', source: { type: 'base64', media_type: 'image/jpeg', data: imageBase64 } },
            { type: 'text', text: 'Extract all career-relevant text from this image. Include job titles, skills, salary, company name, deadlines, and any other career information visible.' }
          ]
        }]
      });
      rawContent = visionResponse.content[0].text;
    }

    // Get existing wiki summary for context
    const overviewPage = await getWikiPage(userId, 'overview');
    const existingSummary = overviewPage?.content?.slice(0, 500) || '';

    // Step 1: Analysis
    const analysis = await ingestStep1(rawContent, existingSummary, user.stack || []);
    console.log(`[Ingest] Step 1 complete. Pages to create: ${analysis.wiki_pages_to_create?.length}`);

    // Step 2: Generate wiki pages
    const createdPages = [];
    for (const pageInfo of (analysis.wiki_pages_to_create || [])) {
      const pageContent = await ingestStep2GeneratePage(pageInfo, analysis, user.stack || []);
      await saveWikiPage(userId, pageInfo.key, pageContent);
      await updateWikiIndex(userId, pageInfo);
      createdPages.push(pageInfo.key);
    }

    // Update overview
    const overviewContent = `---\ntype: overview\nupdated: ${new Date().toISOString().split('T')[0]}\n---\n\n# Career Overview\n\n${analysis.summary}\n\nLast ingested: ${new Date().toLocaleString()}`;
    await saveWikiPage(userId, 'overview', overviewContent);

    // Log the ingest
    await appendToLog(userId, {
      action: 'ingest',
      detail: `${inputType} source processed. ${createdPages.length} pages created/updated.`
    });

    res.json({
      success: true,
      pages_created: createdPages,
      analysis_summary: analysis.summary,
      companies_found: analysis.companies_found?.length || 0,
      gaps_identified: analysis.gaps_identified?.length || 0,
      hackathons_found: analysis.hackathons_found?.length || 0
    });

  } catch (error) {
    console.error('[/api/ingest] error:', error);
    res.status(500).json({ error: 'Ingest failed', detail: error.message });
  }
});

// GET /api/graph/:userId
app.get('/api/graph/:userId', async (req, res) => {
  try {
    const { userId } = req.params;
    const user = await getUser(userId);
    if (!user) return res.status(404).json({ error: 'User not found' });
    const graphData = await getGraphData(userId);
    res.json(graphData);
  } catch (error) {
    res.status(500).json({ error: 'Graph fetch failed' });
  }
});

// GET /api/wiki/:userId/page
app.get('/api/wiki/:userId/:pageType/:pageName', async (req, res) => {
  try {
    const { userId, pageType, pageName } = req.params;
    const page = await getWikiPage(userId, `${pageType}/${pageName}`);
    if (!page) return res.status(404).json({ error: 'Page not found' });
    res.json(page);
  } catch (error) {
    res.status(500).json({ error: 'Wiki fetch failed' });
  }
});

// POST /api/chat
app.post('/api/chat', async (req, res) => {
  try {
    const { userId, question } = req.body;
    if (!userId || !question) return res.status(400).json({ error: 'userId and question required' });

    const user = await getUser(userId);
    const allPages = await getAllWikiPages(userId);
    
    // Simple relevance: search for question keywords in page content
    const keywords = question.toLowerCase().split(' ').filter(w => w.length > 3);
    const relevantPages = allPages
      .filter(p => keywords.some(kw => (p.content || '').toLowerCase().includes(kw)))
      .slice(0, 10);

    // If no wiki pages yet, fall back to pre-loaded data context
    const pagesForQuery = relevantPages.length > 0 ? relevantPages : allPages.slice(0, 5);

    const userContext = {
      stack: user?.stack || [],
      experience: user?.experience_level || 'beginner',
      goals: user?.goals || []
    };

    const result = await queryWiki(question, pagesForQuery, userContext);
    res.json(result);
  } catch (error) {
    console.error('[/api/chat] error:', error);
    res.status(500).json({ error: 'Chat failed', answer: 'Something went wrong. Please try again.' });
  }
});

// GET /api/roadmap/:userId
app.get('/api/roadmap/:userId', async (req, res) => {
  try {
    const { userId } = req.params;
    const user = await getUser(userId);
    if (!user) return res.status(404).json({ error: 'User not found' });

    const allPages = await getAllWikiPages(userId);
    const gapPages = allPages.filter(p => p.key?.startsWith('gaps/'));
    const companyPages = allPages.filter(p => p.key?.startsWith('companies/'));
    const hackathonPages = allPages.filter(p => p.key?.startsWith('hackathons/'));

    const roadmap = await generateRoadmap(
      user.stack || [],
      gapPages,
      companyPages,
      hackathonPages
    );

    res.json(roadmap);
  } catch (error) {
    console.error('[/api/roadmap] error:', error);
    res.status(500).json({ error: 'Roadmap generation failed' });
  }
});

// GET /api/wiki-pages/:userId
app.get('/api/wiki-pages/:userId', async (req, res) => {
  try {
    const { userId } = req.params;
    const pages = await getAllWikiPages(userId);
    res.json({ pages, count: pages.length });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch wiki pages' });
  }
});

---

STEP 6 — UPDATE frontend/src/App.jsx

Extend existing App.jsx. Keep all existing state and logic.
Add new state below existing state.

// Add to existing state:
const [graphData, setGraphData] = useState(null)
const [selectedNode, setSelectedNode] = useState(null)
const [chatHistory, setChatHistory] = useState([])
const [roadmap, setRoadmap] = useState(null)
const [currentView, setCurrentView] = useState("graph")
const [wikiPages, setWikiPages] = useState([])
const [ingestLoading, setIngestLoading] = useState(false)

// Add fetch functions:
const fetchGraph = async (uid) => {
  try {
    const res = await axios.get(`${API}/api/graph/${uid}`);
    setGraphData(res.data);
  } catch (e) { console.error('graph fetch failed', e); }
};

const fetchWikiPages = async (uid) => {
  try {
    const res = await axios.get(`${API}/api/wiki-pages/${uid}`);
    setWikiPages(res.data.pages || []);
  } catch (e) { console.error('wiki pages fetch failed', e); }
};

// Call after userId is set:
useEffect(() => {
  if (userId) {
    fetchGraph(userId);
    fetchWikiPages(userId);
  }
}, [userId]);

// Layout when view === "dashboard":
// Replace old Dashboard render with:
if (view === "dashboard") {
  return (
    <div className="flex h-screen bg-white overflow-hidden">
      <Sidebar
        wikiPages={wikiPages}
        currentView={currentView}
        setCurrentView={setCurrentView}
        onEntityClick={(nodeId) => setSelectedNode(nodeId)}
      />
      <div className="flex-1 flex flex-col overflow-hidden">
        <Topbar
          currentView={currentView}
          userId={userId}
          userStack={userStack}
        />
        <div className="flex-1 overflow-hidden relative">
          {returnContext?.hasHistory && (
            <MemoryBadge
              message={returnContext.message}
              urgentItems={returnContext.urgentItems}
            />
          )}
          {currentView === "graph" && (
            <CareerGraph
              graphData={graphData}
              userStack={userStack}
              onNodeClick={setSelectedNode}
            />
          )}
          {currentView === "ingest" && (
            <IngestPanel
              userId={userId}
              onIngestComplete={() => {
                fetchGraph(userId);
                fetchWikiPages(userId);
              }}
            />
          )}
          {currentView === "chat" && (
            <ChatInterface
              userId={userId}
              chatHistory={chatHistory}
              setChatHistory={setChatHistory}
            />
          )}
          {currentView === "roadmap" && (
            <RoadmapView
              userId={userId}
              roadmap={roadmap}
              setRoadmap={setRoadmap}
            />
          )}
          {currentView === "journey" && (
            <JourneyView userId={userId} wikiPages={wikiPages} />
          )}
        </div>
      </div>
      {selectedNode && (
        <DetailPanel
          nodeId={selectedNode}
          wikiPages={wikiPages}
          userId={userId}
          onClose={() => setSelectedNode(null)}
        />
      )}
    </div>
  );
}

---

STEP 7 — CREATE frontend/src/components/CareerGraph.jsx

import { useEffect, useRef } from "react";
import Graph from "graphology";
import { circular } from "graphology-layout";
import forceAtlas2 from "graphology-layout-forceatlas2";
import Sigma from "sigma";

const NODE_COLORS = {
  user:      "#F59E0B",  // amber
  companies: "#F97316",  // orange
  skills:    "#3B82F6",  // blue
  gaps:      "#EF4444",  // red
  hackathons:"#8B5CF6",  // purple
  default:   "#94A3B8"   // gray
};

const NODE_SIZES = {
  user: 20,
  companies: 14,
  skills: 10,
  gaps: 9,
  hackathons: 12,
  default: 8
};

export default function CareerGraph({ graphData, userStack, onNodeClick }) {
  const containerRef = useRef(null);
  const sigmaRef = useRef(null);
  const graphRef = useRef(null);

  useEffect(() => {
    if (!containerRef.current || !graphData) return;

    const graph = new Graph();
    graphRef.current = graph;

    // Add user center node
    graph.addNode("user_center", {
      label: "You",
      size: NODE_SIZES.user,
      color: NODE_COLORS.user,
      x: 0, y: 0
    });

    // Add wiki page nodes
    for (const node of (graphData.nodes || [])) {
      if (node.id === "user_center") continue;
      try {
        const parts = node.id.split("/");
        const type = parts[0] || "default";
        graph.addNode(node.id, {
          label: node.label || parts[1] || node.id,
          size: NODE_SIZES[type] || NODE_SIZES.default,
          color: NODE_COLORS[type] || NODE_COLORS.default,
          x: Math.random() * 10 - 5,
          y: Math.random() * 10 - 5,
          nodeType: type
        });
      } catch (e) {}
    }

    // Connect user to all company and hackathon nodes
    for (const node of (graphData.nodes || [])) {
      if (node.id === "user_center") continue;
      const type = node.id.split("/")[0];
      if (type === "companies" || type === "hackathons") {
        try {
          graph.addEdge("user_center", node.id, {
            size: 1.5,
            color: "#CBD5E1"
          });
        } catch (e) {}
      }
    }

    // Add wiki edges
    for (const edge of (graphData.edges || [])) {
      try {
        if (graph.hasNode(edge.from) && graph.hasNode(edge.to) &&
            !graph.hasEdge(edge.from, edge.to)) {
          graph.addEdge(edge.from, edge.to, {
            size: 1,
            color: "#E2E8F0"
          });
        }
      } catch (e) {}
    }

    // Layout
    circular.assign(graph);
    const positions = forceAtlas2(graph, {
      iterations: 100,
      settings: {
        gravity: 1,
        scalingRatio: 2,
        strongGravityMode: true
      }
    });
    forceAtlas2.assign(graph, { iterations: 50 });

    // Init sigma
    if (sigmaRef.current) { sigmaRef.current.kill(); }
    const sigma = new Sigma(graph, containerRef.current, {
      renderEdgeLabels: false,
      defaultEdgeColor: "#CBD5E1",
      defaultNodeColor: "#94A3B8",
    });

    sigma.on("clickNode", ({ node }) => {
      if (onNodeClick) onNodeClick(node);
    });

    sigma.on("enterNode", ({ node }) => {
      const attrs = graph.getNodeAttributes(node);
      containerRef.current.style.cursor = "pointer";
    });

    sigma.on("leaveNode", () => {
      containerRef.current.style.cursor = "default";
    });

    sigmaRef.current = sigma;

    return () => {
      if (sigmaRef.current) { sigmaRef.current.kill(); sigmaRef.current = null; }
    };
  }, [graphData]);

  if (!graphData) return (
    <div className="flex-1 flex items-center justify-center text-gray-400">
      <div className="text-center">
        <div className="text-4xl mb-4">🗺</div>
        <p className="text-lg font-medium text-gray-600">Your career graph is empty</p>
        <p className="text-sm mt-2">Go to Feed to add job descriptions, URLs, or screenshots</p>
      </div>
    </div>
  );

  return (
    <div className="relative w-full h-full">
      <div ref={containerRef} className="w-full h-full" />
      {/* Node legend */}
      <div className="absolute bottom-4 left-4 bg-white border border-gray-200 rounded-lg p-3 shadow-sm text-xs">
        <div className="font-semibold text-gray-600 mb-2 uppercase tracking-wide">Node Types</div>
        {[
          { color: "#F59E0B", label: "You" },
          { color: "#F97316", label: "Companies" },
          { color: "#3B82F6", label: "Skills" },
          { color: "#EF4444", label: "Gaps" },
          { color: "#8B5CF6", label: "Hackathons" }
        ].map(item => (
          <div key={item.label} className="flex items-center gap-2 mb-1">
            <div className="w-3 h-3 rounded-full" style={{ backgroundColor: item.color }} />
            <span className="text-gray-600">{item.label}</span>
          </div>
        ))}
      </div>
      {/* Empty state hint */}
      {graphData.nodes?.length <= 1 && (
        <div className="absolute inset-0 flex items-center justify-center pointer-events-none">
          <div className="text-center text-gray-400">
            <p className="text-sm">Feed career data to build your graph</p>
          </div>
        </div>
      )}
    </div>
  );
}

---

STEP 8 — CREATE frontend/src/components/IngestPanel.jsx

import { useState, useRef } from "react";
import axios from "axios";

const API = import.meta.env.VITE_API_URL || "http://localhost:3001";

export default function IngestPanel({ userId, onIngestComplete }) {
  const [activeTab, setActiveTab] = useState("text");
  const [textContent, setTextContent] = useState("");
  const [urlContent, setUrlContent] = useState("");
  const [processing, setProcessing] = useState(false);
  const [result, setResult] = useState(null);
  const [error, setError] = useState(null);
  const [activityLog, setActivityLog] = useState([]);
  const fileInputRef = useRef(null);

  const addLog = (msg) => setActivityLog(prev => [...prev, { msg, time: new Date().toLocaleTimeString() }]);

  const handleIngest = async () => {
    setProcessing(true);
    setResult(null);
    setError(null);
    setActivityLog([]);

    try {
      let body = { userId };

      if (activeTab === "text") {
        if (!textContent.trim()) { setError("Please paste some content"); setProcessing(false); return; }
        addLog("Reading text content...");
        body.content = textContent;
        body.type = "text";

      } else if (activeTab === "url") {
        if (!urlContent.trim()) { setError("Please enter a URL"); setProcessing(false); return; }
        addLog("Fetching URL content...");
        body.content = urlContent;
        body.type = "url";

      } else if (activeTab === "screenshot") {
        const file = fileInputRef.current?.files?.[0];
        if (!file) { setError("Please select an image"); setProcessing(false); return; }
        addLog("Reading image...");
        const base64 = await new Promise((res) => {
          const reader = new FileReader();
          reader.onload = (e) => res(e.target.result.split(",")[1]);
          reader.readAsDataURL(file);
        });
        body.content = "screenshot";
        body.type = "screenshot";
        body.imageBase64 = base64;
      }

      addLog("Running analysis (Step 1)...");
      addLog("Extracting companies, skills, hackathons...");

      const response = await axios.post(`${API}/api/ingest`, body);

      addLog("Generating wiki pages (Step 2)...");
      addLog(`Created ${response.data.pages_created?.length || 0} wiki pages`);
      addLog("Updating career graph...");
      addLog("Done!");

      setResult(response.data);
      if (onIngestComplete) onIngestComplete();

      // Clear inputs
      setTextContent("");
      setUrlContent("");
      if (fileInputRef.current) fileInputRef.current.value = "";

    } catch (err) {
      setError(err.response?.data?.error || "Processing failed. Please try again.");
      addLog("Error: " + (err.response?.data?.error || err.message));
    } finally {
      setProcessing(false);
    }
  };

  const tabs = ["text", "url", "screenshot"];
  const tabLabels = { text: "Paste Text", url: "Enter URL", screenshot: "Upload Image" };

  return (
    <div className="flex-1 overflow-auto p-6 max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-1">Feed Career Data</h1>
      <p className="text-gray-500 text-sm mb-6">
        Paste job descriptions, LinkedIn URLs, hackathon pages, or screenshots.
        DevRadar extracts career insights and adds them to your personal wiki.
      </p>

      {/* Tab switcher */}
      <div className="flex gap-1 bg-gray-100 rounded-lg p-1 mb-4 w-fit">
        {tabs.map(tab => (
          <button
            key={tab}
            onClick={() => setActiveTab(tab)}
            className={`px-4 py-2 rounded-md text-sm font-medium transition-all ${
              activeTab === tab
                ? "bg-white shadow-sm text-gray-900"
                : "text-gray-500 hover:text-gray-700"
            }`}
          >
            {tabLabels[tab]}
          </button>
        ))}
      </div>

      {/* Text tab */}
      {activeTab === "text" && (
        <div>
          <label className="text-sm font-medium text-gray-700 block mb-2">
            Paste job description, notes, or any career text
          </label>
          <textarea
            value={textContent}
            onChange={e => setTextContent(e.target.value)}
            placeholder="Senior Frontend Engineer at Razorpay&#10;Requirements: React, TypeScript, GraphQL&#10;Salary: 18-28 LPA&#10;Location: Bangalore..."
            className="w-full h-48 border border-gray-200 rounded-lg p-3 text-sm text-gray-900 focus:outline-none focus:border-blue-400 resize-none"
          />
        </div>
      )}

      {/* URL tab */}
      {activeTab === "url" && (
        <div>
          <label className="text-sm font-medium text-gray-700 block mb-2">
            Enter a URL (LinkedIn job, Devfolio hackathon, company career page)
          </label>
          <input
            type="url"
            value={urlContent}
            onChange={e => setUrlContent(e.target.value)}
            placeholder="https://linkedin.com/jobs/view/..."
            className="w-full border border-gray-200 rounded-lg p-3 text-sm text-gray-900 focus:outline-none focus:border-blue-400"
          />
          <div className="mt-3 flex gap-2 flex-wrap">
            {["linkedin.com/jobs", "devfolio.co", "unstop.com", "hackerearth.com"].map(hint => (
              <span key={hint} className="text-xs bg-gray-100 text-gray-500 px-2 py-1 rounded">
                {hint}
              </span>
            ))}
          </div>
        </div>
      )}

      {/* Screenshot tab */}
      {activeTab === "screenshot" && (
        <div>
          <label className="text-sm font-medium text-gray-700 block mb-2">
            Upload a screenshot (job post, LinkedIn, tweet, anything)
          </label>
          <div
            className="border-2 border-dashed border-gray-200 rounded-lg p-8 text-center cursor-pointer hover:border-blue-300 transition-colors"
            onClick={() => fileInputRef.current?.click()}
          >
            <div className="text-4xl mb-2">📷</div>
            <p className="text-sm text-gray-500">Click to select image or drag and drop</p>
            <p className="text-xs text-gray-400 mt-1">PNG, JPG, JPEG supported</p>
          </div>
          <input
            ref={fileInputRef}
            type="file"
            accept="image/*"
            className="hidden"
          />
        </div>
      )}

      {error && (
        <div className="mt-3 p-3 bg-red-50 border border-red-200 rounded-lg text-sm text-red-600">
          {error}
        </div>
      )}

      {/* Process button */}
      <button
        onClick={handleIngest}
        disabled={processing}
        className="mt-4 w-full bg-blue-500 hover:bg-blue-600 disabled:bg-blue-300 text-white font-medium py-3 rounded-lg text-sm transition-colors"
      >
        {processing ? "Processing..." : "Process & Add to Career Wiki"}
      </button>

      {/* Activity log */}
      {activityLog.length > 0 && (
        <div className="mt-4 bg-gray-50 rounded-lg p-3 border border-gray-100">
          <div className="text-xs font-medium text-gray-500 mb-2 uppercase tracking-wide">Activity</div>
          {activityLog.map((log, i) => (
            <div key={i} className="flex gap-2 text-xs text-gray-600 mb-1">
              <span className="text-gray-400">{log.time}</span>
              <span>{log.msg}</span>
            </div>
          ))}
        </div>
      )}

      {/* Result */}
      {result && (
        <div className="mt-4 bg-green-50 border border-green-200 rounded-lg p-4">
          <div className="font-medium text-green-800 mb-2">Successfully processed</div>
          <div className="text-sm text-green-700">{result.analysis_summary}</div>
          <div className="flex gap-4 mt-3 text-xs text-green-600">
            <span>{result.companies_found} companies found</span>
            <span>{result.gaps_identified} gaps identified</span>
            <span>{result.hackathons_found} hackathons found</span>
          </div>
          <div className="mt-2 text-xs text-green-600">
            Wiki pages created: {result.pages_created?.join(", ") || "none"}
          </div>
        </div>
      )}
    </div>
  );
}

---

STEP 9 — CREATE frontend/src/components/ChatInterface.jsx

import { useState, useRef, useEffect } from "react";
import axios from "axios";

const API = import.meta.env.VITE_API_URL || "http://localhost:3001";

export default function ChatInterface({ userId, chatHistory, setChatHistory }) {
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const bottomRef = useRef(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [chatHistory]);

  const sendMessage = async () => {
    if (!input.trim() || loading) return;
    const question = input.trim();
    setInput("");
    setChatHistory(prev => [...prev, { role: "user", content: question }]);
    setLoading(true);

    try {
      const res = await axios.post(`${API}/api/chat`, { userId, question });
      setChatHistory(prev => [...prev, {
        role: "assistant",
        content: res.data.answer,
        citations: res.data.citations || []
      }]);
    } catch (err) {
      setChatHistory(prev => [...prev, {
        role: "assistant",
        content: "Something went wrong. Please try again.",
        citations: []
      }]);
    } finally {
      setLoading(false);
    }
  };

  const suggestions = [
    "Which hackathon should I register for right now?",
    "What should I learn this week?",
    "Am I ready for Razorpay?",
    "Compare Groww vs Razorpay for my profile"
  ];

  return (
    <div className="flex flex-col h-full">
      {/* Messages */}
      <div className="flex-1 overflow-auto p-4 space-y-4">
        {chatHistory.length === 0 && (
          <div className="text-center mt-16">
            <div className="text-4xl mb-4">💬</div>
            <h2 className="text-lg font-semibold text-gray-700 mb-2">Ask anything about your career</h2>
            <p className="text-sm text-gray-400 mb-6">DevRadar answers from your personal career wiki</p>
            <div className="flex flex-col gap-2 items-center">
              {suggestions.map(s => (
                <button
                  key={s}
                  onClick={() => setInput(s)}
                  className="text-sm text-blue-500 border border-blue-200 rounded-lg px-4 py-2 hover:bg-blue-50 transition-colors"
                >
                  {s}
                </button>
              ))}
            </div>
          </div>
        )}
        {chatHistory.map((msg, i) => (
          <div key={i} className={`flex ${msg.role === "user" ? "justify-end" : "justify-start"}`}>
            <div className={`max-w-2xl ${msg.role === "user"
              ? "bg-blue-500 text-white rounded-2xl rounded-br-sm px-4 py-3"
              : "bg-white border border-gray-100 shadow-sm rounded-2xl rounded-bl-sm px-4 py-3"
            }`}>
              <p className="text-sm whitespace-pre-wrap">{msg.content}</p>
              {msg.citations?.length > 0 && (
                <div className="mt-3 pt-3 border-t border-gray-100">
                  <p className="text-xs text-gray-400 mb-1">Sources from your career wiki:</p>
                  <div className="flex flex-wrap gap-1">
                    {msg.citations.filter(c => msg.content.includes(`[${c.index}]`)).map(c => (
                      <span key={c.index} className="text-xs bg-gray-50 border border-gray-200 rounded px-2 py-0.5">
                        [{c.index}] {c.key}
                      </span>
                    ))}
                  </div>
                </div>
              )}
            </div>
          </div>
        ))}
        {loading && (
          <div className="flex justify-start">
            <div className="bg-white border border-gray-100 shadow-sm rounded-2xl px-4 py-3">
              <div className="flex gap-1">
                <div className="w-2 h-2 bg-gray-300 rounded-full animate-bounce" />
                <div className="w-2 h-2 bg-gray-300 rounded-full animate-bounce" style={{ animationDelay: "0.1s" }} />
                <div className="w-2 h-2 bg-gray-300 rounded-full animate-bounce" style={{ animationDelay: "0.2s" }} />
              </div>
            </div>
          </div>
        )}
        <div ref={bottomRef} />
      </div>

      {/* Input */}
      <div className="border-t border-gray-100 p-4">
        <div className="flex gap-2">
          <input
            value={input}
            onChange={e => setInput(e.target.value)}
            onKeyDown={e => e.key === "Enter" && !e.shiftKey && sendMessage()}
            placeholder="Ask about your career, companies, skills..."
            className="flex-1 border border-gray-200 rounded-lg px-4 py-2.5 text-sm focus:outline-none focus:border-blue-400"
          />
          <button
            onClick={sendMessage}
            disabled={loading || !input.trim()}
            className="bg-blue-500 hover:bg-blue-600 disabled:bg-blue-300 text-white px-4 py-2.5 rounded-lg text-sm font-medium transition-colors"
          >
            Send
          </button>
        </div>
      </div>
    </div>
  );
}

---

STEP 10 — CREATE frontend/src/components/RoadmapView.jsx

import { useState, useEffect } from "react";
import axios from "axios";

const API = import.meta.env.VITE_API_URL || "http://localhost:3001";

export default function RoadmapView({ userId, roadmap, setRoadmap }) {
  const [loading, setLoading] = useState(false);

  const fetchRoadmap = async () => {
    if (roadmap) return;
    setLoading(true);
    try {
      const res = await axios.get(`${API}/api/roadmap/${userId}`);
      setRoadmap(res.data);
    } catch (e) { console.error(e); }
    finally { setLoading(false); }
  };

  useEffect(() => { fetchRoadmap(); }, [userId]);

  const RESOURCE_ICONS = { docs: "📖", video: "🎥", practice: "🛠", blog: "📝" };

  if (loading) return (
    <div className="flex-1 flex items-center justify-center text-gray-400">
      <p>Generating your personalized roadmap...</p>
    </div>
  );

  if (!roadmap) return (
    <div className="flex-1 flex items-center justify-center">
      <div className="text-center">
        <div className="text-4xl mb-4">🗺</div>
        <p className="text-gray-500">Feed career data first to generate your roadmap</p>
        <button onClick={fetchRoadmap} className="mt-4 text-sm text-blue-500 hover:underline">
          Generate Roadmap
        </button>
      </div>
    </div>
  );

  return (
    <div className="flex-1 overflow-auto p-6 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-1">Your Career Roadmap</h1>
      <p className="text-sm text-gray-400 mb-2">
        Powered by Claude AI · Based on your career wiki
      </p>
      <p className="text-sm text-gray-500 mb-6">Total timeline: ~{roadmap.overall_timeline_weeks} weeks</p>

      {/* Immediate hackathons */}
      {roadmap.immediate_hackathons?.length > 0 && (
        <div className="bg-orange-50 border border-orange-200 rounded-xl p-4 mb-6">
          <h2 className="text-sm font-semibold text-orange-800 mb-3">Register Now — Current Stack Match</h2>
          <div className="space-y-2">
            {roadmap.immediate_hackathons.map((h, i) => (
              <div key={i} className="flex items-center justify-between">
                <div>
                  <span className="font-medium text-orange-900 text-sm">{h.name}</span>
                  <span className="text-orange-600 text-xs ml-2">Deadline: {h.deadline}</span>
                </div>
                <div className="flex items-center gap-2">
                  <span className="text-xs bg-orange-100 text-orange-700 px-2 py-0.5 rounded">{h.match}% match</span>
                  {h.register_url && (
                    <a href={h.register_url} target="_blank" rel="noopener noreferrer"
                       className="text-xs text-blue-500 hover:underline">Register →</a>
                  )}
                </div>
              </div>
            ))}
          </div>
        </div>
      )}

      {/* Weekly plan */}
      <div className="space-y-6">
        {roadmap.weeks?.map((week, i) => (
          <div key={i} className="border border-gray-100 rounded-xl overflow-hidden">
            <div className="bg-gray-50 px-4 py-3 flex items-center justify-between">
              <div>
                <span className="font-semibold text-gray-800">{week.week_range}</span>
                <span className="ml-2 text-sm text-gray-500">— {week.focus_skill}</span>
              </div>
              <span className="text-xs text-gray-400">{week.daily_time_hours}h/day</span>
            </div>
            <div className="p-4">
              <p className="text-xs text-gray-500 mb-3">{week.why}</p>
              <div className="space-y-2 mb-3">
                {week.resources?.map((r, j) => (
                  <div key={j} className="flex items-start gap-2">
                    <span>{RESOURCE_ICONS[r.type] || "📌"}</span>
                    <div>
                      {r.url ? (
                        <a href={r.url} target="_blank" rel="noopener noreferrer"
                           className="text-sm text-blue-500 hover:underline">{r.title}</a>
                      ) : (
                        <span className="text-sm text-gray-700">{r.title}</span>
                      )}
                    </div>
                  </div>
                ))}
              </div>
              {week.hackathon_to_target && (
                <div className="bg-purple-50 rounded-lg px-3 py-2 text-xs text-purple-700">
                  🏆 {week.hackathon_to_target}
                </div>
              )}
              {week.milestone && (
                <div className="mt-2 text-xs text-gray-400 border-t pt-2">
                  Milestone: {week.milestone}
                </div>
              )}
            </div>
          </div>
        ))}
      </div>

      {/* Company readiness */}
      {roadmap.company_readiness?.length > 0 && (
        <div className="mt-6 border border-gray-100 rounded-xl p-4">
          <h2 className="font-semibold text-gray-800 mb-3">Company Readiness</h2>
          <div className="space-y-2">
            {roadmap.company_readiness.map((c, i) => (
              <div key={i} className="flex justify-between text-sm">
                <span className="text-gray-700">{c.company}</span>
                <span className="text-gray-400">{c.ready_in} — after {c.after_skill}</span>
              </div>
            ))}
          </div>
        </div>
      )}

      <button
        onClick={() => { setRoadmap(null); fetchRoadmap(); }}
        className="mt-4 text-xs text-gray-400 hover:text-gray-600"
      >
        Regenerate roadmap
      </button>
    </div>
  );
}

---

STEP 11 — CREATE frontend/src/components/Sidebar.jsx

import { useState } from "react";

const VIEW_ITEMS = [
  { id: "graph",   label: "Career Graph",  icon: "🗺" },
  { id: "ingest",  label: "Feed Data",     icon: "➕" },
  { id: "chat",    label: "Ask Anything",  icon: "💬" },
  { id: "roadmap", label: "Roadmap",       icon: "📍" },
  { id: "journey", label: "My Journey",    icon: "⏱" }
];

export default function Sidebar({ wikiPages, currentView, setCurrentView, onEntityClick }) {
  const [expandedSections, setExpandedSections] = useState({
    companies: true, skills: true, hackathons: false, gaps: false
  });

  const companies = wikiPages.filter(p => p.key?.startsWith("companies/"));
  const skills = wikiPages.filter(p => p.key?.startsWith("skills/"));
  const hackathons = wikiPages.filter(p => p.key?.startsWith("hackathons/"));
  const gaps = wikiPages.filter(p => p.key?.startsWith("gaps/"));

  const toggle = (section) => setExpandedSections(prev => ({...prev, [section]: !prev[section]}));

  return (
    <div className="w-60 bg-white border-r border-gray-100 flex flex-col h-screen flex-shrink-0">
      {/* Logo */}
      <div className="px-4 py-4 border-b border-gray-100">
        <div className="font-bold text-gray-900 text-base">DevRadar</div>
        <div className="text-xs text-blue-500 mt-0.5">WikiThon 2025</div>
      </div>

      {/* Nav items */}
      <div className="px-2 py-2 border-b border-gray-100">
        {VIEW_ITEMS.map(item => (
          <button
            key={item.id}
            onClick={() => setCurrentView(item.id)}
            className={`w-full flex items-center gap-2.5 px-3 py-2 rounded-md text-sm text-left transition-colors mb-0.5 ${
              currentView === item.id
                ? "bg-blue-50 text-blue-600 font-medium"
                : "text-gray-600 hover:bg-gray-50"
            }`}
          >
            <span>{item.icon}</span>
            <span>{item.label}</span>
          </button>
        ))}
      </div>

      {/* Wiki entities */}
      <div className="flex-1 overflow-auto px-2 py-2">
        <div className="text-xs font-medium text-gray-400 uppercase tracking-wider px-3 mb-2">
          Career Wiki
        </div>

        {[
          { key: "companies", label: "Companies", items: companies, color: "text-orange-500" },
          { key: "skills", label: "Skills", items: skills, color: "text-blue-500" },
          { key: "hackathons", label: "Hackathons", items: hackathons, color: "text-purple-500" },
          { key: "gaps", label: "Gaps", items: gaps, color: "text-red-500" }
        ].map(section => (
          <div key={section.key} className="mb-1">
            <button
              onClick={() => toggle(section.key)}
              className="w-full flex items-center justify-between px-3 py-1.5 text-xs text-gray-500 hover:bg-gray-50 rounded-md"
            >
              <span>{expandedSections[section.key] ? "▾" : "▸"} {section.label}</span>
              {section.items.length > 0 && (
                <span className="bg-gray-100 text-gray-500 rounded px-1.5 text-xs">
                  {section.items.length}
                </span>
              )}
            </button>
            {expandedSections[section.key] && section.items.map(page => (
              <button
                key={page.key}
                onClick={() => { onEntityClick(page.key); setCurrentView("graph"); }}
                className="w-full text-left px-6 py-1 text-xs text-gray-600 hover:bg-gray-50 rounded-md truncate"
              >
                {page.key.split("/")[1]?.replace(/-/g, " ")}
              </button>
            ))}
            {expandedSections[section.key] && section.items.length === 0 && (
              <div className="px-6 py-1 text-xs text-gray-300 italic">
                Feed data to populate
              </div>
            )}
          </div>
        ))}
      </div>

      {/* Bottom status */}
      <div className="px-4 py-3 border-t border-gray-100">
        <div className="text-xs text-gray-400">
          {wikiPages.length} wiki pages
        </div>
        <div className="flex items-center gap-1.5 mt-1">
          <div className="w-1.5 h-1.5 bg-green-400 rounded-full animate-pulse" />
          <span className="text-xs text-green-600">HydraDB active</span>
        </div>
      </div>
    </div>
  );
}

---

STEP 12 — CREATE frontend/src/components/DetailPanel.jsx

import { useState } from "react";
import axios from "axios";

const API = import.meta.env.VITE_API_URL || "http://localhost:3001";

export default function DetailPanel({ nodeId, wikiPages, userId, onClose }) {
  if (!nodeId) return null;

  const page = wikiPages.find(p => p.key === nodeId);
  const parts = nodeId.split("/");
  const type = parts[0];
  const name = parts[1]?.replace(/-/g, " ") || nodeId;

  const typeColors = {
    companies: "bg-orange-50 text-orange-700 border-orange-200",
    skills: "bg-blue-50 text-blue-700 border-blue-200",
    gaps: "bg-red-50 text-red-700 border-red-200",
    hackathons: "bg-purple-50 text-purple-700 border-purple-200",
    user: "bg-amber-50 text-amber-700 border-amber-200"
  };

  return (
    <div className="w-80 bg-white border-l border-gray-100 shadow-lg flex flex-col h-screen flex-shrink-0">
      {/* Header */}
      <div className="flex items-center justify-between px-4 py-3 border-b border-gray-100">
        <div className="flex items-center gap-2">
          <span className={`text-xs font-medium px-2 py-0.5 rounded border capitalize ${typeColors[type] || typeColors.skills}`}>
            {type}
          </span>
          <span className="text-sm font-semibold text-gray-900 capitalize">{name}</span>
        </div>
        <button onClick={onClose} className="text-gray-400 hover:text-gray-600 text-lg leading-none">×</button>
      </div>

      {/* Content */}
      <div className="flex-1 overflow-auto p-4">
        {page ? (
          <div>
            <div className="text-xs text-gray-400 mb-3">From your career wiki · {page.updated_at?.split("T")[0]}</div>
            <div className="prose prose-sm max-w-none">
              <pre className="whitespace-pre-wrap text-xs text-gray-600 font-sans leading-relaxed">
                {page.content?.replace(/---[\s\S]*?---/, "").trim()}
              </pre>
            </div>
          </div>
        ) : (
          <div className="text-center mt-8 text-gray-400">
            <div className="text-3xl mb-2">{nodeId === "user_center" ? "👤" : "📄"}</div>
            {nodeId === "user_center" ? (
              <p className="text-sm">This is you — the center of your career graph.</p>
            ) : (
              <p className="text-sm">No wiki page yet for this node.<br/>Feed data containing "{name}" to generate one.</p>
            )}
          </div>
        )}
      </div>
    </div>
  );
}

---

STEP 13 — CREATE frontend/src/components/JourneyView.jsx

export default function JourneyView({ userId, wikiPages }) {
  const events = wikiPages
    .filter(p => p.updated_at)
    .sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at))
    .map(p => ({
      key: p.key,
      label: p.key.replace(/\//g, " › ").replace(/-/g, " "),
      time: new Date(p.updated_at).toLocaleString(),
      type: p.key.split("/")[0]
    }));

  const dotColors = {
    companies: "bg-orange-400",
    skills: "bg-blue-400",
    gaps: "bg-red-400",
    hackathons: "bg-purple-400",
    overview: "bg-gray-400",
    index: "bg-gray-300",
    log: "bg-gray-300"
  };

  return (
    <div className="flex-1 overflow-auto p-6 max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-1">My Journey</h1>
      <p className="text-sm text-gray-400 mb-6">Everything DevRadar has learned about your career</p>

      <div className="grid grid-cols-2 gap-3 mb-6">
        {[
          { label: "Wiki Pages", value: wikiPages.length },
          { label: "Companies", value: wikiPages.filter(p => p.key?.startsWith("companies/")).length },
          { label: "Skills Tracked", value: wikiPages.filter(p => p.key?.startsWith("skills/")).length },
          { label: "Gaps Found", value: wikiPages.filter(p => p.key?.startsWith("gaps/")).length }
        ].map(stat => (
          <div key={stat.label} className="bg-gray-50 rounded-xl p-3 border border-gray-100">
            <div className="text-2xl font-bold text-gray-800">{stat.value}</div>
            <div className="text-xs text-gray-400">{stat.label}</div>
          </div>
        ))}
      </div>

      {events.length === 0 ? (
        <div className="text-center py-16 text-gray-400">
          <p>No journey data yet.</p>
          <p className="text-sm mt-1">Feed career data to build your timeline.</p>
        </div>
      ) : (
        <div className="relative">
          <div className="absolute left-3 top-0 bottom-0 w-0.5 bg-gray-100" />
          <div className="space-y-4">
            {events.map((event, i) => (
              <div key={i} className="flex gap-4 relative">
                <div className={`w-2.5 h-2.5 rounded-full mt-1.5 flex-shrink-0 z-10 ${dotColors[event.type] || "bg-gray-300"}`} />
                <div>
                  <div className="text-sm text-gray-700 capitalize font-medium">{event.label}</div>
                  <div className="text-xs text-gray-400">{event.time}</div>
                </div>
              </div>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}

---

STEP 14 — CREATE frontend/src/components/Topbar.jsx

export default function Topbar({ currentView, userId, userStack }) {
  const VIEW_TITLES = {
    graph: "Career Graph",
    ingest: "Feed Career Data",
    chat: "Ask Anything",
    roadmap: "Your Roadmap",
    journey: "My Journey"
  };

  return (
    <div className="h-12 bg-white border-b border-gray-100 flex items-center justify-between px-4 flex-shrink-0">
      <span className="font-semibold text-gray-800 text-sm">{VIEW_TITLES[currentView] || currentView}</span>
      <div className="flex gap-1.5 items-center">
        {(userStack || []).slice(0, 4).map(skill => (
          <span key={skill} className="text-xs bg-blue-50 text-blue-600 border border-blue-100 px-2 py-0.5 rounded-full">
            {skill}
          </span>
        ))}
        {(userStack || []).length > 4 && (
          <span className="text-xs text-gray-400">+{userStack.length - 4}</span>
        )}
      </div>
    </div>
  );
}

---

STEP 15 — UPDATE StackInput.jsx FOR LIGHT THEME

Keep all existing logic. Only update colors:
  bg-gray-900 → bg-white or bg-gray-50
  text-white → text-gray-900
  border-gray-700 → border-gray-200
  bg-teal-500 → bg-blue-500
  Dark pills → Light pills with border

---

STEP 16 — UPDATE MemoryBadge.jsx FOR LIGHT THEME

Keep all existing logic. Only update colors:
  bg-dark-teal → bg-amber-50
  border-teal → border-amber-200
  text-white → text-amber-900
  Urgent pills: bg-amber-100 text-amber-700

---

FINAL CHECKS

1. All existing endpoints still return the same responses
2. Seed demo still works: node backend/seed-demo.js
3. Frontend builds: cd frontend && npm run build
4. No console errors in browser
5. Graph loads when userId exists
6. IngestPanel shows activity log during processing
7. Chat returns answers with citations
8. Roadmap generates after feeding some data

---

GIT COMMIT PATTERN

After each step commit with lowercase message:
  "add fetcher.js for url and screenshot processing"
  "extend hydradb with wiki page storage functions"
  "add ingest endpoints to server"
  "add career graph with sigma.js"
  "add ingest panel three tab ui"
  "add chat interface with citations"
  "add roadmap view with resources"
  "add sidebar with wiki entity navigation"
  "add detail panel for node click"
  "add journey view timeline"
  "update stack input to light theme"
  "update memory badge to light theme"
  "connect all views in app jsx"
  "fix mobile layout"
  "ship it"
```
