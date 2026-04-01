Now I have the full file. Here's the audit:

---

## SalesCommand — Code Audit Report

### 1. Overall Assessment

**File size:** 2,590 lines (~370 CSS + ~2,020 JS/HTML), single-file monolith. Approx 15,500 tokens of code.

**Strengths:**
- Impressive amount of functionality in one file — dashboard, pipeline, forecast, team, scorecards, coaching, goals, alerts, deals, AI chat, and a full Sandler qualification engine
- Clean, consistent dark-theme design system with well-structured CSS variables
- Sandler Copilot is genuinely deep: Assessment, Guided Discovery, Deal Intelligence, Messaging Plan, and Drafts tabs form a real coaching workflow
- Rule-driven alert generation (`generateAlerts()`) produces meaningful, actionable alerts from live data
- Sandler stage guardrails (`validateStageAdvance`) enforce qualification discipline — a core product differentiator
- AI Chat (canned responses) is contextually aware of last-viewed rep/deal
- Add Prospect modal captures both deal execution fields and prospect setup attributes (furniture industry-specific: store count, corporate structure, merchandising quality)
- Role-based filtering (CRO/VP/Manager/Rep) with nav and data scoping works

**Major risks / fragility:**
- All state is in-memory. A page refresh loses every edit, new prospect, Sandler field update, and coaching note. Only `copilotChats` and the API key persist via `localStorage`
- The AI Manager chat uses canned string-matching, not an LLM — users who type anything outside the ~12 recognized patterns hit a fallback wall
- No data persistence layer at all — no `localStorage` for DEALS/state, no API, no file export
- The entire app is built for a **generic B2B SaaS sales team**, not for RAP's specific business (furniture protection plans, dealer channel, designer acquisition)

---

### 2. Feature Inventory

#### Fully Working & Solid
| Feature | Notes |
|---|---|
| **Dashboard** (KPIs, revenue chart, funnel, leaderboard, unqualified pipeline card) | Fully rendered from DEALS/REPS data. Responsive. |
| **Pipeline page** (stage columns, KPIs, deal selection) | Click-to-expand works. Add Opportunity button present. |
| **Deals page** (table, stage filter tabs, deal detail panel) | Sorting by value, filter by stage, clickable rows all work. |
| **Forecast page** (commit/best/upside, WoW delta, per-rep table) | Clean display. Data derived correctly from reps. |
| **Team page** (activity vs target, quota, coverage, win rate) | Links to scorecards. |
| **Scorecards** (rep deep-dive: stats, activity, deals, goals, notes) | Well-structured. Sets AI context. |
| **Sandler Qualification Panel** (8-field edit/view popovers, score, stage guardrails) | Core differentiator. Edit flow, templates, guidance, and stage-advance validation all work. |
| **Sandler Copilot — Assessment tab** (situation, prioritized actions, top question) | Rule-driven, no AI needed. Solid coaching output. |
| **Sandler Copilot — Guided Discovery** (15 ordered questions, prereq gating, progress) | Deep and well-structured. |
| **Sandler Copilot — Deal Intelligence** (read-only synthesis, 6-dimension score) | Clean aggregation of Sandler + Discovery data. |
| **Sandler Copilot — Messaging Plan** (outreach sequence for champion/EB/stakeholders) | Auto-generates from captured intel. |
| **Sandler Copilot — Drafts** (email, call opener, manager summary) | Template-driven, copy-to-clipboard works. |
| **Copilot AI Chat** (per-deal, Anthropic API, full Sandler context in system prompt) | Real LLM integration. API key stored in browser. |
| **Alert system** (rule-driven: activity, coverage, stalled deals, Sandler gaps) | Comprehensive. Severity sorting, dismissal, nav badge. |
| **Coaching page** (auto-prioritized list, 1:1 prep, recent notes) | Functional. Links to AI for agenda generation. |
| **Goals page** (on-track/at-risk/behind, progress bars) | Static data but renders cleanly. |
| **Compose modal** (send coaching note, nudge, praise to rep/team) | Persists to COACHING_NOTES array. |
| **Add Prospect modal** (deal setup + account/prospect setup fields) | Validation, temp ID, pushes to DEALS array. Includes RAP-specific fields (store count, corporate structure, etc.). |
| **Help page** | Thorough feature guide. |
| **Role-based nav/filtering** | CRO/VP/Manager/Rep views filter data and nav correctly. |

