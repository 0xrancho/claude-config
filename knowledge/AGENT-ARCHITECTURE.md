# Agent Architecture & Permissions Model

> Personal agent library for rancho - specialized agents with their own contexts, knowledge bases, and behaviors.

---

## Philosophy

**Agents are not plan methods.** They're independent entities with:
- **Identity** - A name and role (Interview Agent, Architect Agent)
- **Knowledge** - Access to specific methodologies and previous designs
- **Capabilities** - What tools they can use
- **Handoffs** - Which agents they can delegate to

**Planning is read-only.** No permission prompts needed for exploration.

---

## Directory Structure

```
~/.claude/
├── commands/                        # Agent entry points (/agent-name)
│   ├── interview.md                 # Discovery Q&A sessions
│   ├── worktree.md                  # Multi-agent collaboration
│   ├── ralph.md                     # Execution loop methodology
│   ├── ux-design.md                 # Visual development + Playwright
│   ├── brand-extract.md             # Brand analysis (wraps project)
│   ├── architect.md                 # LLM orchestration specialist
│   └── orchestrate.md               # Meta-agent, knows all agents
│
├── knowledge/                       # Shared agent knowledge base
│   ├── AGENT-ARCHITECTURE.md        # This file
│   ├── PERMISSIONS.md               # Detailed permissions reference
│   │
│   ├── methodologies/               # How agents work
│   │   ├── ralph-standard.md        # Loop-based execution
│   │   ├── interview-protocol.md    # Q&A discovery process
│   │   ├── worktree-agents.md       # Multi-agent role definitions
│   │   └── ux-validation.md         # Visual testing protocol
│   │
│   ├── previous-designs/            # Proven orchestration patterns
│   │   ├── prover-orchestration/    # From Prover project
│   │   ├── brand-extractor-pipeline/
│   │   └── README.md                # Index of patterns
│   │
│   └── references/                  # External knowledge
│       └── llm-orchestration-patterns.md
│
├── settings.json                    # Global settings
└── settings.local.json              # Personal permissions (not committed)
```

---

## Agents

### `/interview` - Discovery Agent
**Purpose:** Clarify requirements before planning through structured Q&A.

**When to use:**
- Requirements are vague ("add authentication")
- Unknown unknowns need surfacing
- Stakeholder has ideas but not specs

**Knowledge:**
- `~/.claude/knowledge/methodologies/interview-protocol.md`

**Capabilities:**
- AskUserQuestion (multi-round Q&A)
- Read, Glob, Grep (codebase exploration)
- Write (.interview-session.json state)

**Handoffs to:** `/ralph`, `/worktree`, `/architect`

---

### `/worktree` - Multi-Agent Orchestrator
**Purpose:** Spawn 2-5 specialized agents (Architect, Detective, Craftsman) for complex features.

**When to use:**
- Complex features with multiple concerns
- Need multiple perspectives (security, UX, architecture)
- Large refactors touching many files

**Knowledge:**
- `~/.claude/knowledge/methodologies/worktree-agents.md`

**Capabilities:**
- Task (spawn subagents)
- Bash(tmux:*), Bash(worktree:*)
- Full read access
- Write (WORKTREE_COORDINATION.md, PLAN.md)

**Handoffs to:** Internal agents (Facilitator, Detective, Craftsman, Aesthete)

---

### `/ralph` - Execution Loop Agent
**Purpose:** Autonomous task execution with feedback loops.

**When to use:**
- Clear requirements, ready to implement
- Bulk work that can run AFK
- Iterative building with quality gates

**Knowledge:**
- `~/.claude/knowledge/methodologies/ralph-standard.md`

**Capabilities:**
- Full read/write/edit
- Bash (build, test, lint, typecheck)
- Git operations

**Stop condition:** `<promise>COMPLETE</promise>` when all PRD items pass.

---

### `/ux-design` - Visual Development Agent
**Purpose:** Visual changes with automatic Playwright validation.

**When to use:**
- Any visual/UI change
- Responsive design work
- Visual bug fixes
- Design replication ("make it look like X")

