# Proofstack Architecture - Quick Reference

## Pattern: ReAct Loop with Separate Evaluation

```
Plan → Execute → Evaluate → (Complete | Re-plan with gaps)
                              ↑
                              └── max 3 iterations
```

## Data Flow

```
                    ┌──────────────────────────────────────┐
                    │           FRONTEND                    │
                    │  (passes scratchpad with request)     │
                    └──────────────┬───────────────────────┘
                                   │
                                   ▼
┌──────────────────────────────────────────────────────────────────┐
│                         orchestrate()                             │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                      ReAct Loop                              │ │
│  │                                                              │ │
│  │  ┌─────────┐    ┌──────────┐    ┌───────────┐               │ │
│  │  │ Planner │───▶│ Executor │───▶│ Evaluator │               │ │
│  │  └─────────┘    └──────────┘    └─────┬─────┘               │ │
│  │       ▲                               │                      │ │
│  │       │         gaps + results        │                      │ │
│  │       └───────────────────────────────┘                      │ │
│  │                                                              │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│                              ▼                                    │
│                      ┌─────────────┐                              │
│                      │ Synthesizer │                              │
│                      └──────┬──────┘                              │
└─────────────────────────────┼────────────────────────────────────┘
                              │
                              ▼
                    ┌──────────────────────────────────────┐
                    │           RESPONSE                    │
                    │  { response, scratchpad, tools_used } │
                    └──────────────────────────────────────┘
```

## Scratchpad Schema (Zod)

```javascript
ScratchpadSchema = z.object({
  workspace_id: z.string(),
  turn_count: z.number(),
  established_icp: z.object({
    summary: z.string(),
    key_criteria: z.array(z.string()),
    chunk_ids: z.array(z.string())
  }).nullable(),
  active_prospects: z.array(z.object({
    company_name: z.string(),
    status: z.enum(['mentioned', 'researched', 'qualified', 'saved']),
    data: z.record(z.any())
  })),
  last_query: z.object({
    message: z.string(),
    tools_used: z.array(z.string()),
    results_summary: z.string()
  }).nullable(),
  pending_actions: z.array(z.object({
    action: z.enum(['save_prospect', 'enrich_prospect', 'build_list']),
    data: z.any(),
    awaiting_confirmation: z.boolean()
  }))
})
```

## Plan Schema

```javascript
PlanSchema = z.object({
  reasoning: z.string().max(1500),
  steps: z.array(z.object({
    tool: z.enum(['rag_search', 'web_search', 'crm_read', 'crm_write', 'qualify']),
    params: z.record(z.any()),
    purpose: z.string().max(500),
    depends_on: z.number().optional()
  })).min(1).max(15),
  scratchpad_updates: z.record(z.any()).optional()
})
```

## Key Interfaces

### Planner Input
```javascript
{
  message: "user request",
  contextSummary: "Turn 3. ICP: SaaS companies. Active: Acme, Beta",
  conversationHistory: [...],
  replanContext: {  // Only on re-plan
    gaps: ["Need website for Acme"],
    previousResults: "[web_search] SUCCESS: ..."
  }
}
```

### Planner Output
```javascript
{
  reasoning: "User wants to enrich leads. First fetch them from CRM.",
  steps: [
    { tool: "crm_read", params: { limit: 4 }, purpose: "Get recent leads" }
  ]
}
```

### Evaluator Output
```javascript
// Complete
{ complete: true }

// Incomplete
{
  complete: false,
  gaps: [
    "Write enrichment data to CRM for Acme: industry='SaaS'",
    "Score Beta Corp against ICP"
  ]
}
```

## Tool Executor Interface

```javascript
async function executeStep(step, workspaceId, scratchpad, previousResult) {
  const { tool, params } = step;
  const mergedParams = previousResult
    ? { ...params, _previous: previousResult }
    : params;

  switch (tool) {
    case 'rag_search': return await executeRagSearch(mergedParams, workspaceId);
    case 'web_search': return await executeWebSearch(mergedParams);
    case 'crm_read':   return await executeCrmRead(mergedParams, workspaceId);
    case 'crm_write':  return await executeCrmWrite(mergedParams, workspaceId);
    case 'qualify':    return await executeQualify(mergedParams, workspaceId);
  }
}
```

## Enrichment Flow

```
web_search({ query: "Acme Corp", enrich: true })
  │
  ├── 1. Search web for "Acme Corp"
  ├── 2. Get top result URL
  ├── 3. Scrape URL (Firecrawl)
  ├── 4. Extract structured data (GPT-4)
  ├── 5. Validate company name matches
  │
  └── Return: {
        company_name: "Acme Corp",
        website_url: "https://acme.com",
        enrichment: {
          industry: "SaaS",
          location: "San Francisco, CA",
          company_size: "50-200",
          description: "..."
        }
      }
```

## Error Handling

```javascript
try {
  const result = await executeStep(step, ...);
  results.push({ success: true, data: result });
} catch (error) {
  results.push({ success: false, error: error.message });
  // Continue to next step - don't abort plan
}
```

## Confirmation Flow

```
User: "Save Acme to CRM"
  │
  ├── scratchpad.pending_actions.push({
  │     action: 'save_prospect',
  │     data: { company_name: 'Acme', ... },
  │     awaiting_confirmation: true
  │   })
  │
  └── Response: "I'll save Acme. Confirm?"

User: "yes"
  │
  ├── isConfirmation("yes") → true
  ├── executeConfirmedAction(pending)
  └── clearPendingActions()
```
