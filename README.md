Let me fetch both links in parallel — the hackathon page and the GitHub repository.

Good. Now let me dig deeper into the repository code and structure.

I now have enough to write a thorough analysis. Let me synthesize everything.

---

# Comprehensive Analysis: Hackathon × Orbit-UI

---

## Part 1: Hackathon — "Agents Under Pressure: Build Your Own OS"

### Core Problem Statement

The hackathon's central thesis is: *most AI agents fail outside curated demos*. You must build systems that survive real-world chaos — not just happy paths.

### Evaluation Tracks (The Judging Rubric)

This is the most important thing to internalize. Everything is scored against four axes:

| Track | What It Means Concretely |
|-------|--------------------------|
| **Memory** | Does the agent retain and retrieve context across steps, runs, and sessions? Can it recall earlier decisions, avoid repeating mistakes? |
| **Tools** | Does the agent effectively use external APIs, browser, file system, code execution? Does it compose tools correctly? |
| **Recovery** | When a tool fails, a page changes, or an API errors out — does the agent detect, adapt, and retry rather than crash? |
**Adaptation** | When task parameters shift mid-run (e.g., a form changes, a page 404s), does the agent dynamically adjust its strategy? |

### Suggested Build Categories

The event explicitly names **"Custom OS implementations" (inspired by Google I/O)** as the first suggestion — this is the one most judges will be receptive to given the theme. Other valid tracks: AI research agents, workflow automation, long-context copilots, memory-based assistants, error-resilient systems.

### Constraints & Format

- 48-hour virtual hackathon on Discord
- No explicit mandatory technology stack — you can use any LLM provider
- Registration requires wallet token ownership (HydraDB Discord community)
- $500 prize pool split across 1st, 2nd, and special bounties

---

## Part 2: Deep Dive — Orbit-UI Repository

### What It Is

Orbit is a **self-hosted, visual workflow builder for desktop AI agents** that runs inside a Docker container with a full virtual desktop environment. Think n8n or Zapier, but instead of API webhooks, the agent physically operates a real browser and desktop via Playwright + noVNC.

The tagline "running inside its own Virtual Computer" is what makes this hackathon-aligned — it literally is a mini OS for AI agents.

### Architecture

```
User Browser (port 3000)
    │
    ▼
Frontend [React + Vite + ReactFlow]
    │  REST + SSE (port 8000)
    ▼
Backend [FastAPI]
    ├── SQLite (workflows, runs, secrets tables)
    ├── Code Generator (codegen.py) — JSON graph → executable Python
    ├── State Manager (state.py) — pub-sub SSE, pause/resume events
    └── Scheduler — croniter-based, per-minute polling
    │
    ▼
VM Container [Docker]
    ├── Playwright + Chrome (port 6080 noVNC, DISPLAY=:99)
    ├── OculoOS Daemon (port 7878)
    └── /workspace volume (shared file system)
```

### How Execution Works (The Core Loop)

1. User builds a visual graph in the React UI (nodes connected by edges)
2. On "Run", backend serializes the graph and passes it to `codegen.py`
3. Codegen does topological sort, detects retry loops, and **generates a Python async script** from the graph
4. The generated script is dynamically imported and executed
5. State updates are streamed to the UI via SSE in real time
6. User can pause (human-in-the-loop), resume, or stop at any time

### Supported Node Types

| Node | Function |
|------|----------|
| `Navigate` | Browser navigation to URL |
| `Do` | Freeform AI action on screen |
| `Check` | Conditional branching (true/false edges) |
| `Fill` | Form filling with credential vault support |
| `Read` | Structured data extraction with Pydantic schema |
| `Code` | Inline Python execution with access to prior outputs |
| `Agent` | Custom agent class inheriting `BaseActionAgent` |
| `ForEach` | Iteration over a list |
| `Bootstrap` | Initialization step |

### Tech Stack

- **Backend**: Python, FastAPI, SQLite, croniter, LiteLLM-compatible (Gemini default, also OpenAI/Anthropic/OpenRouter)
- **Frontend**: React, Vite, Nginx, ReactFlow (visual graph), Monaco Editor
- **Infrastructure**: Docker Compose, noVNC, Playwright, OculoOS daemon
- **AI**: Default `gemini-2.5-flash-preview`, per-node LLM override supported; MCP (Model Context Protocol) server integration via `MCPToolset`

### Key Strengths