**Knowledge:**
- `~/.claude/knowledge/methodologies/ux-validation.md`

**Capabilities:**
- MCP Playwright tools
- Screenshot comparison
- Full CSS/component editing
- Browser automation

**Workflow:** Before screenshot → implement → after screenshot → compare → iterate

---

### `/brand-extract` - Brand Analysis Agent
**Purpose:** Extract brand identity from URLs using the brand-extractor project.

**When to use:**
- Building a new branded website
- Need to understand client's visual identity
- Creating brand guidelines from existing sites

**Knowledge:**
- Project at `/home/rancho/brand-extractor/`

**Capabilities:**
- Bash(npm run extract:*)
- Read brand-extractor output
- Can be called as subroutine by other agents

**Output:** JSON brand profile, generated brand book

---

### `/architect` - LLM Orchestration Specialist
**Purpose:** Design and review LLM pipelines, multi-agent systems, orchestration patterns.

**When to use:**
- Designing new AI/LLM features
- Reviewing orchestration architecture
- Comparing approaches (tool calling vs function calling, etc.)
- "How should I structure this AI workflow?"

**Knowledge:**
- `~/.claude/knowledge/previous-designs/` (your proven patterns)
- `~/.claude/knowledge/references/llm-orchestration-patterns.md`
- Access to Prover project patterns

