Sales Commander Agentic Team

Mermaid Overview
Implementation‑oriented view and **agent catalog** describing roles and relationships of agentic team.

## Global agent graph (Mermaid)

```mermaid
flowchart LR
    %% Core state and orchestrator
    subgraph Core["Core Runtime"]
        ST[SalesCommanderState]
        RT[Router / Orchestrator]
    end

    %% Supervisor and goal-setter
    CEO[[CEO Agent<br/>Goal Setter]]
    AIM[[AI Manager<br/>Supervisor & Chief of Staff]]

    %% Opportunity & Strategy agents
    subgraph OppStrat["Opportunity & Strategy Agents"]
        RS[Research<br/>Market / Account Intel]
        SG[Signals<br/>External Monitor]
        GT[Game Theory<br/>Strategist]
        MKT[Marketing / Agency<br/>Porter + Blue Ocean + Sandler]
    end

    %% Pipeline & Performance agents
    subgraph PipePerf["Pipeline & Performance Agents"]
        SM[Sales Manager<br/>Analytics + Simulator]
        CP[Copilot<br/>Deal Coach]
        PC[Performance Coach<br/>(internal)]
        RW[Rewards<br/>Incentives & Gamification]
    end

    %% Value realization
    subgraph Value["Value Realization Agents"]
        OPS[Ops / Post‑Sale<br/>Implementation & Expansion]
    end

    %% Data flow with state
    ST <-- read/write --> RT

    RS --> ST
    SG --> ST
    GT --> ST
    MKT --> ST
    SM --> ST
    CP --> ST
    PC --> ST
    RW --> ST
    OPS --> ST

    %% Supervisor relationships
    CEO --> AIM
    ST --> AIM
    AIM --> RT

    %% Router dispatch
    RT --> RS
    RT --> SG
    RT --> GT
    RT --> MKT
    RT --> SM
    RT --> CP
    RT --> PC
    RT --> RW
    RT --> OPS

    %% Key inter‑agent dependencies (logical, via state)
    RS --> GT
    RS --> MKT
    SG --> GT
    SG --> MKT
    SM --> PC
    SM --> RW
    SM --> AIM
    CP --> AIM
    GT --> MKT
    MKT --> AIM
    RW --> PC
    OPS --> AIM
    OPS --> SM

    %% Human interaction
    AIM -->|Briefs, Plans, Instructions| HUMAN[(Managers & Reps)]
    HUMAN -->|High‑level choices, feedback| AIM
```

- Any graph‑based framework (LangGraph, CrewAI with a graph add‑on, ReDel, vendor SDKs) can map each box to a node and edges to message/state flow.

***

## Agent catalog: purpose, tools, inputs, outputs, relations

### 1) CEO Agent – Goal Setter

- **Purpose**: encode business intent.  
- **Tools**: none beyond config/state access.  
- **Reads**: leadership inputs, historical performance summaries.  
- **Writes**: `GoalSpec` (targets, risk tolerance, focus segments) into state.  
- **Upstream/Downstream**: feeds AI Manager; rarely called.

***

### 2) AI Manager – Supervisor & Chief of Staff

- **Purpose**: single human‑facing brain; supervises all agents, selects strategies, manages execution.  
- **Tools**:  
  - Full read/write on `SalesCommanderState`.  
  - Analytics helpers (grouping, ranking, filtering).  
  - Access to Sales Manager’s simulation/Monte Carlo API.  
- **Reads**: `GoalSpec`, all agent outputs (briefs, alerts, StrategyScorecards, PerformanceInsights, etc.).  
- **Writes**:  
  - `ProposedPlans` and chosen `Plans`.  
  - Human‑facing artifacts: Strategy Briefs, Manager Cockpit views, Daily Rep Cards, 1:1 prep sheets.  
  - Sub‑tasks for Router to dispatch to agents.  
- **Relations**:  
  - Above everything; only agent talking to humans.  
  - Uses Router to run cycles; uses Sales Manager for simulation; uses Performance Coach and Rewards for rep‑level guidance.

***

### 3) Router / Orchestrator (non‑LLM)

- **Purpose**: scheduling, task routing, and state merging; no domain intelligence.  
- **Tools**: scheduler, queue, shared‑state access, logging/trace.  
- **Reads/Writes**: `SalesCommanderState`, `Task` objects.  
- **Relations**:  
  - Called by AI Manager with tasks (e.g., `DailyScan`, `PlanReview`).  
  - Invokes the right subset of agents and merges their structured outputs back into state.

***

### 4) Research – Market / Account Intelligence