#### Partially Working / Incomplete
| Feature | Notes |
|---|---|
| **AI Manager chat** | Works but is 100% canned string-matching. Handles ~12 intents well. Anything else hits fallback. Not connected to an LLM despite appearing to be one. |
| **Prospect Setup fields** | Captured into `prospectSetup` nested object on save, but **never displayed anywhere** after creation — no "flip modal back view" exists yet. |
| **Discovery "skip" feature** | Marks field as `(skipped)` which counts as filled for prereqs but is excluded from intelligence score. Slightly inconsistent — user may not realize skipped fields reduce their score. |

#### Stubbed / Placeholder (Not Functional)
| Feature | Notes |
|---|---|
| **`prospectSetup` flip modal** | CSS comment references "flip modal back view" — not implemented. |
| **Backend persistence** | Comments reference "DB layer will assign canonical deal ID" — no backend exists. |
| **`index.html` / `login.html`** | Sidebar links to these pages but they may not exist or be functional. |
| **Data export / import** | No mechanism to save or load data. |

#### Dead / Unused Code
| Item | Notes |
|---|---|
| `state.aiContext` for deals | Set when clicking deals but the AI chat only uses it for pronoun resolution ("this deal"), not for deeper context. |
| `forecast_prior_commit` | Hardcoded on each rep for WoW delta display. Never updated — always shows the same delta. |
| `MONTHLY` data | Static array, never modified. The "Mar" entry is hardcoded to `actual:260000` but the app doesn't derive this from DEALS. |
| `project-tree.txtdir` | Binary/UTF-16 file with "Invalid path" content — not useful. |

#### Conflicting or Problematic Logic
| Item | Notes |
|---|---|
| **DEALS is `const` but mutated** | `DEALS` is declared `const` but `.push()`, stage changes, and Sandler field edits mutate it. Works in JS but signals intent confusion. |
| **COACHING_NOTES is `let` but DEALS/REPS are `const`** | Inconsistent mutation strategy. Both are mutated at runtime. |
| **`escapeHTML` used inconsistently** | Some rendered strings use it, others inject raw HTML (e.g., `a.msg` in alerts contains `escapeHTML(d.company)` at generation time but `${r.name}` is unescaped). Low risk with dummy data, real risk with user input. |

---

### 3. RAP-Specific Gaps

**Dealer channel strengthening/expansion:**
- The app models generic B2B enterprise deals. There is **no concept of a "dealer"** as a distinct entity with stores, attachment rates, training status, or program performance
- Prospect Setup fields (store count, corporate structure, merchandising quality) are captured but never surfaced post-creation — no dealer health dashboard, no dealer scorecard
- No dealer activation workflow (onboarding checklist, training milestones, merchandising deployment)
- No dealer segmentation (A/B/C tiers, growth vs. maintenance, at-risk for churn)
- No "share of wallet" or attachment rate tracking per dealer
- No competitive displacement tracking (which dealers have Guardsman, Montage, etc.)
- No concept of "2026 tough retail market" — no macro context that shapes recommended actions

**Designer acquisition (new channel):**
- Zero designer-related functionality. No designer pipeline, no designer profile, no commerce engine integration
- No concept of the designer sales motion (which is fundamentally different from dealer sales — relationship-based, lower volume, higher margin, design project-driven)
- No distinction between dealer reps and designer-focused reps

**Force-multiplied ready-to-execute actions:**
- The Sandler Copilot + Drafts tab is the strongest "force multiplier" feature — it generates ready-to-send emails, call scripts, and manager summaries
- However, all actions assume the user knows Sandler methodology. An average rep would likely find the terminology intimidating
- No "daily action list" — what should this rep do RIGHT NOW, in priority order, with one-click execution?
- No pre-written objection handling scripts for RAP-specific objections ("protection plans don't sell," "my customers don't ask for it," "the margin isn't worth the floor space")

**CEO/sales manager visibility:**
- Dashboard KPIs are solid for a generic team view
- No RAP-specific metrics: total attachment rate, revenue per dealer, claims ratio, program penetration by region
- No "health of the business" view that a CEO would want (dealer churn risk, new dealer pipeline velocity, designer channel ramp)

**Porter-style strategic differentiation:**
- No competitive intelligence tracking
- No switching cost analysis per dealer
- No "moat" metrics (e.g., "how many dealers have our merchandising integrated into their floor?")

---

### 4. Technical Issues & Dead Ends