**Capabilities:**
- Full read access (any project)
- WebFetch, WebSearch (research)
- Plan mode (design, don't implement)

**Specialties:**
- Tool calling patterns
- Multi-agent coordination
- State management across context windows
- Prompt chaining and feedback loops
- When to use structured output vs free-form

---

### `/orchestrate` - Meta-Agent
**Purpose:** Route requests to appropriate agents, combine agents for complex tasks.

**When to use:**
- Unclear which agent fits
- Need multiple agents in sequence
- "Help me figure out how to approach this"

**Knowledge:**
- All agent definitions
- `~/.claude/knowledge/previous-designs/`

**Capabilities:**
- Invoke any other agent
- Full read access
- Plan mode by default

**Behavior:**
1. Analyze the request
2. Check if previous designs apply
3. Recommend agent(s)
4. Optionally delegate directly

---

## Permissions Model

### Design Principles

1. **Planning = Read-Only = No Prompts**
   - All exploration tools pre-approved
   - Never interrupt discovery flow

2. **Writing Requires Intent**
   - Edit/Write tools require explicit mode
   - Prevents accidental modifications

3. **Dangerous Commands Require Approval**
   - `rm -rf`, `git push --force`, production access
   - Always prompt, never pre-approve

4. **Agents Inherit + Restrict**
   - All agents get base read permissions
   - Implementation agents get write permissions
   - No agent gets more than base + their needs

### Permission Tiers

#### Tier 0: Always Allowed (No Prompt Ever)
```json
{
  "allow": [
    "Read",
    "Glob",
    "Grep",
    "WebFetch",
    "WebSearch",
    "TodoWrite",
    "Task(Explore)",
    "Task(Plan)",
    "Bash(ls:*)",
    "Bash(find:*)",
    "Bash(cat:*)",
    "Bash(head:*)",
    "Bash(tail:*)",
    "Bash(grep:*)",
    "Bash(rg:*)",
    "Bash(tree:*)",
    "Bash(wc:*)",
    "Bash(file:*)",
    "Bash(stat:*)",
    "Bash(du:*)",
    "Bash(df:*)",
    "Bash(pwd:*)",
    "Bash(which:*)",
    "Bash(whereis:*)",
    "Bash(type:*)",
    "Bash(echo:*)",
    "Bash(printf:*)",
    "Bash(date:*)",
    "Bash(whoami:*)",
    "Bash(hostname:*)",
    "Bash(uname:*)",
    "Bash(env:*)",
    "Bash(printenv:*)",
    "Bash(git status:*)",
    "Bash(git log:*)",
    "Bash(git diff:*)",
    "Bash(git show:*)",
    "Bash(git branch:*)",
    "Bash(git remote:*)",
    "Bash(git config --get:*)",
    "Bash(git rev-parse:*)",
    "Bash(jq:*)",
    "Bash(yq:*)",
    "Bash(sort:*)",
    "Bash(uniq:*)",
    "Bash(cut:*)",
    "Bash(awk:*)",
    "Bash(sed -n:*)",
    "Bash(diff:*)",
    "Bash(comm:*)"
  ]
}
```

#### Tier 1: Implementation Mode (Pre-Approved for Agents)
```json
{
  "allow": [
    "Edit",
    "Write",
    "NotebookEdit",
    "Bash(mkdir:*)",
    "Bash(touch:*)",
    "Bash(cp:*)",
    "Bash(mv:*)",
    "Bash(git add:*)",
    "Bash(git commit:*)",
    "Bash(git checkout:*)",
    "Bash(git switch:*)",
    "Bash(git stash:*)",
    "Bash(npm:*)",
    "Bash(npx:*)",
    "Bash(yarn:*)",
    "Bash(pnpm:*)",
    "Bash(bun:*)",
    "Bash(pip:*)",
    "Bash(python:*)",
    "Bash(cargo:*)",
    "Bash(go:*)",
    "Bash(make:*)",
    "Bash(docker:*)",
    "Bash(docker-compose:*)",
    "Bash(curl:*)",
    "Bash(wget:*)"
  ]
}
```

#### Tier 2: Ask Every Time
```json
{
  "ask": [
    "Bash(git push:*)",
    "Bash(git merge:*)",
    "Bash(git rebase:*)",
    "Bash(npm publish:*)",
    "Bash(docker push:*)"
  ]
}
```

#### Tier 3: Always Deny
```json
{
  "deny": [
    "Bash(rm -rf /:*)",
    "Bash(rm -rf ~:*)",
    "Bash(rm -rf /*:*)",
    "Bash(> /dev/sda:*)",
    "Bash(dd if=:*)",
    "Bash(mkfs:*)",
    "Bash(:(){ :|:& };::*)",
    "Bash(chmod -R 777 /:*)",
    "Bash(git push --force origin main:*)",
    "Bash(git push --force origin master:*)",
    "Bash(DROP DATABASE:*)",
    "Bash(DROP TABLE:*)",
    "Read(//**/.*credentials*)",
    "Read(//**/.*secrets*)",
    "Read(//**/.env.production)"
  ]
}
```

### Agent-Specific Permissions

| Agent | Tier 0 | Tier 1 | Special |
|-------|--------|--------|---------|
| `/interview` | ✓ | - | AskUserQuestion |
| `/worktree` | ✓ | ✓ | Bash(tmux:*), Bash(worktree:*) |
| `/ralph` | ✓ | ✓ | Full implementation |
| `/ux-design` | ✓ | ✓ | MCP Playwright |
| `/brand-extract` | ✓ | - | Bash(npm run extract:*) |
| `/architect` | ✓ | - | Plan mode only |
| `/orchestrate` | ✓ | - | Task(*) to invoke agents |

---

## Master Permissions Config

This goes in `~/.claude/settings.local.json`:

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "WebFetch",
      "WebSearch",
      "TodoWrite",
      "Edit",
      "Write",
      "NotebookEdit",
      "Task(*)",
      "Skill(*)",

      "Bash(ls:*)",
      "Bash(find:*)",
      "Bash(cat:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(grep:*)",
      "Bash(rg:*)",
      "Bash(tree:*)",
      "Bash(wc:*)",
      "Bash(file:*)",
      "Bash(stat:*)",
      "Bash(du:*)",
      "Bash(df:*)",
      "Bash(pwd:*)",
      "Bash(which:*)",
      "Bash(whereis:*)",
      "Bash(type:*)",
      "Bash(echo:*)",
      "Bash(printf:*)",
      "Bash(date:*)",
      "Bash(whoami:*)",
      "Bash(hostname:*)",
      "Bash(uname:*)",
      "Bash(env:*)",
      "Bash(printenv:*)",
      "Bash(jq:*)",
      "Bash(yq:*)",
      "Bash(sort:*)",
      "Bash(uniq:*)",
      "Bash(cut:*)",
      "Bash(awk:*)",
      "Bash(sed:*)",
      "Bash(diff:*)",
      "Bash(comm:*)",
      "Bash(xargs:*)",

      "Bash(git:*)",
      "Bash(gh:*)",

      "Bash(mkdir:*)",
      "Bash(touch:*)",
      "Bash(cp:*)",
      "Bash(mv:*)",
      "Bash(rm:*)",
      "Bash(chmod:*)",
      "Bash(chown:*)",

      "Bash(npm:*)",
      "Bash(npx:*)",
      "Bash(yarn:*)",
      "Bash(pnpm:*)",
      "Bash(bun:*)",
      "Bash(~/.bun/bin/bun:*)",
      "Bash(pip:*)",
      "Bash(pip3:*)",
      "Bash(python:*)",
      "Bash(python3:*)",
      "Bash(cargo:*)",
      "Bash(go:*)",
      "Bash(make:*)",
      "Bash(cmake:*)",

      "Bash(docker:*)",
      "Bash(docker-compose:*)",
      "Bash(docker compose:*)",

      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(nc:*)",
      "Bash(netstat:*)",
      "Bash(lsof:*)",
      "Bash(ss:*)",

      "Bash(tmux:*)",
      "Bash(screen:*)",
      "Bash(worktree:*)",

      "Bash(sleep:*)",
      "Bash(timeout:*)",
      "Bash(clear:*)",
      "Bash(tee:*)",
      "Bash(true:*)",
      "Bash(false:*)",
      "Bash(test:*)",
      "Bash([:*)",

      "mcp__playwright__*",
      "mcp__plugin_gws_google-workspace__*"
    ],
    "deny": [
      "Bash(rm -rf /:*)",
      "Bash(rm -rf ~:*)",
      "Bash(rm -rf /*:*)",
      "Bash(> /dev/sd:*)",
      "Bash(dd if=/dev/zero:*)",
      "Bash(mkfs:*)",
      "Bash(:(){ :|:& };::*)",
      "Bash(chmod -R 777 /:*)",
      "Bash(git push --force origin main:*)",
      "Bash(git push --force origin master:*)"
    ]
  }
}
```

---

## Usage Examples

### Natural Invocation
```
User: "interview me about the auth feature"
Claude: [invokes /interview agent]

