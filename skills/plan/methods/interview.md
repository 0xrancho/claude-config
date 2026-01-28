---
name: interview
trigger_words: [interview, discover, discovery, clarify, questions, understand, requirements, ask]
description: AI-guided discovery through structured interviews. Runs 2-3 rounds of clarifying questions before planning. Use when requirements are unclear or you want the AI to surface questions you wouldn't think to ask.
---

# Interview Method

*"The quality of your plan depends on the quality of your questions."* - Discovery before planning prevents building the wrong thing.

## The Problem This Solves

Jumping straight to planning fails when:
- **Unclear requirements** - You have an idea but haven't thought through details
- **Unknown unknowns** - There are questions you don't know to ask
- **Assumption blindness** - You assume things that should be validated
- **Scope creep risk** - Without clear boundaries, features expand endlessly

## The Interview Approach

An AI-guided discovery process where:
1. You provide an initial task/feature description
2. AI generates **targeted multiple-choice questions** to clarify requirements
3. Each round builds on previous answers, going deeper
4. You can pick options, write custom answers, or skip
5. Final output: **comprehensive plan informed by the Q&A session**

```
┌─────────────────────────────────────────────────────────────────┐
│                    INTERVIEW WORKFLOW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT ──▶ ROUND 1 ──▶ ROUND 2 ──▶ ROUND 3 ──▶ PLAN             │
│  Task      Scope &     UX &        Technical    Actionable      │
│  Idea      Context     Approach    Details      Implementation  │
│                                                                  │
│  Each round:                                                     │
│  • 3-5 multiple choice questions                                 │
│  • Options: [1-4] Pick  |  [0] Custom  |  [s] Skip              │
│  • Context accumulates for smarter follow-ups                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Round Structure

### Round 1: Scope & Context
Focus: Understanding the big picture
- What problem are you solving?
- Who is the target user?
- What are the key constraints?
- What exists already vs needs building?
- What's the success criteria?

### Round 2: Approach & Experience
Focus: How it should work
- What's the user flow?
- What are the edge cases?
- How should errors be handled?
- What's the priority order?
- What can be deferred?

### Round 3: Technical Details
Focus: Implementation specifics
- What architecture patterns fit?
- What dependencies are needed?
- What's the testing strategy?
- What are the security considerations?
- What's the rollout approach?

## Question Format

Each question is presented as:

```
[2/5] What level of error handling do you need?

  1) Basic - console.log errors, fail fast
  2) Standard - try/catch with user-friendly messages
  3) Robust - retry logic, graceful degradation, error boundaries
  4) Enterprise - centralized error tracking, alerting, recovery

  ─────────────────────────────
  0) Write your own answer
  s) Skip this question
```

## State Storage

Interview state is stored in `.interview-session.json`:

```json
{
  "id": "a1b2c3d4",
  "task": "Add user authentication with OAuth",
  "created": "2024-01-15T10:00:00Z",
  "updated": "2024-01-15T10:15:00Z",
  "status": "round_2",
  "config": {
    "questionsPerRound": 4,
    "choicesPerQuestion": 4,
    "totalRounds": 3
  },
  "rounds": [
    {
      "number": 1,
      "focus": "Scope & Context",
      "questions": [
        {
          "question": "What OAuth providers do you need to support?",
          "choices": [
            "Google only",
            "Google and GitHub",
            "Multiple providers (Google, GitHub, Microsoft, etc.)",
            "Custom OAuth provider"
          ],
          "answer": "Google and GitHub",
          "skipped": false
        }
      ],
      "completed": "2024-01-15T10:05:00Z"
    }
  ],
  "plan": null
}
```

## Commands

### `interview-start "task description"`

Begin a new interview session.

**Workflow:**
1. Create session file with task description
2. Generate Round 1 questions based on task
3. Present questions interactively
4. Save answers and proceed to Round 2

**Example:**
```
/plan interview "Add real-time notifications to the dashboard"

Starting interview session...

Round 1: Scope & Context

[1/4] What type of notifications do you need?
  1) In-app only (toast/banner)
  2) In-app + browser push notifications
  3) In-app + email notifications
  4) Full omnichannel (in-app, push, email, SMS)

