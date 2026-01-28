---
name: plan
description: Project planning with selectable methodologies. Invoke with method hints like "/plan with ralph", "/plan prd", or "/plan user stories".
---

# Plan Skill

Intelligent project planning that routes to specialized agents.

## Architecture Change

**Methods are now standalone agents.** This skill acts as a router to:

| Agent | Command | Location |
|-------|---------|----------|
| Interview Agent | `/interview` | `~/.claude/commands/interview.md` |
| Ralph Agent | `/ralph` | `~/.claude/commands/ralph.md` |
| Worktree Agent | `/worktree` | `~/.claude/commands/worktree.md` |
| Architect Agent | `/architect` | `~/.claude/commands/architect.md` |
| UX Design Agent | `/ux-design` | `~/.claude/commands/ux-design.md` |
| Brand Extract Agent | `/brand-extract` | `~/.claude/commands/brand-extract.md` |
| Orchestrate Agent | `/orchestrate` | `~/.claude/commands/orchestrate.md` |
| superUX Agent | `/superUX` | `~/.claude/commands/superUX.md` |

## Usage

```bash
/plan <description>                    # Auto-select agent based on task
/plan interview <description>          # Route to /interview agent
/plan with ralph <description>         # Route to /ralph agent
/plan worktree <description>           # Route to /worktree agent
/plan prd <description>                # Use PRD-style planning (legacy)
/plan user stories <description>       # Use user stories format (legacy)
```

## Direct Agent Invocation (Preferred)

Instead of going through `/plan`, users can invoke agents directly:

```bash
/interview "add user authentication"
/ralph                                  # Start execution loop
/worktree "complex feature with security concerns"
/architect "review my orchestration design"
/ux-design "make the header sticky"
/brand-extract "https://example.com"
/orchestrate "help me figure out how to approach this"
/superUX "what's that blue swirly text element?"  # Visual analysis & UX optimization
```

## Routing Logic

Parse the user's command to determine which agent to invoke:

| User Says | Route To | Best For |
|-----------|----------|----------|
| `interview`, `discover`, `clarify`, `questions` | `/interview` | Unclear requirements, need discovery |
| `with ralph`, `ralph`, `blocks`, `sprint`, `execute` | `/ralph` | Clear requirements, iterative execution |
| `worktree`, `multi-agent`, `agents`, `team`, `parallel` | `/worktree` | Multiple perspectives, complex features |
| `architect`, `design`, `orchestration`, `review` | `/architect` | LLM pipelines, architecture review |
| `ux`, `visual`, `design`, `ui` | `/ux-design` | Visual changes with validation |
| `brand`, `extract`, `identity` | `/brand-extract` | Brand analysis from URLs |
| `superUX`, `analyze`, `what is that`, `element`, `UX audit` | `/superUX` | Visual analysis, element identification, UX optimization |
| `help`, `approach`, `how should I`, `orchestrate` | `/orchestrate` | Routing, meta-planning |
| `prd`, `product`, `requirements` | `methods/prd.md` | Feature specs (legacy) |
| `user stories`, `stories`, `agile` | `methods/user-stories.md` | Team backlog (legacy) |

## Auto-Detection Heuristics

If no method is specified, analyze the task and route to the appropriate agent:

### Route to `/interview` when:
- Requirements are vague or incomplete
- User says "I want to..." or "I'm thinking about..."
- Task description lacks specifics
- User mentions "not sure", "maybe", "explore"

### Route to `/ralph` when:
- Task involves building/modifying with clear requirements
- User mentions "implement", "build", "execute"
- PRD or requirements already exist

### Route to `/worktree` when:
- Complex feature with multiple concerns
- Needs security review, architecture review, AND implementation
- User mentions "thorough", "review", "multiple perspectives"

### Route to `/architect` when:
- Designing AI/LLM pipelines
- Reviewing orchestration patterns
- User mentions "architecture", "design", "how should I structure"

### Route to `/ux-design` when:
- Visual/UI changes
- User mentions "look", "style", "responsive", "UI"

### Route to `/brand-extract` when:
- URL provided with brand analysis intent
- User mentions "brand", "identity", "analyze this site"

### Route to `/superUX` when:
- User asks "what is that element/thing?"
- Non-technical visual descriptions ("swirly", "bouncy", "that thingy")
- UX optimization or audit requests
- Design system extraction
- User mentions "analyze", "assess", "element identification"

### Route to `/orchestrate` when:
- Unclear which agent to use
- Complex multi-phase project
- User asks "how should I approach this?"

## Execution Flow

1. Parse user command for method/agent hints
2. If agent keyword found → invoke that agent directly
3. If no clear keyword → use auto-detection heuristics
4. If still unclear → invoke `/orchestrate` for routing help
5. Pass the task description to the selected agent

## Permissions

Planning operations are fully pre-approved. No permission prompts for:
- Read, Glob, Grep (all exploration)
- WebFetch, WebSearch (research)
- Task(Explore), Task(Plan) (subagents)
- All read-only bash commands

See `~/.claude/knowledge/AGENT-ARCHITECTURE.md` for full permissions model.

## Knowledge Base

All agents have access to:
```
~/.claude/knowledge/
├── AGENT-ARCHITECTURE.md     # System design
├── methodologies/            # How each agent works
│   ├── interview-protocol.md
│   ├── ralph-standard.md
│   ├── worktree-agents.md
│   └── ux-validation.md
├── previous-designs/         # Proven patterns
└── references/               # External knowledge
```

## State Storage

Each agent manages its own state:
- Interview: `.interview-session.json` in project root
- Ralph: `prd.json`, `progress.txt`, `.ralph-board.json`
- Worktree: `WORKTREE_COORDINATION.md`, `PLAN.md`
- UX Design: `screenshots/` directory
- Brand Extract: `output/` in brand-extractor project

## Legacy Methods

These remain in `methods/` for backward compatibility:
- `methods/prd.md` - PRD-style planning
- `methods/user-stories.md` - User story format

The interview, ralph, and worktree methods have been promoted to standalone agents and should be invoked directly.
