# Previous Designs

This directory contains proven orchestration patterns from past projects. The `/architect` agent references these when designing new systems.

## Purpose

- Avoid reinventing the wheel
- Learn from what worked (and what didn't)
- Provide concrete examples for new designs
- Enable pattern matching: "this is similar to X"

## How to Add Patterns

When you complete a project with interesting orchestration:

1. Create a subdirectory: `project-name/`
2. Include:
   - `README.md` - What the project does, what pattern it uses
   - `architecture.md` - How the orchestration works
   - Key code snippets (anonymized if needed)
   - Lessons learned

## Pattern Template

```markdown
# Project: [Name]

## What It Does
[Brief description]

## Orchestration Pattern
[Which pattern: sequential chain, parallel fan-out, router, feedback loop, multi-agent, etc.]

## Architecture
[Diagram or description of components and data flow]

## Key Decisions
- [Why this pattern over alternatives]
- [Trade-offs made]

## What Worked Well
- [Successes]

## What I'd Do Differently
- [Lessons learned]

## Reusable Components
- [Code or patterns that can be extracted]
```

## Current Patterns

### Proofstack Orchestration
**Location:** `proofstack-orchestration/`
**Pattern:** ReAct Loop with Separate Evaluation
**Key Innovation:** Evaluator only identifies gaps, planner handles re-planning

Key components:
- Scratchpad working memory (stateless API)
- Dynamic tool registry with runtime schema loading
- Confirmation workflow for destructive actions
- Enrichment pipeline (search → scrape → extract → validate)

### To Extract

- **brand-extractor** - Web crawling + AI extraction pipeline
