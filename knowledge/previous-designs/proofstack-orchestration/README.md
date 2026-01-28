# Proofstack Orchestration

> B2B sales enablement assistant with ReAct loop orchestration

**Source:** `/home/rancho/proofstack-demo/lib/orchestrator/`
**Tech Stack:** JavaScript, OpenAI GPT-4, Supabase, Firecrawl, Zod

## What It Does

AI assistant for B2B sales that can:
- Search knowledge base (RAG with pgvector)
- Research companies on the web
- Read/write CRM records (opportunities)
- Score prospects against ICP criteria

## Orchestration Pattern

**ReAct Loop with Separate Evaluation**

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  User Message                                           │
│       │                                                 │
│       ▼                                                 │
│  ┌─────────┐                                            │
│  │ PLANNER │ ─── Generates execution plan (JSON)        │
│  └────┬────┘                                            │
│       │                                                 │
│       ▼                                                 │
│  ┌──────────┐                                           │
│  │ EXECUTOR │ ─── Runs each step, collects results      │
│  └────┬─────┘                                           │
│       │                                                 │
│       ▼                                                 │
│  ┌───────────┐    Complete?                             │
│  │ EVALUATOR │────────────┬─── Yes ──▶ SYNTHESIZER      │
│  └─────┬─────┘            │                             │
│        │                  │                             │
│        │ Gaps found       │                             │
│        ▼                  │                             │
│  RE-PLAN with gaps ───────┘                             │
│  (max 3 iterations)                                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Key Design Decisions

### 1. Evaluator Only Identifies Gaps (Doesn't Plan)

```javascript
// WRONG: Evaluator generates plans
evaluate() → { complete: false, nextSteps: [...] }

// RIGHT: Evaluator only identifies gaps, planner replans
evaluate() → { complete: false, gaps: ["Need website for Acme Corp"] }
generatePlan(message, replanContext: { gaps, previousResults })
```

**Why:** Separation of concerns. Evaluator is a judge, planner is the executor. Mixing roles caused confusion.

### 2. Scratchpad = Session Working Memory

```javascript
{
  workspace_id: "uuid",
  turn_count: 3,
  established_icp: { summary: "...", key_criteria: [...] },
  active_prospects: [
    { company_name: "Acme", status: "researched", data: {...} }
  ],
  last_query: { message: "...", tools_used: [...] },
  pending_actions: [
    { action: "save_prospect", data: {...}, awaiting_confirmation: true }
  ]
}
```

Scratchpad is:
- Passed from frontend (stateless API)
- Validated with Zod on every request
- Updated by both planner and executors
- Serialized back in response

### 3. Plan Only What You Have Data For

The planner is explicitly told:
> "If a step requires data you don't have yet, plan ONLY the data-fetching step. You'll get another chance after seeing results."

**Example:**
- User: "Enrich my last 4 leads with website URLs"
- Iteration 1: Plan `crm_read` to get the 4 leads
- Iteration 2: After seeing names, plan 4 `web_search` calls with actual names

### 4. Dynamic Tool Schema

CRM fields are loaded from the database at runtime:

```javascript
export async function formatToolsForPrompt() {
  const crmSchema = await getCrmSchemaDescription(); // From Supabase
  // Include in system prompt
}
```

**Why:** Allows users to customize CRM schema without code changes.

### 5. Confirmation Workflow

Destructive actions require confirmation:

```javascript
pending_actions: [
  { action: "save_prospect", data: {...}, awaiting_confirmation: true }
]

// Next message: "yes" / "no"
isConfirmation(message) → execute pending action
isCancellation(message) → clear pending action
```

## File Structure

```
lib/orchestrator/
├── index.js          # Main orchestrate() loop
├── scratchpad.js     # Working memory management
├── validators.js     # Zod schemas for all boundaries
└── router.js         # Intent classification helpers

lib/executors/
├── index.js          # Executor registry
├── rag.js            # Knowledge base search
├── web.js            # Web search + scrape + enrichment
├── crm.js            # Supabase read/write
└── qualify.js        # ICP scoring

lib/tools/
└── index.js          # Tool registry with schemas

lib/prompts/
├── planner.js        # Plan generation prompts
├── extraction.js     # Data extraction prompts
└── system.js         # System prompts
```

## Core Functions

### orchestrate()
```javascript
async function orchestrate(message, workspaceId, conversationHistory, rawScratchpad) {
  // 1. Validate scratchpad
  // 2. Check for pending confirmations
  // 3. Start turn
  // 4. ReAct loop (max 3 iterations):
  //    - generatePlan()
  //    - executePlan()
  //    - evaluate()
  //    - if gaps, re-plan with context
  // 5. synthesize() response
  // 6. Complete turn
  return { response, scratchpad, tools_used, debug }
}
```

### generatePlan()
```javascript
async function generatePlan(message, scratchpad, history, replanContext) {
  // Build dynamic system prompt with tool schemas
  // Include replan context (gaps + previous results) if re-planning
  // Return validated plan JSON
}
```

### evaluate()
```javascript
async function evaluate(originalMessage, results, scratchpad, history) {
  // Compare original request to results
  // Return { complete: true } or { complete: false, gaps: [...] }
  // DOES NOT generate next steps
}
```

## Tool Registry

| Tool | Purpose | Params |
|------|---------|--------|
| `rag_search` | Knowledge base search | query, chunk_types, limit |
| `web_search` | Web search + enrichment | query, company_name, enrich, url |
| `crm_read` | Read opportunities | limit, status, company_name, sortBy |
| `crm_write` | Create/update opportunities | company_name, ...fields |
| `qualify` | Score against ICP | company_name, company_info |

## Enrichment Pipeline

When `web_search` is called with `enrich: true`:

```
1. Search web for company
2. Get top result URL
3. Scrape URL with Firecrawl
4. Extract structured data with GPT-4
5. Validate company name matches (prevent mixups)
6. Return CRM-ready fields
```

## What Worked Well

1. **Separate evaluator** - Clean separation reduced planning errors
2. **Iteration cap** - Max 3 iterations prevents infinite loops
3. **Dynamic schemas** - CRM flexibility without code changes
4. **Scratchpad** - Context persists without database storage
5. **Rich logging** - Colored, timestamped logs for debugging

## What Could Be Improved

1. **Parallel execution** - Steps run sequentially, could parallelize independent ones
2. **Streaming** - Currently waits for full response
3. **Error recovery** - Failed steps don't have retry logic
4. **Token tracking** - Not tracking costs across iterations

## Reusable Components

### Scratchpad Pattern
Working memory that travels with the request. Useful for any multi-turn agent.

### Evaluator Pattern
Separate evaluation from planning. Evaluator is a judge, not an actor.

### Dynamic Tool Registry
Tools with schemas loaded at runtime. Great for customizable agents.

### Confirmation Workflow
Pending actions requiring user approval before execution.

## Integration Points

- **Frontend:** Passes scratchpad, receives updated scratchpad
- **Supabase:** CRM storage, schema introspection
- **Firecrawl:** Web search and scraping
- **OpenAI:** GPT-4 for planning, evaluation, synthesis
