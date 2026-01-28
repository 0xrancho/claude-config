# Agentic Toolbox Decision Framework

Use this during plan mode to select appropriate tools for your agentic solution.

---

## 1. Data Acquisition

### Do you need web data?

```
Need web data?
│
├─ YES → What kind?
│   │
│   ├─ Static content (articles, docs, product pages)
│   │   │
│   │   ├─ One-off/few pages → Simple fetch (requests, httpx)
│   │   ├─ Many pages, same site → Crawler (Firecrawl crawl, Scrapy)
│   │   └─ Need structured output → Extraction API (Firecrawl extract, Hyperbrowser extract)
│   │
│   ├─ Dynamic/JS-heavy content
│   │   │
│   │   ├─ Need to interact (click, scroll, fill forms) → Browser automation (Playwright, Puppeteer)
│   │   ├─ Just need rendered HTML → Headless browser or scraping API with JS rendering
│   │   └─ Behind auth/CAPTCHA → Managed browser (Hyperbrowser, Browserbase, Apify)
│   │
│   ├─ Search results
│   │   │
│   │   ├─ Google/Bing results → Search API (SerpAPI, Serper, Firecrawl search)
│   │   └─ Site-specific search → Site's API if available, else scrape
│   │
│   └─ Real-time/streaming data → WebSockets, SSE, or polling
│
└─ NO → Skip to Section 2
```

### Web Tool Selection Matrix

| Requirement | Simple Fetch | Firecrawl | Hyperbrowser MCP | Playwright |
|-------------|--------------|-----------|------------------|------------|
| Static HTML | Yes | Yes | Yes | Overkill |
| JS rendering | No | Yes | Yes | Yes |
| Structured extraction | Manual | Built-in | Built-in | Manual |
| Multi-page crawl | Manual | Built-in | Built-in | Manual |
| Form interaction | No | Limited | Yes | Yes |
| CAPTCHA handling | No | Partial | Yes | Manual |
| Anti-bot bypass | No | Good | Excellent | DIY |
| Agent autonomy | Via code | Via code | Native | Via code |

### Integration Pattern Decision

```
How will the agent use this tool?
│
├─ Agent decides what/when to scrape autonomously
│   │
│   ├─ Using MCP-native client (Claude Desktop, Cursor) → MCP server
│   └─ Custom orchestrator → Wrap as tool function
│
├─ You control scraping, feed results to agent
│   │
│   └─ REST API (Firecrawl, Apify, etc.)
│
└─ Batch/scheduled jobs (no agent involvement)
    │
    └─ Scrapy, cron + scripts, or managed service
```

---

## 2. External APIs & Services

```
Need external service data?
│
├─ Structured API available?
│   │
│   ├─ YES → Use official SDK/REST client
│   │   │
│   │   ├─ Rate limited? → Add retry logic, caching
│   │   ├─ Auth required? → Credential management (env vars, secrets manager)
│   │   └─ Complex auth (OAuth)? → Use SDK, don't roll your own
│   │
│   └─ NO → Scrape (see Section 1) or find alternative
│
├─ Need to aggregate multiple APIs?
│   │
│   └─ Consider: API aggregator, custom middleware, or agent tool composition
│
└─ Real-time updates needed?
    │
    ├─ Webhooks available → Set up receiver endpoint
    └─ No webhooks → Polling with appropriate interval
```

---

## 3. Data Processing & Transformation

```
What processing is needed?
│
├─ Text transformation
│   │
│   ├─ Simple (regex, split, join) → Built-in string ops
│   ├─ Structured parsing (HTML, JSON, XML) → BeautifulSoup, jq, lxml
│   ├─ NLP tasks → spaCy, NLTK, or LLM
│   └─ LLM-based extraction → Structured output (JSON mode, function calling)
│
├─ Data enrichment
│   │
│   ├─ Lookup/join with external data → API calls, database joins
│   ├─ AI-generated fields → LLM with schema enforcement
│   └─ Computed fields → Transform functions
│
├─ Validation & cleaning
│   │
│   ├─ Schema validation → Pydantic, JSON Schema, Zod
│   ├─ Deduplication → Hash-based or fuzzy matching
│   └─ Normalization → Custom transform pipeline
│
└─ Large dataset processing
    │
    ├─ Fits in memory → pandas, polars
    ├─ Too large for memory → Dask, chunked processing
    └─ Need persistence → Database (see Section 4)
```

---

## 4. Storage & Persistence