User: "ralph, implement the items in prd.json"
Claude: [invokes /ralph agent, starts loop]

User: "architect, review this orchestration design"
Claude: [invokes /architect agent, reads and analyzes]

User: "ux-design, make the header sticky with a shadow on scroll"
Claude: [invokes /ux-design, captures before, implements, validates]

User: "orchestrate: I need to build a branded landing page for client X"
Claude: [invokes /orchestrate]
  → Recommends: /brand-extract first, then /interview for requirements,
     then /worktree for implementation with UX agent involvement
```

### Chained Workflows
```
/interview "new payment system"
  → generates requirements
  → hands off to /architect for design review
  → hands off to /worktree for implementation

/orchestrate "migrate from REST to GraphQL"
  → analyzes scope
  → recommends /interview (clarify scope)
  → then /worktree (multi-agent implementation)
  → with /architect oversight
```

---

## Implementation Checklist

- [ ] Create agent command files in `~/.claude/commands/`
- [ ] Move methodologies to `~/.claude/knowledge/methodologies/`
- [ ] Extract orchestration patterns from Prover to `previous-designs/`
- [ ] Update `settings.local.json` with master permissions
- [ ] Test each agent independently
- [ ] Test agent handoffs
- [ ] Document in this file as patterns emerge

---

## Notes

- **Agents are prompts, not code.** Each agent is a markdown file that sets context.
- **Knowledge is shared.** All agents can read from `~/.claude/knowledge/`.
- **Permissions are global.** Can't scope permissions per-agent (Claude Code limitation).
- **Hooks can add context-aware control** if needed later.
