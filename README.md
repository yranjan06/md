You are running a four-role swarm to bring up and verify the Off-Duty product
end to end. The repository is already written and builds cleanly. Your job is to
get it running live with realistic cafe data, confirm the three interfaces stay
in sync, and stress test it until you are confident it will hold up in a recorded
demo.
Project context
Off-Duty is an inventory-aware ordering agent for a small cafe, built for the
Google Cloud Rapid Agent Hackathon, MongoDB track.

Backend: FastAPI plus MongoDB (motor), in backend/. Entry is
app.main:app. Services live in backend/app/services/. The agent loop is in
backend/app/agent/. Seed script is backend/scripts/seed.py.
Frontend: React plus Vite, in frontend/. Three role-locked screens at routes
/ (customer), /kitchen (cook), /owner. API client in frontend/src/api.js.
The hero feature is dynamic inventory-aware bundling in
backend/app/services/bundling.py. It scores each product by overstock, slow
sales velocity, and nearness to expiry, then offers the highest-pressure item
as a bundle at a discount that is always capped by owner rules.
Voice is optional. STT goes through Groq Whisper and TTS through Sarvam, both
proxied by the backend so keys never reach the browser. If keys are absent the
UI runs in text mode and nothing breaks.

Read these files first before doing anything: README.md,
docs/ARCHITECTURE.md, backend/README.md, backend/app/services/bundling.py,
backend/app/services/ordering.py, backend/app/services/inventory.py,
backend/scripts/seed.py, frontend/src/api.js, and the three screen files in
frontend/src/screens/.
The four roles
You will play all four. For each decision, the relevant role speaks first, then
the Integration Lead grills it before anything is executed. Do not skip the
grilling step. Write the discussion out loud so the reasoning is visible.

Backend Engineer. Owns FastAPI, MongoDB, services, and the seed data. Cares
about correct writes, immutable inventory events, and clean aggregation.
Frontend Engineer. Owns the three React screens and the polling that keeps
them in sync. Cares about the customer, cook, and owner views showing the same
truth within the poll window.
Data Steward. Owns realistic seed data and the demo narrative. Cares that the
numbers tell a believable cafe story and that the bundling feature has
something real to push.
Integration Lead, also the griller. Challenges every plan, looks for race
conditions and silent failures, and signs off only when acceptance criteria
pass.

Working agreement
For each phase: the owning role proposes a concrete plan in a few sentences. The
Integration Lead asks the hard questions and the owning role answers or revises.
Once settled, execute, then report what actually happened versus what was
expected. If they differ, fix it before moving on.
Important: the repo already has the user's own fixes
The person who owns this repository has already made changes and fixes of their
own on top of the original code. Do not assume the files are in their first
written state, and do not revert anything.
Before any phase, do this first:

Run git status and git diff to see what has already been changed, if the
repo is under git. If it is not, just read the current files as they are now.
Treat the current files on disk as the source of truth, not your memory of how
the code was originally written or any description in this prompt.
When you make a change, modify the current version in place. Build on the
user's fixes rather than overwriting them. If your plan would undo one of their
changes, stop and call it out, explain why, and ask before touching it.
If something this prompt describes does not match what you see in the code, the
code wins. Note the difference out loud and adapt your plan to the real state.

Integration Lead, enforce this: at the start, summarize what the user has already
changed so the rest of the swarm works from the real current code.
Phase 0: environment and build
Backend Engineer: set up a Python virtual environment, install
backend/requirements.txt, copy .env.example to .env, and fill MONGODB_URI
with a real Atlas cluster (a free M0 is fine for the demo). Set DB_NAME=offduty.
Leave GEMINI_API_KEY blank for now so the agent runs in its deterministic stub
mode during the first pass. Frontend Engineer: install frontend dependencies and
copy frontend/.env.example to frontend/.env with
VITE_API_BASE=http://localhost:8000.
Integration Lead, grill before running: What happens if the Atlas connection
string is wrong, does /health report it clearly rather than crashing on boot?
Confirm the lifespan index creation does not fail on a fresh empty database. Then
execute: build the frontend once to confirm it compiles, start the backend, and
hit /health. Expected: status ok, mongodb connected.
Phase 1: realistic seed data
Data Steward: review backend/scripts/seed.py. The demo story needs one item that
is clearly overstocked and near expiry and not selling, so the bundling brain has
an obvious thing to push. Confirm the brownie fits that role and the cold brew is
the popular anchor. If you want a richer menu, expand the product list to fifteen
or twenty real cafe items with believable prices, keeping the same shape, and keep
at least one slow overstocked perishable so the hero feature still fires.
Integration Lead, grill: After seeding, does GET /api/owner/push-scores return
the brownie at or near the top with a high score, and is the proposed discount at
or below the owner rule cap? If not, the data does not support the demo and must
be adjusted. Execute the seed, then call push-scores and read the result out loud.
Phase 2: the end-to-end live run, all three screens
Open three browser contexts so you can see them at once:

customer at http://localhost:5173/
cook at http://localhost:5173/kitchen
owner at http://localhost:5173/owner

Run this exact scenario and confirm the sync at every step. Each screen polls the
same backend, so a change on one must appear on the others within a few seconds.

On the customer screen, place an order for a cold brew. Use the demo order
button if the agent is in stub mode, or type the order if a Gemini key is set.
Expected: a bundle offer for the overstocked brownie appears, a bill is shown,
a UPI QR renders, and a token number is issued.
Confirm the order. Expected: the customer sees the token and an estimated wait.
Switch to the cook screen. Expected: within the poll window a new ticket
appears with that exact token number and the right items, marked unclaimed.
On the cook screen, claim the ticket. Expected: status moves to preparing and a
timer starts counting up.
Mark it delivered. Expected: status moves to ready.
Mark it collected. Expected: the ticket leaves the active board.
Switch to the owner screen. Expected: orders count went up, the audit trail
shows the create order, the bundle decision, and any negotiation, each with the
collections and document ids it touched. If stock crossed the threshold, a low
stock entry and a restock task appear.
On the owner screen, resolve the restock task by entering a new on hand value.
Expected: an inventory event with before and after is written and the item
leaves the low stock list.

Integration Lead, grill while you run it: Does the token on the cook screen match
the token the customer saw, not a different number. Did the inventory event record
the correct before and after, not a guess. Does the audit trail actually name the
collections and ids, or is it vague. Is the negotiation discount capped at the
owner rule even when the customer asks for more. If any answer is no, stop and fix
the service before continuing.
Phase 3: stress and failure grilling
Now the Integration Lead drives a set of harder cases. Run each one and confirm
the system behaves, then write down the result.

Two counters, one sequence. Place an order from counter_1 and another from
counter_2 close together. Confirm the token numbers are global and strictly
increasing across both, and the cook sees one combined line.
Low stock mid service. Drive an item below its threshold through repeated
orders. Confirm the soft warning behavior: existing orders still go through,
the item is flagged low, new recommendations stop pushing it, and a restock
task is created. Confirm placed orders are never cancelled.
Protected stock. Try to push a protected item through the bundler. Confirm it
is never offered below its protected floor.
Negotiation ceiling. Ask for a discount above the approval threshold. Confirm
it is marked needs approval rather than silently granted.
Voice off. With no STT or TTS keys, confirm the mic is disabled and the UI runs
in text mode with no errors in the console.
Agent fallback. With no Gemini key, confirm the chat still answers from real
inventory through the stub, and the order pipeline still works.
Concurrency. Fire two confirm requests at once and confirm tokens do not
collide and inventory does not go negative.

Integration Lead sign off: list each acceptance item below and mark it pass only
after you have seen it with your own eyes in the running app.
Acceptance criteria

Health endpoint reports mongodb connected.
Seed produces a push score list with the overstocked perishable on top.
Customer order produces a bundle, a bill, a UPI QR, and a token.
The same token appears on the cook screen within the poll window.
Claim starts a timer, delivered moves to ready, collected clears the ticket.
Owner orders count, audit trail, low stock, and restock tasks update live.
Inventory events record correct before and after values.
Token numbers are global and increasing across two counters.
Soft warning never cancels a placed order.
Negotiation never exceeds the owner cap, and high asks need approval.
Voice off and Gemini off both degrade gracefully with no crash.

When every item is pass, write a short report: what works, anything still rough,
and the exact commands to start all three screens for the recording. If a Gemini
key is available, do a second pass through Phase 2 with it set, so the agent
produces a natural spoken upsell line instead of the stub answer.