1. **Visual workflow builder already exists** — the hardest UI work is done
2. **Code generation is clever** — graph → Python is a powerful primitive that keeps workflows inspectable and debuggable
3. **Real browser execution** — not a simulated environment, gives demos genuine "wow factor"
4. **MCP support** — can connect to external tool servers (file system, APIs, databases)
5. **Human-in-the-loop** — pause/resume is already built
6. **Cron scheduling** — agents can be triggered on a schedule
7. **SSE streaming** — real-time node status in the UI
8. **Secrets vault** — credentials never exposed to LLM context

### Key Weaknesses

1. **No cross-run memory** — each workflow execution starts completely fresh; the agent cannot recall previous runs, past failures, or accumulated knowledge
2. **No self-healing/recovery** — when a node fails, execution stops; there's no automatic retry with a different strategy
3. **Static workflows** — once a workflow is defined, its structure is fixed; no runtime adaptation of the graph based on what the agent discovers
4. **Single-workflow concurrency** — only one workflow runs at a time
5. **Gemini-centric** — while LiteLLM-compatible, defaults are Gemini-specific, which may limit flexibility
6. **No agent-to-agent communication** — workflows are isolated; no orchestrator/subagent patterns

---

## Part 3: Mapping Against Hackathon Requirements

### Track-by-Track Gap Analysis

#### Track 1: Memory — **Critical Gap**

| Hackathon Expectation | Current State | Gap |
|-----------------------|--------------|-----|
| Context retention across steps | Partial — `{{node_id.field}}` template vars pass outputs between nodes within a run | Within-run only |
| Cross-run memory | None — each run is independent | Full gap |
| Long-term knowledge retrieval | None | Full gap |
| Avoiding past mistakes | None | Full gap |

**What's needed**: A memory layer (vector store or structured SQLite) that persists observations, outcomes, and learnings across runs. The agent should be able to query "what happened the last 3 times I tried to log into this site?" before attempting.

#### Track 2: Tools — **Existing Strength, Needs Demo**

Orbit already wins here. It has browser automation, form filling, code execution, MCP integration, file management, and API webhooks. The main work is making tool failures *visible and recoverable* rather than fatal.

#### Track 3: Recovery — **Partial, Needs Systematic Work**

| Hackathon Expectation | Current State | Gap |
|-----------------------|--------------|-----|
| Retry mechanisms | Loop nodes exist, but manual | Needs auto-detect |
| Error propagation control | Run stops on failure | Needs graceful degradation |
| Failure detection | SSE streams errors to UI | No automatic agent response |
| Fallback strategies | None | Full gap |

**What's needed**: A `RecoveryAgent` node type that intercepts errors, analyzes the failure (screenshot + error message), and either retries with a modified approach or routes to a fallback branch.

#### Track 4: Adaptation — **Full Gap**

Currently workflows are static DAGs. The agent cannot add nodes, reroute edges, or change strategy mid-run. This is the hardest track to satisfy but also potentially the most impressive.

**What's needed**: A "meta-agent" node that can inspect the current workflow state, decide to spawn sub-workflows or modify the execution plan, and route accordingly.

---

## Part 4: Recommendations — What to Build

### Strategic Recommendation

**Build directly on top of Orbit-UI, not a wrapper.** The existing codebase gives you:
- The hardest parts already done (visual graph, code generation, browser execution, Docker infra)
- A compelling demo surface (visual workflow execution with real browser)
- A narrative that exactly matches "Build your own OS"

Adding a wrapper layer would duplicate effort without benefit. The goal is to extend the existing node types and state management.

### Architecture Changes Required

```
Current: Static Graph → Codegen → Execute → Done
Target:  Static Graph → Codegen → Execute ↔ Memory Layer
                                           ↕
                                    Recovery Agent
                                           ↕
                                  Adaptation Planner
```

### Implementation Roadmap

#### Priority 1 — Memory Layer (High Impact, Medium Effort) [Day 1, Hours 0-8]

Add a persistent memory store that all agent nodes can read from and write to.

**Minimal implementation**:
```python
# backend/memory.py
# SQLite table: memories(id, workflow_id, run_id, key, value, embedding, timestamp)
# Functions: remember(key, value), recall(query, top_k=5), forget(key)
```

- Add a `Memory` node type to codegen that calls `remember()` / `recall()`
- Pass `memory_context` to all `Agent` and `Do` nodes as part of their system prompt
- Store: task outcomes, error messages, screenshots of failure states, successful approaches

**Why SQLite over vector DB**: Lower complexity, no extra dependency, and for a 48-hour hackathon you can do semantic search via embeddings stored as BLOB or use keyword matching — good enough for the demo.

#### Priority 2 — Recovery Agent Node (High Impact, Medium Effort) [Day 1, Hours 8-16]