**Bug risks:**
- `escapeHTML` inconsistency — `generateAlerts()` uses template literals with `${r.name}` unescaped. If a rep name contained `<script>` it would inject. Low risk with controlled data but a real bug in production.
- `openComposeFor(v)` matches rep by name string, fragile if names change
- `_nextProspectId` starts at 9000 — collides if someone manually sets deal IDs in that range
- `DEALS.find(x=>x.id===state.selectedDeal)||DEALS[0]` — if `selectedDeal` is invalid, silently falls back to first deal rather than showing an error

**Performance / maintainability:**
- `generateAlerts()` is called on every Sandler field save, stage advance, and prospect add. It iterates all REPS and DEALS each time. Fine at current scale (<20 deals, 6 reps) but would degrade at 200+ deals
- Render functions build HTML strings with deeply nested template literals — very hard to debug or test
- No separation between data layer, business logic, and view rendering — all interleaved
- `render()` does a full innerHTML replace every time — loses scroll position, input focus, form state

**Unused / transitional code:**
- `state.aiContext` is set but barely used (only pronoun resolution)
- `forecast_prior_commit` is static — "WoW delta" is a lie (always shows the same number)
- `MONTHLY` data disconnected from DEALS — dashboard chart doesn't reflect actual closed revenue by month
- `prospectSetup` object is captured and stored but never read or displayed

**Confusion points for frontend dev:**
- Mixed `var`/`let`/`const` usage (Sandler popover code uses `var`, newer code uses `const`/`let`)
- Some functions use arrow syntax, Sandler popover code uses `function(){}` — inconsistent style
- `SANDLER_FIELD_LABELS` is defined twice (line 746 and embedded in SANDLER_FIELDS) — redundant
- Comments like `#1`, `#3`, `#9`, `#15`, etc. reference an issue tracker or changelog that isn't included — mysterious without context

---

### 5. Recommendations for First Cleanup Pass

**Priority order — do these before adding new intelligence:**

1. **Persist DEALS + state to localStorage** — Add a `saveState()` function that writes DEALS, COACHING_NOTES, and accountIntelligence to `localStorage` on every mutation. Load on init. This is 20 lines of code and eliminates the #1 user-facing bug (losing all work on refresh). Without this, no demo or training session is viable.

2. **Remove `forecast_prior_commit` lie or make it dynamic** — The "WoW delta" column on the Forecast page always shows the same number. Either remove it or add a mechanism to snapshot prior week's commit. Showing fake movement undermines trust in every other number.

3. **Surface `prospectSetup` data in Deal Detail** — The Add Prospect modal captures 8 RAP-specific fields (store count, corporate structure, geography, growth market flag, etc.) but they vanish after save. Add a "Prospect Profile" section to `renderDealDetail()` that displays these fields. This is pure rendering work — data is already there.

4. **Fix `escapeHTML` inconsistency in `generateAlerts()`** — Audit every alert template string. Replace `${r.name}` and `${d.rep}` with `${escapeHTML(r.name)}` etc. Quick regex-find-and-fix to prevent XSS when user-entered data flows through.

5. **Disconnect `MONTHLY` chart or derive from DEALS** — The dashboard "Revenue vs Target" chart uses a hardcoded `MONTHLY` array that doesn't match DEALS data. Either derive monthly revenue from Closed Won deal dates or add a comment making clear it's illustrative. Currently it silently contradicts the KPIs above it.

6. **Normalize `var`/`let`/`const` and function style** — Do a single pass converting all `var` to `const`/`let` and normalizing to arrow functions. This is cosmetic but removes confusion for the frontend dev and signals the codebase is maintained.

7. **Add a "Daily Actions" section to Dashboard or Coaching** — Before building agentic intelligence, add a simple static renderer that pulls: (a) deals with `due === 'Today'`, (b) reps with activity below 75% of target, (c) Sandler fields missing at late stages. Display as a prioritized checklist. This becomes the anchor point for future AI-generated actions and immediately serves the "force multiplier for average reps" goal.

8. **Rename brand from "SalesCommand" to RAP-specific branding** — The app currently presents as a generic B2B sales tool. Even before functionality changes, updating the title, sidebar brand, and help page to reference Risk Assurance Partners, furniture protection plans, and the dealer/designer channels will help stakeholders see this as *their* tool, not a template.







Now I have both files fully read. Let me produce both reports.

---

## Part 1: Agents.html Audit

