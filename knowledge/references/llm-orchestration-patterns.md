# LLM Orchestration Patterns Reference

A catalog of orchestration patterns for the `/architect` agent.

## Pattern Catalog

### 1. Sequential Chain

```
Input → LLM A → LLM B → LLM C → Output
```

**Use when:**
- Each step depends on previous output
- Order matters (extract → validate → format)
- Simple, linear workflows

**Pros:**
- Easy to debug (follow the chain)
- Clear data flow
- Simple error handling

**Cons:**
- Latency adds up (sequential)
- One failure blocks entire chain
- Can't parallelize

**Example:**
```
Document → Extract entities → Validate entities → Format JSON → Store
```

---

### 2. Parallel Fan-Out / Fan-In

```
        ┌→ LLM A ─┐
Input ──┼→ LLM B ──┼→ Aggregate → Output
        └→ LLM C ─┘
```

**Use when:**
- Independent analyses can run simultaneously
- Need multiple perspectives on same input
- Latency is critical

**Pros:**
- Faster (parallel execution)
- One failure doesn't block others
- Get diverse outputs

**Cons:**
- Aggregation can be complex
- Higher token cost
- Need to handle partial failures

**Example:**
```
Article → [Sentiment, Summary, Keywords, Categories] → Aggregate → Profile
```

---

### 3. Router Pattern

```
Input → Router LLM → Route A → Handler A
                   → Route B → Handler B
                   → Route C → Handler C
```

**Use when:**
- Different inputs need different handling
- Classification determines next step
- Building support/assistant systems

**Pros:**
- Specialized handlers for each case
- Efficient (only run relevant handler)
- Easy to add new routes

**Cons:**
- Router can misclassify
- Need to handle unknown routes
- More prompts to maintain

**Example:**
```
Customer message → Classify intent → [Billing, Technical, Sales, General] → Specialized handler
```

---

### 4. Feedback Loop (Ralph Pattern)

```
┌─────────────────────────────────┐
│                                 │
│  Task → LLM → Validate ──┬─────→│ Complete
│            ↑             │      │
│            └── Fix ←─────┘      │
│                (if failed)      │
└─────────────────────────────────┘
```

**Use when:**
- Output must meet quality bar
- Have automated validators (tests, types, lint)
- Iterative refinement needed

**Pros:**
- Guaranteed quality (if validators are good)
- Self-correcting
- Can run autonomously

**Cons:**
- Can loop forever if validators too strict
- Token cost increases with iterations
- Need good validators

**Example:**
```
Requirement → Generate code → [Typecheck, Test, Lint] → Pass? → Commit : Fix
```

---

### 5. Multi-Agent Debate (Worktree Pattern)

```
                  Facilitator
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Architect    Detective     Craftsman
        │             │             │
        └─────────────┴─────────────┘
                      │
                  Consensus
```

**Use when:**
- Need multiple perspectives
- Trade-offs to evaluate
- Want to catch blind spots

**Pros:**
- Diverse viewpoints
- Higher quality decisions
- Self-reviewing

**Cons:**
- Coordination overhead
- Higher token cost
- Slower

**Example:**
```
Feature request → [Architect: design, Detective: security, Craftsman: quality] → Synthesize → Plan
```

---

### 6. Orchestrator Pattern

```
                 Orchestrator
                      │
    ┌─────────┬───────┼───────┬─────────┐
    ↓         ↓       ↓       ↓         ↓
  Agent A  Agent B  Agent C  Agent D  Agent E
```

**Use when:**
- Complex task requiring different specialists
- Dynamic routing based on context
- Building agent systems

**Pros:**
- Flexible composition
- Specialists stay focused
- Orchestrator handles complexity

**Cons:**
- Orchestrator is a bottleneck
- Complex to debug
- Need clear agent interfaces

**Example:**
```
"Build landing page" → Orchestrator → [Brand extract → Interview → UX Design → Ralph]
```

---

### 7. Hierarchical Delegation

```
Manager
   │
   ├── Team Lead A
   │      ├── Worker A1
   │      └── Worker A2
   │
   └── Team Lead B
          ├── Worker B1
          └── Worker B2
```

**Use when:**
- Very large tasks
- Need to decompose into subtasks
- Teams working on different parts

**Pros:**
- Scales to complex projects
- Clear ownership
- Parallel work possible

**Cons:**
- Coordination overhead
- Information loss between layers
- Complex debugging

---

### 8. RAG (Retrieval-Augmented Generation)

```
Query → Embed → Vector Search → Retrieve docs → LLM (with context) → Answer
```

**Use when:**
- Need to answer from knowledge base
- Information changes frequently
- Can't fit all context in prompt

**Pros:**
- Access to large knowledge
- Up-to-date information
- Reduces hallucination

**Cons:**
- Retrieval quality is critical
- Need embedding infrastructure
- Chunking strategy matters

---

### 9. Plan-and-Execute

```
Task → Planner LLM → [Step 1, Step 2, Step 3, ...] → Executor → Results
```

**Use when:**
- Multi-step tasks
- Need explicit plan before execution
- Want human review of plan

**Pros:**
- Explicit reasoning visible
- Can review/modify plan
- Handles complex tasks

**Cons:**
- Plans can be wrong
- Replanning on failure is hard
- Two-phase latency

---

### 10. Tool-Using Agent

```
Task → LLM → [Decide: answer | use tool] → Tool → Observe → LLM → [Decide: done | continue]
```

**Use when:**
- Need access to external capabilities
- Can't do everything with LLM alone
- Interactive exploration needed

**Pros:**
- Extends LLM capabilities
- Can interact with world
- Flexible problem solving

**Cons:**
- Tool errors propagate
- Can get stuck in loops
- Need good tool descriptions

---

## Choosing a Pattern

| Situation | Recommended Pattern |
|-----------|-------------------|
| Linear workflow, order matters | Sequential Chain |
| Multiple independent analyses | Parallel Fan-Out |
| Different inputs need different handling | Router |
| Need quality guarantee | Feedback Loop |
| Need multiple perspectives | Multi-Agent Debate |
| Dynamic specialist routing | Orchestrator |
| Very large, decomposable tasks | Hierarchical Delegation |
| Need external knowledge | RAG |
| Complex multi-step tasks | Plan-and-Execute |
| Need external capabilities | Tool-Using Agent |

## Combining Patterns

Patterns can be combined:

- **RAG + Feedback Loop**: Retrieve → Generate → Validate → Fix
- **Router + Sequential**: Classify → Route → Chain specific to route
- **Orchestrator + Multi-Agent**: Orchestrator decides which team to deploy
- **Plan-and-Execute + Tool-Using**: Plan steps → Execute with tools

## Anti-Patterns

### Over-Orchestration
- Too many layers between input and output
- Every call goes through 3 agents
- Fix: Simplify, use direct chains

### No Validators
- Feedback loops without quality checks
- Trust LLM output blindly
- Fix: Add automated validators

### Router Everywhere
- Every decision point is a router
- Classification overhead for simple cases
- Fix: Use conditionals when possible

### Infinite Context
- Trying to fit everything in prompt
- Context window overflow
- Fix: Use RAG, summarization, state management
