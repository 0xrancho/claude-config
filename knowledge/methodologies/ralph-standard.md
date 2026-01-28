---
name: ralph-wiggum
trigger_words: [ralph, blocks, sprint, iterative, kanban, loop]
description: Single-agent iterative sprint-style planning. Breaks projects into testable blocks with completion promises and bash loops. For multi-agent collaboration, use `/plan --worktree` instead.
---

# Ralph Wiggum Method

*"Me fail English? That's unpossible!"* - But failing a sprint block? Totally possible. That's why we iterate.

## The Problem This Solves

Monolithic chronological plans fail for complex local applications:
- **Stale** - Implementation reveals unknowns that invalidate the plan
- **Untrackable** - Hard to see what's actually done vs planned
- **No criteria** - No clear definition of "done" per task
- **No iteration** - No testing/feedback built into workflow

## The Ralph Wiggum Approach

An iterative sprint-style method where:
1. Projects break into discrete, testable **blocks** (not sequential phases)
2. Each block has a **completion promise** (exact success criteria)
3. Blocks are picked **asynchronously** (Kanban-style board)
4. Each block cycles: **do work → test → iterate → check off**
5. **Max iterations** prevent infinite spinning

## Block Structure

```yaml
block:
  id: unique-id                           # kebab-case identifier
  title: "Short descriptive title"        # Human-readable name
  description: "What this block accomplishes"
  completion_promise: "Exact condition that means DONE"
  max_iterations: 10                      # Default: 10
  dependencies: [other-block-ids]         # Optional: blocks that must complete first
  test_command: "command to verify"       # Optional: automated verification
  notes: []                               # Runtime notes from iterations
```

## Board States

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   BACKLOG   │───▶│ IN_PROGRESS │───▶│   TESTING   │───▶│    DONE     │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                          │                  │
                          │                  │
                          ▼                  ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   BLOCKED   │    │   BLOCKED   │
                   └─────────────┘    └─────────────┘
```

| State | Meaning |
|-------|---------|
| `backlog` | Not started, waiting to be picked |
| `in_progress` | Currently being worked on |
| `testing` | Work complete, verifying completion promise |
| `blocked` | Waiting on dependency, user input, or hit max iterations |
| `done` | Completion promise verified and met |

## Loop Behavior

Each iteration within a block:

```
┌─────────────────────────────────────────────────────────┐
│  ITERATION n of max_iterations                          │
├─────────────────────────────────────────────────────────┤
│  1. CHECK completion_promise                            │
│     └─▶ If met: mark DONE, exit loop                    │
│                                                         │
│  2. DO incremental work toward goal                     │
│     └─▶ Make smallest meaningful progress               │
│                                                         │
│  3. TEST if test_command defined                        │
│     └─▶ Run verification, capture output                │
│                                                         │
│  4. REPORT progress                                     │
│     └─▶ Add note about what was tried/learned           │
│                                                         │
│  5. If n >= max_iterations:                             │
│     └─▶ Mark BLOCKED with reason, exit loop             │
└─────────────────────────────────────────────────────────┘
```

## Board File Format

Store state in `.ralph-board.json` (project root) or `/tmp/ralph-board-{project}.json`:

```json
{
  "project": "project-name",
  "created": "2024-01-15T10:00:00Z",
  "updated": "2024-01-15T14:30:00Z",
  "blocks": [
    {
      "id": "tmux-session",
      "title": "Create tmux session with split panes",
      "description": "Set up the base tmux environment",
      "completion_promise": "gws-canvas command creates session with 2 panes",
      "test_command": "tmux list-panes -t gws | wc -l",
      "max_iterations": 5,
      "dependencies": [],
      "state": "done",
      "iteration": 3,
      "notes": [
        "Iter 1: Created basic session, but panes not splitting",
        "Iter 2: Fixed split command syntax",
        "Iter 3: Verified 2 panes exist, completion promise met"
      ]
    },
    {
      "id": "canvas-spawn",
      "title": "Canvas spawns to correct pane",
      "description": "Launch canvas renderer in the designated pane",
      "completion_promise": "bun canvas command renders in pane 0",
      "max_iterations": 10,
      "dependencies": ["tmux-session"],
      "state": "in_progress",
      "iteration": 2,
      "notes": [
        "Iter 1: Canvas runs but in wrong pane",
        "Iter 2: Trying send-keys approach..."
      ]
    }
  ]
}
```

## Commands

### `ralph-plan "project description"`

Generate initial blocks from a project description.

**Workflow:**
1. Analyze the project description
2. Identify discrete, testable work units
3. Determine dependencies between units
4. Generate blocks with completion promises
5. Create board file with all blocks in `backlog`

**Output:** Board file created, summary of blocks shown

---

### `ralph-board`

Show current board state in visual format.

**Output:**
```
╔══════════════════════════════════════════════════════════════════╗
║  RALPH BOARD: gws-canvas                                         ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  BACKLOG          IN_PROGRESS       TESTING          DONE        ║
║  ─────────────    ─────────────     ─────────────    ─────────── ║
║  [error-handle]   [canvas-spawn]                     [tmux-sess] ║
║  [cleanup]        iter 2/10                          iter 3/5    ║
║                                                                  ║
║  BLOCKED                                                         ║
║  ─────────────                                                   ║
║  [ipc-comm] reason: need canvas working first                    ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

### `ralph-pick [block-id]`

Start working on a block. Moves it to `in_progress`.

**Workflow:**
1. Verify block exists and is in `backlog`
2. Check all dependencies are `done`
3. Move block to `in_progress`
4. Set iteration to 0
5. Display block details and completion promise