> 2

[2/4] How real-time do they need to be?
...
```

---

### `interview-resume`

Continue a paused interview session.

**Workflow:**
1. Load session from `.interview-session.json`
2. Resume at current round/question
3. Continue interactive flow

---

### `interview-status`

Show current interview progress.

**Output:**
```
╔══════════════════════════════════════════════════════════════════╗
║  INTERVIEW: Add real-time notifications                          ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Status: Round 2 of 3                                            ║
║  Questions answered: 7/12                                        ║
║                                                                  ║
║  Round 1 (Scope & Context)     ✓ Complete                        ║
║  Round 2 (Approach & UX)       ◐ In Progress (3/4)               ║
║  Round 3 (Technical Details)   ○ Pending                         ║
║                                                                  ║
║  Key decisions so far:                                           ║
║  • In-app + push notifications                                   ║
║  • WebSocket-based, <1s latency                                  ║
║  • Priority: mentions, then activity, then system                ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

### `interview-generate`

Generate the plan from completed interview.

**Workflow:**
1. Compile all Q&A into context
2. Generate comprehensive implementation plan
3. Output to `PLAN.md` or specified file
4. Optionally convert to Ralph blocks

**Output includes:**
- Executive summary
- Requirements (from Q&A)
- Technical approach
- Implementation steps
- Testing strategy
- Risk considerations

---

### `interview-to-ralph`

Convert interview results to Ralph blocks.

**Workflow:**
1. Read completed interview
2. Break down plan into discrete blocks
3. Generate completion promises from requirements
4. Create `.ralph-board.json`

**Example:**
```
Converting interview to Ralph blocks...

Created 6 blocks:
  • ws-connection - "WebSocket connects and stays alive"
  • notification-api - "POST /notify triggers real-time update"
  • toast-component - "Notification appears within 1s of trigger"
  • push-integration - "Browser push notification appears when tab unfocused"
  • preference-settings - "User can toggle notification types"
  • notification-history - "User can view past 50 notifications"

Board saved to .ralph-board.json
Run `ralph-board` to view
```

---

## AskUserQuestion Integration

When running inside Claude Code, interview questions use the `AskUserQuestion` tool:

```typescript
// Round 1, Question 1
AskUserQuestion({
  questions: [{
    question: "What OAuth providers do you need to support?",
    header: "OAuth",
    options: [
      { label: "Google only", description: "Single provider, simplest setup" },
      { label: "Google + GitHub", description: "Common combo for dev tools" },
      { label: "Multiple providers", description: "Google, GitHub, Microsoft, etc." },
      { label: "Custom provider", description: "Your own OAuth server" }
    ],
    multiSelect: false
  }]
})
```

## Prompt Templates

### Question Generation Prompt

```
You are conducting a requirements interview for a software task.

Task: "${task}"
Round: ${roundNumber} of 3
Focus: ${roundFocus}

${previousQA ? `
Previous Q&A:
${previousQA.map(qa => `Q: ${qa.question}\nA: ${qa.answer}`).join('\n\n')}
` : ''}

Generate ${questionsCount} multiple-choice questions to clarify requirements.
Each question should have ${choicesCount} distinct, meaningful options.

Round focus:
- Round 1: Scope, context, constraints, users, success criteria
- Round 2: User experience, workflows, edge cases, priorities
- Round 3: Technical implementation, architecture, testing, security

Format:
<questions>
<question>
<text>Your question here</text>
<choices>
<choice>Option 1</choice>
<choice>Option 2</choice>
<choice>Option 3</choice>
<choice>Option 4</choice>
</choices>
</question>
</questions>
```

### Plan Generation Prompt

```
Based on this requirements interview, create a comprehensive implementation plan.

Task: "${task}"

Interview Q&A:
${allQA.map(qa => `Q: ${qa.question}\nA: ${qa.answer}`).join('\n\n')}

Create a plan that includes:
1. Executive Summary (2-3 sentences)
2. Requirements (extracted from answers)
3. Technical Approach
4. Implementation Steps (ordered)
5. Testing Strategy
6. Risks and Mitigations
7. Out of Scope (explicitly deferred items)

Be specific and actionable. Reference the interview answers to justify decisions.
```