Add a special node type that wraps other nodes with error catching.

**Approach**:
```python
# New node verb: "Recover"
# codegen generates:
try:
    await original_node_action()
except Exception as e:
    screenshot = await take_screenshot()
    recovery_plan = await llm.generate(
        f"Task failed: {e}\nScreenshot: {screenshot}\nPast attempts: {memory.recall('failures')}\n"
        "Suggest 3 alternative approaches."
    )
    # Execute recovery plan as a sub-workflow
    memory.remember("failures", str(e))
```

- Wire this into the graph as an optional "recovery edge" on any node
- The UI already supports `conditional_true`/`conditional_false` edges — add a `on_error` edge type

#### Priority 3 — Adaptive Re-planning Node (Medium Impact, High Effort) [Day 1, Hours 16-24]

A "Planner" node that can dynamically construct a sub-workflow at runtime.

**Approach**: Rather than modifying the live graph (hard), have the Planner node generate a *temporary workflow JSON* and trigger it via the existing `/start` endpoint. This re-uses the entire existing codegen pipeline for adaptive behavior.

```python
# "Plan" node: 
# 1. LLM is given current task, observations, and available node types
# 2. LLM outputs a workflow JSON
# 3. Backend creates+runs the sub-workflow
# 4. Results flow back to parent workflow via memory layer
```

#### Priority 4 — Demo Polish [Day 2, Hours 0-8]

- Add a "Mission Control" dashboard showing: current run, memory contents, past run outcomes, active recovery attempts
- Create a compelling demo workflow that exercises all 4 tracks: e.g., "Research competitor pricing — retry if rate-limited, remember findings across runs, adapt if site layout changes"
- Record a 2-minute demo video showing the agent failing gracefully and recovering

---

## Part 5: MVP vs. Stretch Goals

### MVP for Submission (Achievable in 48 hours)

1. **Memory node** — persist + recall across runs (SQLite, keyword search)
2. **Error recovery** — catch failures, query memory for past solutions, retry with LLM-generated alternative approach
3. **Demo workflow** — e.g., "Monitor a website for price changes, retry on failure, remember history, alert via webhook"
4. **Memory panel in UI** — show the agent's accumulated knowledge in the existing frontend

This directly hits all 4 judging tracks with concrete, demonstrable behavior.

### Stretch Goals (If Time Allows)

- **Adaptive re-planning**: Planner node that spawns sub-workflows dynamically
- **Agent-to-agent**: Orchestrator workflow that delegates to specialist sub-workflows based on task type
- **Embedding-based memory recall**: Replace keyword search with semantic similarity using a lightweight embedding model (e.g., `sentence-transformers` locally, or Gemini embeddings API)
- **Failure pattern detection**: After N runs, automatically identify common failure points and suggest workflow improvements
- **Multi-modal memory**: Store screenshots as part of memory so recovery agent has visual context

---

## Part 6: Key Tradeoffs and Risks

| Decision | Tradeoff |
|----------|----------|
| SQLite memory vs. vector DB | SQLite = zero setup, works offline, less impressive technically. Vector DB (ChromaDB) = better recall quality, extra dependency, 2-3 hours to integrate properly |
| Extend codegen vs. runtime agent | Codegen extension = generated code is inspectable and debuggable, consistent with existing pattern. Runtime agent = more dynamic, harder to debug in 48 hours |
| Build on Orbit vs. scratch | Orbit = faster to demo, large existing surface to show. Scratch = cleaner architecture, full control, but 80% of time spent on infra |
| Recovery as a node type vs. global wrapper | Node type = explicit, user-controlled, fits the visual paradigm. Global wrapper = automatic but opaque, harder to show judges what's happening |

**Biggest risk**: The Orbit backend currently allows only one workflow at a time. If the adaptive re-planning approach spawns sub-workflows, this becomes a concurrency bottleneck. Mitigation: implement sub-workflows as synchronous Python calls from within the parent workflow's generated code (bypass the HTTP layer entirely).

---

## Summary

Orbit-UI is an unusually strong foundation for this specific hackathon. The "Build Your Own OS" framing aligns directly with what Orbit already is. The visual workflow builder, real browser execution, and Docker VM are all demo-ready. The gaps — memory, recovery, adaptation — are well-defined and fillable in 48 hours with targeted additions rather than rewrites.

The single most important thing to build first is the memory layer, because it touches all four judging tracks: memory systems enable better tool use (recall past API formats), enable recovery (recall past failures), and enable adaptation (recall what strategies worked before).
