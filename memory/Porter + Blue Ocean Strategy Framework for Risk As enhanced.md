<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# RAP Strategy Hub – Campaign + Marketing Function Architecture

## 1\. Purpose

This Markdown describes the architecture and prompt system for `porter-hub.html`, the RAP strategy hub that combines Porter’s Five Forces and Blue Ocean Strategy to drive campaign-specific marketing outputs (social ads, email, video scripts, value props, etc.).[^1](https://www.imd.org/blog/marketing/porters-five-forces/)[^3](https://corporatefinanceinstitute.com/resources/management/what-is-blue-ocean-strategy/)

\---

## 2\. Core Concepts and Hierarchy

### 2.1 High-Level Hierarchy

* **Global Marketing Functions** (general options)

  * Define *what kind* of marketing artifact you want (value prop, social ad, email nurture, etc.).
* **Campaigns** (children of the above)

  * Store Porter + Blue Ocean strategy for a specific goal/arena and buyer.
* **Prompt Instances**

  * For a given campaign + marketing function + user instructions, build a final prompt and send it to the best-suited AI engine.

This preserves your intent:

* The **prompt window** is general.
* The **campaign** is a child with its own strategic memory.
* The **marketing function** is chosen via dropdown and determines prompt structure and engine.

\---

## 3\. Data Model (TypeScript-Style)

### 3.1 Marketing Functions (Global Options)

```ts
type MarketingFunction =
  | "value\_prop"
  | "social\_ad"
  | "email\_nurture"
  | "video\_script"
  | "sales\_script"
  | "positioning\_deck";

type Engine = "Claude" | "GPT" | "Grok";

interface FunctionOption {
  id: MarketingFunction;
  label: string;        // e.g., "Social Media Ad"
  description: string;  // how it uses Porter + Blue Ocean
  defaultEngine: Engine;
}
```

These are fixed across the app and independent of any specific campaign.[^4](https://miro.com/blog/blue-ocean-strategy/)

### 3.2 Porter State (Per Campaign)

```ts
interface PorterForceDetail {
  score: number; // 1–5 intensity
  note: string;  // 1–2 line description
}

interface PorterState {
  threatNewEntrants: PorterForceDetail;
  buyerPower: PorterForceDetail;
  supplierPower: PorterForceDetail;
  threatSubstitutes: PorterForceDetail;
  rivalry: PorterForceDetail;
}
```

This captures industry structure and pressure for the campaign’s arena.[^2](https://guides.lib.fsu.edu/c.php?g=862230&p=10727782)[^5](https://www.isc.hbs.edu/strategy/business-strategy/Pages/the-five-forces.aspx)

### 3.3 Blue Ocean State (Per Campaign)

```ts
interface StrategyCanvasFactor {
  name: string;   // e.g., "Claims transparency"
  rapNow: number; // 1–10
  rapTarget: number; // 1–10
}

interface BlueOceanState {
  eliminate: string\[];
  reduce: string\[];
  raise: string\[];
  create: string\[];
  strategyCanvasFactors: StrategyCanvasFactor\[];
  positioningThesis: string; // RAP’s “big idea” for this arena
  brandDesired: string;      // desired 3-year perception
  brandActual: string;       // current perception
}
```

This encodes the ERRC grid and Strategy Canvas for each campaign.[^7](https://www.blueoceanstrategy.com/tools/four-actions-framework/)[^9](https://miro.com/blog/blue-ocean-strategy/)[^4](https://corporatefinanceinstitute.com/resources/management/what-is-blue-ocean-strategy/)

### 3.4 Campaign (Child of Functions)

```ts
type Arena = "dealer" | "designer" | "dtc" | "commerce\_engine";

interface Campaign {
  id: string;
  name: string;
  goal: string;       // e.g., "Grow designer-led plan attachment 30% in 9 months"
  arena: Arena;
  primaryBuyer: string; // e.g., "Principal designers in US coastal metros"
  porter: PorterState;
  blueOcean: BlueOceanState;
}
```

Each campaign holds its own Porter + Blue Ocean strategy and becomes the **campaign memory**.[^10](https://miro.com/blog/blue-ocean-strategy/)[^4](https://corporatefinanceinstitute.com/resources/management/what-is-blue-ocean-strategy/)

### 3.5 Campaign Prompt Instance

```ts
interface CampaignPrompt {
  id: string;
  campaignId: string;
  marketingFunction: MarketingFunction;
  engine: Engine;
  userPrompt: string;   // freeform user input in the window
  finalPrompt: string;  // constructed string sent to the LLM
  lastOutput?: string;
}
```



\---

## 4\. Campaign Snapshot Builder (Porter + Blue Ocean)

### 4.1 Purpose

`buildCampaignSnapshot` turns a campaign’s Porter + Blue Ocean state into a **compact, reusable text block** that can be injected into any function-specific prompt. This is how campaign memory auto-injects context and goals into each marketing prompt.[^8](https://www.wallstreetprep.com/knowledge/blue-ocean-strategy/)[^1](https://corporatefinanceinstitute.com/resources/management/what-is-blue-ocean-strategy/)[^7](https://umbrex.com/resources/frameworks/strategy-frameworks/blue-ocean-four-actions-framework/)

### 4.2 Function Definition

```ts
function buildCampaignSnapshot(c: Campaign): string {
  const dominantFactors = c.blueOcean.strategyCanvasFactors
    .filter(f => f.rapTarget > f.rapNow)
    .map(
      f =>
        `  - ${f.name} (now ${f.rapNow}/10 → target ${f.rapTarget}/10)`
    )
    .join("\\n");

  return `
CAMPAIGN SNAPSHOT
Company: Risk Assurance Partners (RAP)
Campaign: ${c.name}
Goal: ${c.goal}
Arena: ${c.arena}
Primary buyer: ${c.primaryBuyer}

PORTER HIGHLIGHTS
- Threat of new entrants (${c.porter.threatNewEntrants.score}/5): ${c.porter.threatNewEntrants.note}
- Buyer power (${c.porter.buyerPower.score}/5): ${c.porter.buyerPower.note}
- Supplier power (${c.porter.supplierPower.score}/5): ${c.porter.supplierPower.note}
- Threat of substitutes (${c.porter.threatSubstitutes.score}/5): ${c.porter.threatSubstitutes.note}
- Rivalry (${c.porter.rivalry.score}/5): ${c.porter.rivalry.note}

BLUE OCEAN HIGHLIGHTS
- ERRC grid:
  - Eliminate: ${c.blueOcean.eliminate.join(", ") || "—"}
  - Reduce: ${c.blueOcean.reduce.join(", ") || "—"}
  - Raise: ${c.blueOcean.raise.join(", ") || "—"}
  - Create: ${c.blueOcean.create.join(", ") || "—"}
- Strategy Canvas priorities (RAP should dominate):
${dominantFactors || "  - (none specified yet)"}
- Positioning thesis: ${c.blueOcean.positioningThesis || "—"}
- Brand gap: from "${c.blueOcean.brandActual}" to "${c.blueOcean.brandDesired}"
`.trim();
}
```



\---

## 5\. Prompt Builder and Template Routing

### 5.1 Template Lookup

```ts
function getTemplateForFunction(
  marketingFunction: MarketingFunction
): string {
  switch (marketingFunction) {
    case "value\_prop":
      return VALUE\_PROP\_TEMPLATE;
    case "social\_ad":
      return SOCIAL\_AD\_TEMPLATE;
    case "email\_nurture":
      return EMAIL\_NURTURE\_TEMPLATE;
    case "video\_script":
      return VIDEO\_SCRIPT\_TEMPLATE;
    case "sales\_script":
      return SALES\_SCRIPT\_TEMPLATE;
    case "positioning\_deck":
      return POSITIONING\_DECK\_TEMPLATE;
    default:
      return GENERIC\_TEMPLATE;
  }
}
```



### 5.2 Prompt Builder

```ts
function buildPrompt(
  campaign: Campaign,
  marketingFunction: MarketingFunction,
  userPrompt: string,
  includeContext: boolean
): string {
  const snapshot = includeContext ? buildCampaignSnapshot(campaign) : "";
  const baseTemplate = getTemplateForFunction(marketingFunction);

  return baseTemplate
    .replace("{{CAMPAIGN\_SNAPSHOT}}", snapshot)
    .replace("{{USER\_PROMPT}}", userPrompt.trim());
}
```

This is the core: one window, dropdown for function, checkbox for context, and the system composes the final prompt.

\---

## 6\. Function-Specific Base Templates

These templates **bake in Porter + Blue Ocean** so the outputs stay strategically consistent.[^9](https://miro.com/blog/blue-ocean-strategy/)[^4](https://www.blueoceanstrategy.com/tools/four-actions-framework/)

### 6.1 Value Proposition (Claude – Strategic)

```text
ROLE:
You are a senior strategy consultant specializing in Porter’s Five Forces and Blue Ocean Strategy.

{{CAMPAIGN\_SNAPSHOT}}

TASK:
Using the Porter and Blue Ocean context above, craft a differentiated value proposition for this RAP campaign in furniture protection plans.

Requirements:
- Use Porter insights to surface the main competitive pressure this campaign is responding to (e.g., strong buyer power of dealers/designers, high threat of substitutes like self-insurance, intense rivalry among warranty providers).
- Use Blue Ocean (ERRC + Strategy Canvas) to define the new value curve and explain why RAP’s offer is not a me-too dealer warranty:
  - Explicitly reference the Eliminate/Reduce/Raise/Create decisions.
  - Tie them to the factors where RAP wants to dominate on the Strategy Canvas.
- Output:
  1) A 1–2 sentence core value proposition.
  2) 3 supporting benefit pillars, each tied to:
     - one Porter force we are turning in our favor, and
     - one ERRC move (Eliminate/Reduce/Raise/Create).
  3) A one-line tagline that a designer or DTC furniture buyer would instantly understand.

User-specific direction:
{{USER\_PROMPT}}
```



### 6.2 Social Media Ad (GPT/Grok – Creative)

```text
ROLE:
You are a performance marketer turning a blue ocean positioning into social ads for furniture protection plans.

{{CAMPAIGN\_SNAPSHOT}}

TASK:
Create 3–5 social media ad variants for this RAP campaign.

Requirements:
- Reflect the ERRC “Raise” and “Create” elements:
  - Show higher value for designers/DTC buyers (e.g., no-claim-denial, cleaner client experience, profit-share).
  - Highlight any new outcomes or bundles (e.g., embedded protection in design proposals, commerce-engine powered upsells).
- Use Porter insights to name the red-ocean pain:
  - e.g., dealer control, commoditized warranties, claim denials, high buyer power.
- Make at least one ad focus on a Strategy Canvas priority where RAP aims to lead (e.g., claims transparency, workflow integration, partner profitability).
- For each ad, include:
  - Headline (max 10 words).
  - 1–2 sentence body.
  - Clear CTA.

User-specific direction:
{{USER\_PROMPT}}
```



### 6.3 Email Nurture (GPT – Structured)

```text
ROLE:
You are a B2B email strategist designing a nurture sequence anchored in Porter’s Five Forces and Blue Ocean Strategy.

{{CAMPAIGN\_SNAPSHOT}}

TASK:
Write a 4-email nurture sequence for this RAP campaign’s target buyer (designers, dealers, or DTC buyers as indicated in the snapshot).

Requirements:
- Email 1: Name the red-ocean situation:
  - Use Porter forces to describe the current pain (e.g., buyer power, substitutes, rivalry).
- Email 2: Introduce RAP’s blue ocean move:
  - Explain how the ERRC grid changes the rules of the game.
- Email 3: Make the new value curve concrete:
  - Describe, in simple language, how RAP scores higher on the Strategy Canvas factors that matter (e.g., claims experience, partner profit, workflow integration).
- Email 4: De-risk the choice:
  - Address the key substitutes and rivals and show why RAP’s positioning is safer, more profitable, and less stressful.

Each email should include:
- Subject line.
- Preview text.
- 3–5 short paragraphs.
- Clear CTA aligned with the campaign goal.

User-specific direction:
{{USER\_PROMPT}}
```



### 6.4 Video Script (Grok/GPT – Narrative)

```text
ROLE:
You are a scriptwriter turning a Porter + Blue Ocean strategy into a 3–5 minute video for RAP.

{{CAMPAIGN\_SNAPSHOT}}

TASK:
Write a video script explaining RAP’s shift from red-ocean dealer-centric protection plans to a blue-ocean designer/DTC-led model.

Requirements:
- Use at least one Porter force to frame the stakes (e.g., intense rivalry, powerful buyers, easy substitutes).
- Describe the “before” world of dealer add-on warranties vs the “after” world of RAP’s blue ocean positioning:
  - Map to the ERRC grid (what we eliminate, reduce, raise, and create).
  - Reference 2–3 Strategy Canvas factors where RAP aims to be clearly superior.
- Propose 2–3 visual beats where on-screen graphics could show:
  - A simple Strategy Canvas (old vs new value curve).
  - A radar chart of actual vs desired brand attributes.
- Tone: confident, founder-level, focused on “no-drama furniture” and better economics for partners and consumers.

User-specific direction:
{{USER\_PROMPT}}
```



### 6.5 Sales Script (Claude/GPT – Comparative)

```text
ROLE:
You are a sales enablement specialist designing a conversation script grounded in Porter’s Five Forces and Blue Ocean Strategy.

{{CAMPAIGN\_SNAPSHOT}}

TASK:
Create a sales script that helps RAP reps pivot from dealer-centric conversations to designer/DTC-centric opportunities.

Requirements:
- Opening: Frame the red-ocean dealer environment using Porter forces (e.g., high buyer power, rivalry, substitutes).
- Middle: Introduce RAP’s blue ocean actions:
  - Use the ERRC grid to contrast “old dealer add-on warranties” vs “new embedded protection engine”.
  - Weave in Strategy Canvas factors where RAP’s target curve is higher.
- Objection handling:
  - Use Porter insights to anticipate typical objections (risk, switching costs, existing supplier relationships).
  - Answer them using Blue Ocean logic (new value, reduced trade-offs).
- Close: Offer 2–3 specific next steps (e.g., pilot with one designer project, co-branded landing page, commerce-engine demo).

User-specific direction:
{{USER\_PROMPT}}
```



### 6.6 Positioning Deck (Claude – Long Form)

```text
ROLE:
You are a senior strategy consultant creating a positioning deck based on Porter’s Five Forces and Blue Ocean Strategy.

{{CAMPAIGN\_SNAPSHOT}}

TASK:
Produce a 10–15 slide outline for a positioning deck for this RAP campaign.

Requirements:
- Slides should:
  - Explain the current industry structure using Porter’s Five Forces (for the campaign’s arena).
  - Highlight structural threats and advantages.
  - Show RAP’s ERRC grid and how it creates a blue ocean space.
  - Visualize the Strategy Canvas in words (old value curve vs RAP’s target curve).
  - Articulate the positioning thesis and brand journey from actual to desired.
- Output:
  - Slide titles.
  - 3–5 bullets per slide.

User-specific direction:
{{USER\_PROMPT}}
```



\---

## 7\. UI Structure: One Prompt Window with Function Dropdown

### 7.1 Layout

* **Left Sidebar**

  * Campaign selector (list).
  * Small read-only summary: campaign goal, arena, primary buyer.
  * Tiny Porter + Blue Ocean summary (scores, ERRC bullets).[^1](https://www.imd.org/blog/marketing/porters-five-forces/)[^7](https://www.blueoceanstrategy.com/tools/four-actions-framework/)
* **Main Area**

  * Top controls:

    * Dropdown: “Marketing function”

      * Value Prop
      * Social Ad
      * Email Nurture
      * Video Script
      * Sales Script
      * Positioning Deck
    * Engine badge: auto-filled from `FunctionOption.defaultEngine` (editable).
  * **Prompt window**:

    * Textarea for `userPrompt`.
    * Checkbox: “Include campaign strategy context (Porter + Blue Ocean)”.
    * Collapsible preview: shows `buildCampaignSnapshot(campaign)` if context is enabled.
    * “Generate” button → calls `buildPrompt` and sends to selected engine.

This keeps the UX simple, while making strategy reusable and campaign-specific.

\---

## 8\. Example End-to-End Flow

1. User creates a **Campaign**:

   * Names it “Designers – No-Drama Furniture Q3”.
   * Fills goal, arena = `designer`, primary buyer = “interior design studios”.
   * Completes quick Porter + Blue Ocean wizard (forces scores, ERRC, Strategy Canvas, thesis).[^2](https://corporatefinanceinstitute.com/resources/management/what-is-blue-ocean-strategy/)[^10](https://miro.com/blog/blue-ocean-strategy/)
2. User opens the **prompt window**:

   * Selects campaign “Designers – No-Drama Furniture Q3”.
   * Selects marketing function “social\_ad”.
   * Keeps “Include campaign strategy context” checked.
   * Types: “Focus on our no-claim-denial promise and revenue share for designers.”
3. System:

   * Runs `buildCampaignSnapshot(campaign)` → snapshot text.
   * Gets `SOCIAL\_AD\_TEMPLATE`.
   * Runs `buildPrompt(campaign, "social\_ad", userText, true)`.
   * Sends final prompt to GPT or Grok.
4. Output:

   * Ads that already reflect RAP’s Porter + Blue Ocean strategy without the user needing to restate it.

This Markdown file is ready to give Claude (and your devs) as the single source of truth for implementing the RAP strategy hub.

<div align="center">⁂</div>