```
Where does data live?
│
├─ Ephemeral (single session)
│   │
│   └─ In-memory structures (dict, list, dataclass)
│
├─ Persistent, simple structure
│   │
│   ├─ JSON files → For config, small datasets (<10k records)
│   ├─ CSV/Parquet → For tabular data, analytics
│   └─ SQLite → For relational queries, single-user
│
├─ Persistent, complex/large
│   │
│   ├─ Relational (users, transactions) → PostgreSQL, MySQL
│   ├─ Document store (flexible schema) → MongoDB, Firestore
│   ├─ Vector store (embeddings, RAG) → Pinecone, Weaviate, pgvector
│   └─ Key-value/cache → Redis, Valkey
│
└─ Need search?
    │
    ├─ Full-text search → Elasticsearch, Meilisearch, PostgreSQL FTS
    └─ Semantic search → Vector store + embeddings
```

---

## 5. Agent Tool Exposure

### How to expose capabilities to the agent

```
Tool exposure method?
│
├─ MCP-native client
│   │
│   ├─ Existing MCP server available → Configure and connect
│   └─ Custom capability needed → Build MCP server (TypeScript/Python SDK)
│
├─ Framework-based (LangChain, CrewAI, etc.)
│   │
│   └─ Define as tool/function with schema
│       │
│       ├─ Simple I/O → @tool decorator or Tool class
│       └─ Stateful/complex → Custom tool with state management
│
├─ Raw LLM API (Claude, OpenAI)
│   │
│   └─ Function calling / tool_use
│       │
│       ├─ Define JSON schema for parameters
│       ├─ Handle tool calls in your code
│       └─ Return results to model
│
└─ Custom orchestrator
    │
    └─ Whatever interface you've built
```

### Tool Schema Checklist

- [ ] Clear, descriptive name
- [ ] Concise description (what it does, when to use)
- [ ] Well-typed parameters with descriptions
- [ ] Enum constraints where applicable
- [ ] Required vs optional clearly marked
- [ ] Return type documented

---

## 6. Output & Delivery

```
What happens with the result?
│
├─ Display to user
│   │
│   ├─ Terminal/CLI → Formatted text, tables
│   ├─ Web UI → HTML/JSON API response
│   └─ File download → Generate and serve file
│
├─ Pass to another system
│   │
│   ├─ Webhook/API call → HTTP client with retry
│   ├─ Message queue → RabbitMQ, Redis pub/sub, SQS
│   └─ Database write → ORM or raw SQL
│
├─ Feed to another agent/LLM
│   │
│   ├─ Same context → Return as tool result
│   ├─ Different context → Summarize to reduce tokens
│   └─ Async handoff → Queue or event system
│
└─ Store for later
    │
    └─ See Section 4 (Storage)
```

---

## 7. Quick Reference: Common Patterns

### Web Research Agent
```
Tools needed:
├─ Web search (SerpAPI, Tavily, or built-in)
├─ Page fetch with extraction (Firecrawl, Hyperbrowser)
├─ Note storage (in-memory or file)
└─ Output formatter
```

### Data Enrichment Pipeline
```
Tools needed:
├─ Source reader (file, API, database)
├─ Enrichment APIs (lookup services)
├─ LLM for inference/generation
├─ Validator (schema enforcement)
└─ Sink writer (database, file)
```

### Interactive Automation Agent
```
Tools needed:
├─ Browser automation (Playwright via MCP or direct)
├─ Screenshot/DOM reading
├─ Form filling
├─ State tracking
└─ Human-in-the-loop checkpoints
```

### RAG System
```
Tools needed:
├─ Document loader (file readers, web fetch)
├─ Chunker/splitter
├─ Embedding model
├─ Vector store
├─ Retriever
└─ LLM for generation
```

---

## 8. Decision Checklist

Before finalizing tool selection:

- [ ] **Necessity**: Is this tool actually needed, or can existing tools cover it?
- [ ] **Complexity**: Am I over-engineering? Simpler is better.
- [ ] **Token efficiency**: Will this bloat context? Can I extract only what's needed?
- [ ] **Error handling**: What happens when this tool fails?
- [ ] **Rate limits**: Will I hit API limits? Do I need caching/throttling?
- [ ] **Cost**: Per-call pricing? Can I batch or cache?
- [ ] **Auth**: How are credentials managed? Secure?
- [ ] **Testability**: Can I test this tool in isolation?

---

## 9. Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Scraping when API exists | Fragile, ToS issues | Use official API |
| Full page in context | Token waste | Structured extraction |
| New tool per variation | Complexity explosion | Parameterized single tool |
| Synchronous everything | Slow, blocking | Async where beneficial |
| No error handling | Silent failures | Explicit error returns |
| Hardcoded credentials | Security risk | Environment variables |
| Over-abstraction | Premature complexity | Start simple, refactor when needed |