### 1. Overall Assessment

**File:** 575 lines (~206 CSS, ~295 JS, ~74 HTML). Clean, well-structured single-page app.

**Strengths:**
- Visually polished hub-and-spoke topology diagram with interactive agent selection
- 7 well-defined agent roles (Router, Copilot, Sales Manager, Game Theory, Rewards, Signals, Research)
- Agent Intel Output panel shows convincing work product — research summaries, signal alerts, coaching recommendations, strategy analyses
- Chat sidebar with keyword-based routing to specialist agents
- Consistent design system (same CSS variables as sales_commander_main.html)
- Agent cards with status indicators, descriptions, and tag pills
- Filter-by-agent on intel output is functional and intuitive

**Weaknesses / Fragility:**
- **100% static/canned.** No LLM calls, no data flow from sales_commander_main.html, no shared state. Every response is a random pick from a hardcoded pool
- Chat routing is regex-based — hits the right agent most of the time but will misfire on ambiguous input
- Intel entries are frozen — same 8 entries every time, no generation, no timestamps that update
- Agent statuses (active/standby) are cosmetic — no actual on/off behavior
- KPI bar is hardcoded ("7 agents, 3 active, 4 standby, 8 interactions") — never updates
- **The topology renders twice** (lines 364-389) — the first render is immediately overwritten by the second. Dead code on lines 364-378
- `escapeHTML` is reimplemented differently than in main app (DOM-based vs string replace) — works fine but inconsistent
- No connection to DEALS, REPS, or any live data from the main app
- The `topo-bottom` node on line 375 renders `AGENTS[3]` (game-theory) which is already in the left column — this whole first render block is dead

### 2. Feature Inventory

| Feature | State | Notes |
|---|---|---|
| **Topology diagram** (hub & spoke) | Working | Interactive selection, spoke arrows, responsive collapse. |
| **Agent cards** (6 specialists) | Working | Selection highlight, status dots, descriptions, tags. |
| **Intel Output panel** (8 entries) | Working but static | Filters by agent. Content never changes. |
| **Chat sidebar** (keyword routing) | Working but canned | Routes to correct agent ~80% of the time. Random response from pool. |
| **KPI bar** | Cosmetic only | Hardcoded numbers. Never updates. |
| **Agent status** (active/standby) | Cosmetic only | No functional difference. |
| **Router dispatch** | Placeholder | Keyword regex, no real orchestration. |
| **Cross-app data flow** | Non-existent | No shared state, no API, no localStorage bridge to main app. |

### 3. What the Agent Team Gets Right Conceptually