**Errors:**
- Block not found
- Block not in backlog
- Dependencies not met (list which ones)

---

### `ralph-loop "task" --completion-promise "X" [--max-iterations N]`

Run the iteration loop on current in_progress block (or create ad-hoc block).

**Workflow:**
```
for i in 1..max_iterations:
    if completion_promise is met:
        mark block DONE
        return SUCCESS

    do_incremental_work()

    if test_command:
        run_test()

    add_iteration_note()
    report_progress()

mark block BLOCKED("max iterations reached")
return BLOCKED
```

**Flags:**
- `--completion-promise "X"` - Required. The exact condition for done.
- `--max-iterations N` - Optional. Default: 10.
- `--test-command "cmd"` - Optional. Verification command.

---

### `ralph-done [block-id]`

Mark a block complete after verifying completion promise.

**Workflow:**
1. Run test_command if defined
2. Prompt for completion promise verification
3. If verified, move to `done`
4. If not verified, keep in current state with note

---

### `ralph-blocked [block-id] "reason"`

Mark a block as blocked with explanation.

**Workflow:**
1. Move block to `blocked` state
2. Store reason in notes
3. Show which blocks depend on this one (will also be blocked)

---

## Key Principles

### 1. Small Blocks
Each block should be completable in one focused session. If a block feels too big, split it.

**Too big:** "Implement authentication system"
**Right size:** "Add login form", "Create session middleware", "Add logout endpoint"

### 2. Testable
Every block has verification criteria. Prefer automated `test_command` but accept manual verification.

**Bad promise:** "Auth works"
**Good promise:** "POST /login with valid creds returns 200 and session cookie"

### 3. Independent
Minimize dependencies between blocks. Blocks should be pickable in (mostly) any order.

### 4. Iterative
Expect to loop. The first attempt rarely works. Each iteration should make progress or learn something.

### 5. Fail Fast
Max iterations prevent infinite spinning. If stuck after N tries, get help or rethink the approach.

### 6. State Persisted
Board survives context switches. Pick up where you left off.

---

## Example: GWS-Canvas Project

```yaml
blocks:
  - id: tmux-session
    title: "Create tmux session with split panes"
    completion_promise: "gws-canvas command creates session with 2 panes"
    test_command: "tmux list-panes -t gws | wc -l"
    max_iterations: 5
    dependencies: []

  - id: canvas-spawn
    title: "Canvas spawns to correct pane"
    completion_promise: "bun canvas command renders in pane 0"
    dependencies: [tmux-session]
    max_iterations: 10

  - id: ipc-communication
    title: "IPC between Claude and canvas"
    completion_promise: "Selection from canvas received by controller"
    dependencies: [canvas-spawn]
    max_iterations: 10

  - id: selection-highlight
    title: "Visual selection feedback"
    completion_promise: "Selected item shows highlight state in canvas"
    dependencies: [ipc-communication]
    max_iterations: 8

  - id: error-handling
    title: "Graceful error states"
    completion_promise: "Canvas shows error message when IPC fails"
    dependencies: [ipc-communication]
    max_iterations: 5
```

---

## When to Use Ralph Wiggum

**Good fit:**
- Building local applications with multiple components
- Integration work where pieces must work together
- Debugging complex systems
- Projects where you expect to iterate

**Poor fit:**
- Simple single-file changes
- Documentation writing
- One-shot scripts
- Pure research/exploration

---

## TUI Script

A `gum`-based TUI is available at `~/.claude/skills/plan/bin/ralph`:

```bash
# Add to PATH (add to .bashrc/.zshrc)
export PATH="$HOME/.claude/skills/plan/bin:$PATH"

# Or run directly
~/.claude/skills/plan/bin/ralph board
```

**Commands:**
```
ralph board          # Show the board
ralph init [name]    # Initialize new board
ralph add            # Add a block interactively
ralph pick           # Pick a block to work on
ralph iterate [msg]  # Record an iteration
ralph done [id]      # Mark block as done
ralph blocked [id]   # Mark block as blocked
ralph show [id]      # Show block details
```

**Requirements:** `gum` (charm.sh), `jq`

```bash
# Install on macOS
brew install gum jq

# Install on Linux
# gum: https://github.com/charmbracelet/gum#installation
# jq: apt install jq / dnf install jq
```

---

## Integration with TodoWrite

When working on a block, use Claude's `TodoWrite` tool to track micro-tasks within the iteration:

```
Block: canvas-spawn (iteration 3)
├── [x] Check why send-keys not targeting pane
├── [x] Try select-pane before send-keys
├── [ ] Verify canvas renders
└── [ ] Check completion promise
```

The block is the strategic unit. Todos are tactical within each iteration.

---

## Multi-Agent Alternative: Worktree

For complex features needing **multiple perspectives**, use `/plan --worktree` instead:

| Aspect | Ralph (this method) | Worktree |
|--------|---------------------|----------|
| Agents | 1 (single Claude) | 2-5 specialized |
| Perspective | Your questions only | Detective, Architect, Aesthete, etc. |
| Iteration | Bash loop | Ralph loops per agent |
| Best for | Focused tasks | Complex features |

**When to use worktree instead:**
- Security-sensitive changes (Detective asks "What could go wrong?")
- Architectural decisions (Architect asks "How does this scale?")
- Large refactors (multiple perspectives catch more issues)
- When you want questions you wouldn't think to ask

```bash
# Single-agent Ralph (this method)
/plan --ralph

# Multi-agent Worktree (spawns specialized agents)
/plan --worktree
```

Worktree agents use Ralph internally - each agent runs Ralph loops on their task board.
