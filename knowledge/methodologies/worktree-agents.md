---
name: worktree-multi-agent
trigger_words: [worktree, multi-agent, agents, team, parallel, swarm]
description: Multi-agent planning and implementation using worktree CLI. Spawns 2-5 specialized Claude agents (Architect, Detective, Craftsman, etc.) that collaborate via shared docs. Each agent runs Ralph loops internally. Use for complex features needing multiple perspectives.
---

# Worktree Multi-Agent Method

*"None of us is as smart as all of us."* - Multiple specialized agents catch blind spots you'd never find alone.

## The Problem This Solves

Single-agent planning has blind spots:
- **One perspective** - You only ask the questions YOU think of
- **Sequential thinking** - Can't explore multiple approaches simultaneously
- **Role confusion** - Same agent tries to be architect AND security auditor AND code reviewer
- **Context limits** - Complex features exceed single-agent context

## The Worktree Approach

Spawn **2-5 specialized Claude agents** that collaborate through shared documents:

| Agent | Role | Questions They Ask |
|-------|------|-------------------|
| **Facilitator** | Synthesize, coordinate, decide | "Do we have consensus?" |
| **Architect** | System design, scalability | "How should components interact?" |
| **Detective** | Security, edge cases, risks | "What could go wrong?" |
| **Craftsman** | Code quality, testability | "Is this maintainable?" |
| **Aesthete** | Simplicity, elegance | "Can this be simpler?" |
| **Explorer** | Alternatives, innovation | "What if we tried X instead?" |

Each agent runs **Ralph loops internally** for their tasks.

## Two Modes

### Planning Mode (`--plan`)
Agents debate and critique BEFORE implementation:
```bash
worktree open <id> "<desc>" --local --plan -w 4
```
- Focus: Analysis, design, risk identification
- Output: PLAN.md with tasks, risks, decisions
- Completion: `<promise>Plan approved</promise>`

### Implementation Mode (default)
Agents implement WITH Ralph methodology:
```bash
worktree open <id> "<desc>" --local -w 3
```
- Focus: Writing code, tests, docs
- Output: Working implementation
- Completion: `<promise>Worker N tasks complete, all tests pass</promise>`

## Workflow

### Phase 1: Planning Session

```
/plan with worktree

Description: Add user authentication with OAuth

→ Spawns planning agents:
  • Facilitator: Coordinates discussion
  • Detective: "What happens if token expires mid-session?"
  • Architect: "Should we use middleware or decorators?"
  • Aesthete: "Can we simplify the token refresh flow?"

→ Agents collaborate in WORKTREE_COORDINATION.md
→ Output: PLAN.md with consensus approach
```

### Phase 2: Implementation Session

```
Execute the plan with worktree

→ Spawns implementation agents:
  • Coordinator: Leads implementation, assigns tasks
  • Detective: Writes tests, finds edge cases
  • Craftsman: Ensures code quality

→ Each agent runs Ralph loops on their task board
→ Quality gates enforced before commits
→ Output: Working code with tests
```

## Commands

### Start Planning Session
```bash
# Via CLI
worktree open 1 "Add OAuth authentication" --local --plan -w 4

# Attach to watch
tmux attach -t <session>
```

### Check Progress
```bash
worktree status 1
```

### Start Implementation
```bash
# Close planning session first
worktree rm 1

# Start implementation
worktree open 1 "Add OAuth authentication" --local -w 3
```

### Send Iteration Nudge
```bash
worktree ralph 1
```

## File Structure

When worktree runs, it creates:

```
project/
├── CLAUDE.md                    # Issue context + Ralph quality gates
├── WORKTREE_COORDINATION.md     # Worker task boards + communication
├── OVERSEER.md                  # (if --watcher) Progress monitoring
└── PLAN.md                      # (planning mode) Final plan document
```

## Agent Prompts

### Planning Mode Prompts

**Facilitator:**
> Synthesize input from all workers. Identify conflicts. Create unified PLAN.md.
> When all workers approve: `<promise>Plan complete</promise>`

**Detective:**
> What could go wrong? Security holes? Edge cases? Failure modes?
> Challenge assumptions. Find risks others miss.

**Architect:**
> How should components be structured? What interfaces? How does this scale?
> Think about the system, not just the feature.

**Aesthete:**
> Can this be simpler? What can we remove? Is the API intuitive?
> Fight unnecessary complexity.

### Implementation Mode Prompts

**Coordinator:**
> Break down tasks. Assign to worker boards. Implement core logic.
> Run quality gates before every commit.

**Detective:**
> Write comprehensive tests. Find edge cases. Verify security.
> Nothing ships without your approval.

**Craftsman:**
> Ensure code quality. Refactor for clarity. Document decisions.
> Leave the codebase better than you found it.

## When to Use Worktree

**Good fit:**
- Complex features with multiple concerns (auth, API, UI, tests)
- Security-sensitive changes needing review
- Architectural decisions with trade-offs
- Refactoring large modules
- When you want perspectives you wouldn't think of

**Poor fit:**
- Simple bug fixes
- Single-file changes
- Quick scripts
- When you already know exactly what to do

## Comparison: Ralph vs Worktree

| Aspect | `/plan --ralph` | `/plan --worktree` |
|--------|-----------------|-------------------|
| Agents | 1 (you + Claude) | 2-5 specialized |
| Perspective | Single | Multiple archetypes |
| Iteration | Ralph loops | Ralph loops per agent |
| Coordination | Your brain | WORKTREE_COORDINATION.md |
| Best for | Focused tasks | Complex features |

## Integration with Ralph

Worktree **includes** Ralph - it's not either/or:

```
worktree spawns agents
    │
    ├── Agent 1 (Coordinator)
    │   └── Runs Ralph loop on Worker 1 Board
    │
    ├── Agent 2 (Detective)
    │   └── Runs Ralph loop on Worker 2 Board
    │
    └── Agent 3 (Craftsman)
        └── Runs Ralph loop on Worker 3 Board
```

Each agent:
1. Picks tasks from their board
2. Implements with quality gates
3. Marks done when complete
4. Outputs completion promise

## Prerequisites

```bash
# Install worktree CLI (already at ~/worktree-linux)
cd ~/worktree-linux && npm link

# Required tools
sudo apt install tmux jq gum  # Linux
brew install tmux jq gum      # macOS

# Verify
worktree --help
```

## Quick Reference

```bash
# Planning with 4 agents
worktree open 1 "Feature X" --local --plan -w 4

# Implementation with 3 agents + overseer
worktree open 1 "Feature X" --local -w 3 --watcher

# Check status
worktree status 1

# Send iteration prompt
worktree ralph 1

# Clean up
worktree rm 1

# Attach to tmux
tmux attach -t <project>_workers
```

---

## Example Session

```
User: /plan with worktree

Description: Add real-time notifications using WebSockets

Claude: Starting worktree planning session...

📋 PLANNING MODE - 4 agents analyzing your feature

Workers:
  • Worker 1 (Facilitator) - Coordinating discussion
  • Worker 2 (Detective) - Finding risks: "What if connection drops?"
  • Worker 3 (Architect) - Designing: "Socket.io vs native WS?"
  • Worker 4 (Aesthete) - Simplifying: "Do we need all these events?"

Attach to watch: tmux attach -t myproject_workers

When planning completes, run:
  /execute with worktree
```