The agent roster is well-chosen for the stated mission:
- **Copilot** = deal-level coaching (already partially built in main app's Sandler Copilot)
- **Sales Manager** = team-level analysis (already partially built in main app's AI Manager chat)
- **Game Theory** = strategic positioning (unique value — not in main app)
- **Signals** = external monitoring (not in main app, high value for RAP dealer intel)
- **Research** = market/competitive intel (not in main app)
- **Rewards** = behavioral incentives (not in main app)
- **Router** = orchestration layer (correct architectural pattern)

### 4. What's Missing or Broken

- No RAP-specific agent (no "Dealer Intelligence" or "Designer Acquisition" specialist)
- No concept of the agent team *pushing* insights to the human — the chat model is "human asks, agent answers," which is the exact opposite of the stated design intent
- No proactive briefing, no unsolicited alerts, no "the agent team noticed something and wants to tell you"
- Intel entries reference deals from main app (Meridian, Atlas, NorthStar) but have no data connection — they're just hardcoded text that happens to mention the same company names
- No mechanism for agents to *feed each other* (e.g., Research discovering something that triggers Copilot to update a messaging plan)

---

## Part 2: Integration Analysis — Merging Agents into SalesCommander

### The Core Design Principle You Stated

> "AI team main task is to enhance human intelligence, not follow prompts. It is the opposite — AI team pushes to human. Like an exoskeleton for the brain."

This inverts the standard chatbot pattern. The agent team should be an **always-on intelligence layer** that:
1. **Observes** deal data, rep activity, market signals, and qualification gaps
2. **Synthesizes** across agents (Research + Copilot + Game Theory together)
3. **Pushes** actionable insights to the human via the AI Manager interface
4. **Coaches** the human to be smarter — not waits to be asked

The AI Manager in sales_commander_main.html becomes the **delivery surface** — it's where the human sees the agent team's thinking, not where the human directs the agents.

### Current Architecture (Broken)

```
agents.html                    sales_commander_main.html
┌─────────────┐                ┌─────────────────────┐
│ Agent Team  │  (no link)     │  AI Manager Chat    │
│ Topology    │ ───────────X── │  Canned responses   │
│ Canned chat │                │  Sandler Copilot    │
│ Static intel│                │  Deal Intelligence  │
└─────────────┘                └─────────────────────┘
   Separate page                  Separate page
   No shared data                 No agent awareness
```

### Target Architecture (Exoskeleton Model)

```
sales_commander_main.html (unified)
┌──────────────────────────────────────────────────────┐
│  AI MANAGER (human-facing surface)                   │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Morning Briefing (pushed by agent team)        │ │
│  │  "3 things the team found overnight..."         │ │
│  │  Deal Alerts (pushed by Copilot + Signals)      │ │
│  │  Coaching Nudges (pushed by Sales Mgr + Rewards)│ │
│  │  Strategic Intel (pushed by Research + Game Thy) │ │
│  └─────────────────────────────────────────────────┘ │
│                        ▲                             │
│                        │ feeds                       │
│  ┌─────────────────────┴───────────────────────────┐ │
│  │  AGENT TEAM (background intelligence layer)      │ │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────┐ │ │
│  │  │Copilot │ │Sal.Mgr │ │Game Thy│ │ Research  │ │ │
│  │  │Signals │ │Rewards │ │Router  │ │           │ │ │
│  │  └────────┘ └────────┘ └────────┘ └──────────┘ │ │
│  │  Reads: DEALS, REPS, GOALS, accountIntelligence │ │
│  │  Writes: agentInsights[] → rendered in AI Mgr   │ │
│  └─────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Integration Steps

#### Phase 1: Merge Data + Agent Roster into Main App (Day 1-2)

**Step 1. Move AGENTS array into sales_commander_main.html**

Add the 7-agent roster (`AGENTS` from agents.html) as a top-level constant in sales_commander_main.html, alongside REPS and DEALS. This is the agent team's "org chart."

**Step 2. Create `agentInsights[]` — the push channel**

Add a new mutable array to state:
```js
let agentInsights = []; // populated by agent functions, consumed by AI Manager
```

Each insight has:
```js
{
  id: number,
  agentId: string,        // 'copilot', 'signals', etc.
  type: string,           // 'coaching', 'alert', 'research', 'strategy', 'nudge'
  priority: string,       // 'critical', 'high', 'medium', 'low'
  dealId: number|null,    // links to DEALS
  repId: number|null,     // links to REPS
  title: string,          // one-line summary
  body: string,           // full insight text
  timestamp: string,      // generated at creation
  dismissed: boolean,
  actionLabel: string,    // "Review Deal", "Open Scorecard", etc.
  actionFn: string        // onclick handler string
}
```

**Step 3. Build agent insight generators (rule-driven, no LLM needed)**

Port the 8 static `INTEL_ENTRIES` from agents.html into **functions** that generate insights from live data:

- `generateCopilotInsights()` — scans DEALS for Sandler gaps, stalled stages, missing economic buyers. Produces coaching recommendations tied to specific deals/reps.
- `generateSalesMgrInsights()` — scans REPS for activity shortfalls, coverage gaps, declining trends. Produces rep-level coaching nudges.
- `generateSignalsInsights()` — initially canned external signals; later wired to real APIs. Links to specific deals.
- `generateResearchInsights()` — initially canned market intel; later wired to search/enrichment APIs.
- `generateGameTheoryInsights()` — analyses competitive positioning on deals with known alternatives.
- `generateRewardsInsights()` — checks streaks, identifies reps for recognition or intervention.

Call `generateAllInsights()` at init and on every data mutation (same pattern as `generateAlerts()`).

**Step 4. Replace AI Manager's canned responses with agent-sourced content**

The current `AI_CANNED.greeting` (morning briefing) becomes:
```js
function buildMorningBriefing() {
  // Pull top 3-5 insights from agentInsights[], sorted by priority
  // Format as the "morning briefing" the AI Manager delivers
  // Each item credits the originating agent: "⚡ Copilot flagged..."
}
```

The `getAIResponse()` function stays as the human's way to *ask* for more detail, but the default behavior on page load is: **the AI Manager already has things to say**, pushed by the agent team.

#### Phase 2: Add Agent Team Panel to Main App (Day 2-3)

**Step 5. Add "Agent Team" as a nav item**

Add to NAV_ITEMS under the Intelligence group:
```js
{ id:"agents", icon:"🔀", label:"Agent Team" }
```

**Step 6. Build `renderAgentTeam()` page**

Port the topology diagram, agent cards, and intel output from agents.html into a `renderAgentTeam()` function inside sales_commander_main.html. This replaces agents.html entirely.

Key changes from agents.html:
- Intel Output panel reads from `agentInsights[]` (live, not static)
- Agent cards show real metrics (e.g., Copilot card shows "12 deals analyzed, 4 coaching actions pending")
- Clicking an insight navigates to the relevant deal or rep
- No separate chat — the AI Manager chat IS the conversation surface

**Step 7. Agent attribution in AI Manager chat**

When the AI Manager delivers an insight in the chat, prefix it with the originating agent's icon and name:
```
⚡ Copilot: Atlas Manufacturing — Sarah needs to confirm the economic buyer 
before sending the revised proposal. The Director of IT is a champion but 
lacks budget authority.

📊 Sales Manager: Tom Richter's activity dropped 28% this week. 
Recommend a 15-minute check-in before noon.
```

This makes the "team of agents working for you" visible without requiring the human to navigate to a separate page.

#### Phase 3: Proactive Push Behavior (Day 3-4)

**Step 8. AI Manager auto-populates on page load**

When the user opens the AI Manager page, instead of showing a static welcome message, auto-generate a briefing:
```js
function renderAI() {
  // If no chat history, generate morning briefing from agentInsights
  if (state.chatMessages.length <= 1) {
    const briefing = buildMorningBriefing();
    state.chatMessages.push({ role:'ai', text: briefing, agentId:'router' });
  }
  // ... existing render
}
```

**Step 9. Dashboard "Agent Intel" card**

Add a panel to the Dashboard (alongside Priority Alerts and Unqualified Pipeline) that shows the top 3 agent insights. This is the exoskeleton at work — the human sees agent thinking without asking for it.

**Step 10. Deal detail "Agent Notes" section**

In `renderDealDetail()`, add a section below Sandler Qualification that shows any `agentInsights` linked to that deal. This surfaces Research findings, Game Theory analyses, and Signal alerts right where the human is making decisions.

#### Phase 4: Retire agents.html (Day 4)

**Step 11.** Remove agents.html from the project. All functionality now lives inside sales_commander_main.html. Update any links (sidebar, index.html) to point to `navigate('agents')` instead.

### What NOT to Do

- **Don't build a prompt-to-agent chat interface.** The human shouldn't be directing the agents. The agents should be observing data and pushing insights. The human can ask follow-up questions through the AI Manager, but the default flow is agents → human, not human → agents.
- **Don't make agents "request-response."** They should run their analysis functions on data change and on page load, populating `agentInsights[]` proactively. The human never explicitly "runs" an agent.
- **Don't add a separate agent chat.** The AI Manager IS the conversation surface. Agent attribution (icon + name) in chat bubbles is sufficient to show which agent contributed what.
- **Don't wire LLM calls yet.** The rule-driven insight generators (scanning DEALS, REPS, GOALS for patterns) will produce valuable output without any API calls. LLM enhancement comes after the data flow architecture is proven.

### Summary: What Changes, What Stays

| Component | Action |
|---|---|
| `agents.html` | **Retire.** All content merges into main app. |
| `AGENTS` array | **Move** into sales_commander_main.html as a constant. |
| `INTEL_ENTRIES` | **Replace** with `agentInsights[]` generated from live data. |
| Agent topology diagram | **Port** into `renderAgentTeam()` page. |
| Agent chat sidebar | **Remove.** AI Manager chat is the only conversation surface. |
| AI Manager chat | **Upgrade.** Auto-populated with agent-sourced briefings. Agent attribution on responses. |
| `getAIResponse()` | **Keep** for human follow-up questions. Add agent insight awareness. |
| `generateAlerts()` | **Keep.** agentInsights is a parallel channel — alerts are system-level, insights are coaching-level. |
| Dashboard | **Add** "Agent Intel" card showing top 3 proactive insights. |
| Deal Detail | **Add** "Agent Notes" section showing deal-linked insights. |
| Sandler Copilot | **Keep as-is.** Copilot agent's insight generator reads the same data. |