- **Purpose**: discover external patterns and levers.  
- **Tools**: web/API search, RAG over external docs, light fact‑checking.  
- **Reads**: segments, accounts, competitors from state; GoalSpec.  
- **Writes**: `ResearchFindings` (per segment/account) and candidate `Plays`.  
- **Relations**: informs Game Theory, Marketing, AI Manager.

***

### 5) Signals – External Activity Monitor

- **Purpose**: watch external events for target accounts.  
- **Tools**: social/news connectors, relevance classifier.  
- **Reads**: account list, tracked keywords.  
- **Writes**: `SignalsAlerts` tagged to accounts/deals.  
- **Relations**: informs Game Theory, Marketing, Sales Manager, Ops, AI Manager.

***

### 6) Game Theory – Strategist

- **Purpose**: design asymmetric strategies and counter‑moves.  
- **Tools**: scenario/“what‑if” prompts, simple payoff‑matrix helper.  
- **Reads**: ResearchFindings, SignalsAlerts, account/segment structures, GoalSpec.  
- **Writes**: `StrategyIdeas` and structured `Plays` with assumptions and target segments.  
- **Relations**: feeds Marketing and AI Manager; re‑uses its own ideas when plans get revised.

***

### 7) Marketing / Agency – Porter + Blue Ocean + Sandler

- **Purpose**: turn strategies into executable go‑to‑market plays.  
- **Tools**:  
  - Porter analysis templates (5 forces, value chain).  
  - Blue Ocean ERRC grid templates.  
  - Sandler‑aware messaging templates and style guides.  
  - Content generators for emails, sequences, pages.  
- **Reads**: StrategyIdeas, ResearchFindings, SignalsAlerts, Sandler metrics/patterns from state.  
- **Writes**: `CampaignBriefs`, `MessageFrameworks`, and per‑play notes: Porter summary, Blue‑Ocean ERRC, Sandler alignment.  
- **Relations**: feeds AI Manager and human marketers; influenced by Sandler metrics from Copilot/Sales Manager.

***

### 8) Sales Manager – Analytics + Simulator

- **Purpose**: own metrics, forecasting, and simulations.  
- **Tools**:  
  - Aggregation over pipeline and rep performance.  
  - Monte Carlo / scenario simulator service.  
- **Reads**: deals, accounts, reps, historical outcomes; active and candidate Plans.  
- **Writes**:  
  - `PipelineReports`, `ForecastScenarios`.  
  - `StrategyScorecards` (probability to hit GoalSpec, expected value, downside).  
- **Relations**:  
  - Feeds AI Manager for plan selection.  
  - Feeds Performance Coach and Rewards for rep coaching and incentives.  
  - Reads Ops signals to include post‑sale realization in simulations.

***

### 9) Copilot – Deal Coach

- **Purpose**: deal‑level Sandler coaching and pattern discovery.  
- **Tools**: access to deal/account data, notes/transcripts; Sandler rule engine; drafting tools.  
- **Reads**: specific deal contexts, Sandler metrics, active Plan focus.  
- **Writes**:  
  - `CopilotFindings` per deal (gaps, next moves, talk tracks).  
  - `CopilotPatterns` (repeatable moves by segment).  
- **Relations**: feeds AI Manager for deal focus and Performance Coach for rep‑level guidance.

***

### 10) Performance Coach – Rep Execution Coach (internal)

- **Purpose**: translate plans + metrics into day‑to‑day pushes and recognition (never talks directly to humans).  
- **Tools**: streak logic, threshold rules, behavioral‑nudge patterns.  
- **Reads**: Plans, rep activity, metrics, Rewards proposals.  
- **Writes**: `PerformanceInsights` per rep (priorityActions, streaks, riskFlags, recognitionMoments, coachNotes).  
- **Relations**: only visible to AI Manager; AI Manager uses these to generate daily cards, 1:1 prep, and weekly recaps.

***

### 11) Rewards – Incentives & Gamification

- **Purpose**: design incentive structures that support chosen strategies.  
- **Tools**: incentive templates, simple behavioral‑response models.  
- **Reads**: GoalSpec, Plans, rep historical response to incentives.  
- **Writes**: `IncentiveRecommendations` and gamified structures (badges, streak goals).  
- **Relations**: feeds Performance Coach and AI Manager.

***

### 12) Ops / Post‑Sale – Implementation & Expansion

- **Purpose**: ensure value realization on landed deals and find expansion.  
- **Tools**: milestone/usage trackers, churn/expansion risk scoring.  
- **Reads**: closed‑won deals, onboarding data, product usage.  
- **Writes**: `ImplementationAlerts`, `ExpansionOpportunities`, post‑sale outcome stats.  
- **Relations**:  
  - Feeds AI Manager for strategy adjustment.  
  - Feeds Sales Manager so simulations reflect realized value, not just bookings.

***