## Key Principles

### 1. Questions Before Answers
Don't assume requirements. Ask. The interview surfaces things you wouldn't think to specify.

### 2. Multiple Choice with Escape Hatches
Options guide thinking but don't constrain. Custom answers and skip are always available.

### 3. Progressive Depth
Round 1 is broad, Round 3 is specific. Each round builds on previous context.

### 4. Decisions Are Documented
Every answer is saved. The plan traces back to specific Q&A. No mystery about why something was chosen.

### 5. Interruptible
Session saves after each answer. Resume anytime. Don't lose progress.

## When to Use Interview

**Good fit:**
- New features with unclear requirements
- Working with stakeholders who have ideas but not specs
- Projects where you want AI to surface questions
- When you want documented requirement rationale
- Before starting Ralph blocks (interview → ralph)

**Poor fit:**
- Bug fixes with clear reproduction steps
- Tasks you've done many times before
- When requirements are already documented
- Quick scripts or one-off changes

## Comparison: Interview vs Ralph vs Worktree

| Aspect | Interview | Ralph | Worktree |
|--------|-----------|-------|----------|
| **Phase** | Discovery | Execution | Parallel execution |
| **Input** | Vague idea | Clear requirements | Complex feature |
| **Output** | Detailed plan | Working code | Working code |
| **Questions** | AI asks YOU | You check completion | Agents debate |
| **Best for** | Unclear scope | Iterative building | Multiple concerns |

## Workflow Combinations

### Interview → Ralph
Most common. Discover requirements, then execute.

```bash
/plan interview "Add payment processing"
# ... answer questions ...
# Plan generated

interview-to-ralph
# Blocks created from plan

ralph-board
# Start execution
```

### Interview → Worktree
For complex features needing multiple agents.

```bash
/plan interview "Redesign authentication system"
# ... answer questions ...
# Plan generated

/plan worktree
# Use plan as input for multi-agent session
```

## Quick Reference

```bash
# Start new interview
/plan interview "task description"

# Resume interrupted interview
interview-resume

# Check progress
interview-status

# Generate plan (after all rounds)
interview-generate

# Convert to Ralph blocks
interview-to-ralph

# Quick interview (fewer questions)
/plan interview "task" --quick    # 2 questions per round

# Deep interview (more questions)
/plan interview "task" --deep     # 5 questions per round
```

---

## Example Session

```
User: /plan interview "Add dark mode to the application"

Claude: Starting interview session for: "Add dark mode to the application"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ROUND 1: Scope & Context
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[1/4] How should users control dark mode?

  1) System preference only (auto-detect OS setting)
  2) Manual toggle only (user choice in settings)
  3) Both - default to system, allow override
  4) Time-based - dark at night, light during day

> 3

[2/4] What's the scope of theming?

  1) Colors only (background, text, borders)
  2) Colors + shadows/elevation adjustments
  3) Full theme (colors, typography, spacing, icons)
  4) Multiple themes (dark, light, high contrast, custom)

> 2

[3/4] How is styling currently implemented?

  1) CSS variables (custom properties)
  2) CSS-in-JS (styled-components, emotion)
  3) Tailwind CSS
  4) Plain CSS/SCSS with class-based theming

> 3

[4/4] What's the timeline constraint?

  1) Quick win - ship in a day
  2) Standard - 2-3 days with polish
  3) Thorough - 1 week with full testing
  4) No rush - get it right

> 2

✓ Round 1 complete

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ROUND 2: Approach & Experience
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[1/4] How should the theme transition?

  1) Instant switch (no animation)
  2) Quick fade (150ms opacity transition)
  3) Smooth transition (300ms on all colors)
  4) Fancy (animated gradient sweep)

> 2

...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 INTERVIEW COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Generating plan from 12 answered questions...

📄 Plan saved to PLAN.md

Summary:
• Dark mode with system detection + manual override
• Tailwind-based implementation using CSS variables
• Quick fade transitions, persist to localStorage
• 2-3 day timeline with component-by-component rollout

Next steps:
  • Review PLAN.md
  • Run `interview-to-ralph` to create execution blocks
  • Or proceed directly with implementation
```
