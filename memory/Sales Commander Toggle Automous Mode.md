

Sales Commander Agentic Team


Autonomous Toggle On/Off Mode


The toggle is workable; it just needs clear wiring so even “manual” mode doesn’t turn into a generic prompt circus. [nexaitech](https://nexaitech.com/multi-ai-agent-architecutre-patterns-for-scale/)

## 1) Two operating modes

- **Discovery Mode ON (autonomous)**  
  - Scheduler + Router automatically run the full Discover → Simulate → Select → Execute → Learn loop on a cadence and thresholds you define.  
  - Agents run background scans and simulations; AI Manager surfaces proposed plans and execution guidance as described.

- **Discovery Mode OFF (on‑demand)**  
  - Background Discover cycles are paused or rate‑limited.  
  - **Only AI Manager and explicit user actions** can trigger tasks to the team; users still never talk to worker agents directly.  
  - AI Manager can:  
    - run a **targeted mini‑scan** (e.g., “analyze this segment” or “repair this plan”),  
    - query specific reports (via Sales Manager, Copilot, etc.) when the manager asks for insight.

In both modes, AI Manager remains the **single contact surface**; Copilot, Marketing, Performance Coach, etc. stay hidden behind it.

## 2) Wiring the toggle

Add a simple flag in state:

```json
{
  "settings": {
    "discoveryMode": "on"   // on | off
  }
}
```

Router and Scheduler check this:

- If `discoveryMode == "on"`: schedule full cycles (daily scans, weekly planning) and dispatch Discover tasks automatically.  
- If `discoveryMode == "off"`:  
  - skip scheduled Discover tasks,  
  - only accept tasks from AI Manager that are directly tied to a user request or a current plan (e.g., “re‑score this plan,” “analyze these deals”).

AI Manager’s prompts are conditioned on the flag: in OFF mode, it explains that deeper discovery is disabled unless the toggle is turned on, preventing hidden token drains.

## 3) Keeping “manual” mode from becoming an average engine

Even with Discovery Mode off:

- Users can **only** request structured things from AI Manager (e.g., “review Q2 pipeline,” “prepare 1:1 for Alex”), which map to predefined tasks routed to specialist agents.  
- There is no free‑form “chat with Copilot/Marketing” UI and no unbounded brainstorming; everything still flows through the same schemas (Findings, Scorecards, Plans, PerformanceInsights).

So Discovery Mode controls *how proactively* the system hunts for asymmetric opportunities, not whether it behaves as a structured, plan‑driven engine.
