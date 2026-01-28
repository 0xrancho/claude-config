# Agent-Browser - LLM-Optimized Browser Automation

You have access to **agent-browser**, Vercel's headless browser CLI designed specifically for AI agents. Use this for all visual validation, screenshot capture, and browser automation tasks.

## Why Agent-Browser Over Raw Playwright

| Feature | Agent-Browser | Raw Playwright |
|---------|---------------|----------------|
| Element refs | `@e1`, `@e2` (LLM-native) | CSS selectors |
| Snapshots | AI-optimized structure | Generic DOM |
| Token usage | Compact flags available | Verbose |
| Design goal | Built FOR AI agents | General automation |

**Always prefer agent-browser for visual validation workflows.**

## Prerequisites

```bash
# Install globally (one time)
npm install -g agent-browser
agent-browser install  # Downloads Chromium
```

## Core Commands

### Session Management

```bash
# Open a URL (starts session)
agent-browser open https://example.com

# Close session
agent-browser close
```

### Page Snapshots (AI-Friendly)

```bash
# Full snapshot with element references
agent-browser snapshot

# Compact mode (fewer tokens)
agent-browser snapshot -c

# Interactive elements only
agent-browser snapshot -i

# Both compact + interactive
agent-browser snapshot -c -i
```

**Snapshot output example:**
```
@e1: button "Schedule a Consultation"
@e2: link "About Us"
@e3: heading "Honor Your Home"
@e4: image "hero-background.jpg"
```

### Screenshots

```bash
# Full page screenshot
agent-browser screenshot ./path/to/save.png

# Specific element (by reference)
agent-browser screenshot ./element.png --element @e3
```

### Interactions

```bash
# Click by reference
agent-browser click @e2

# Hover (for hover states)
agent-browser hover @e5

# Type text
agent-browser type @e7 "Hello world"

# Scroll
agent-browser scroll down 500
agent-browser scroll up 300
agent-browser scroll to @e10  # Scroll element into view
```

### Find Elements

```bash
# By role
agent-browser find role button

# By text content
agent-browser find text "Schedule"

# By label
agent-browser find label "Email"
```

## Visual Validation Workflow

### Phase 0: Capture Reference Baseline

```bash
# 1. Open reference site
agent-browser open https://retrofit.design

# 2. Capture snapshot (element structure)
agent-browser snapshot > Branding/visual-snapshots/reference-snapshot.txt

# 3. Desktop screenshot
agent-browser screenshot Branding/visual-snapshots/reference-desktop.png

# 4. Scroll sequence (for parallax/scroll effects)
agent-browser scroll down 500
agent-browser screenshot Branding/visual-snapshots/reference-scroll-500.png
agent-browser scroll down 500
agent-browser screenshot Branding/visual-snapshots/reference-scroll-1000.png

# 5. Hover states
agent-browser hover @e3  # CTA button from snapshot
agent-browser screenshot Branding/visual-snapshots/reference-cta-hover.png

# 6. Close
agent-browser close
```

### Validation Loop: Compare Dev to Reference

```bash
# 1. Open dev build
agent-browser open http://localhost:3000

# 2. Capture current state
agent-browser snapshot > current-snapshot.txt
agent-browser screenshot current-desktop.png

# 3. Compare (in your evaluation)
# - Diff snapshots for structural changes
# - Compare screenshots for visual differences

# 4. Check specific elements
agent-browser hover @e3
agent-browser screenshot current-cta-hover.png

# 5. Close
agent-browser close
```

### Responsive Testing

```bash
agent-browser open http://localhost:3000

# Desktop (default)
agent-browser screenshot desktop.png

# Tablet - resize viewport
agent-browser evaluate "window.resizeTo(768, 1024)"
agent-browser screenshot tablet.png

# Mobile
agent-browser evaluate "window.resizeTo(375, 812)"
agent-browser screenshot mobile.png

agent-browser close
```

## Animation & Scroll Documentation

### Capture Scroll Behavior

```bash
agent-browser open http://localhost:3000

# Document scroll sequence
for pos in 0 300 600 900 1200; do
  agent-browser scroll to-position $pos
  agent-browser screenshot scroll-$pos.png
done

agent-browser close
```

### Capture Hover States

```bash
agent-browser open http://localhost:3000
agent-browser snapshot -i > interactive-elements.txt

# For each interactive element in snapshot:
agent-browser screenshot before-hover-cta.png
agent-browser hover @e5
agent-browser screenshot after-hover-cta.png

agent-browser close
```

### Extract Animation CSS

```bash
agent-browser open http://localhost:3000
agent-browser evaluate "
  const el = document.querySelector('.cta-button');
  const style = getComputedStyle(el);
  return {
    transition: style.transition,
    animation: style.animation,
    transform: style.transform
  };
"
agent-browser close
```

## Integration with superUX

When superUX needs browser automation, use agent-browser via Bash:

```
superUX workflow:
1. Bash: agent-browser open [url]
2. Bash: agent-browser snapshot > snapshot.txt
3. Read: snapshot.txt (understand element structure)
4. Bash: agent-browser screenshot current.png
5. Read: current.png (visual analysis)
6. Evaluate against reference
7. Bash: agent-browser close
```

## Compact Output Flags

Reduce token usage in AI contexts:

| Flag | Effect |
|------|--------|
| `-c` | Compact output (minimal whitespace) |
| `-i` | Interactive elements only |
| `-d` | Include dimensions |

```bash
# Minimal snapshot for quick checks
agent-browser snapshot -c -i
```

## Error Handling

```bash
# Check if session is active
agent-browser status

# Force close all sessions
agent-browser close --all

# Restart if stuck
agent-browser close --all && agent-browser open [url]
```

## Common Patterns

### Full Page Validation

```bash
agent-browser open http://localhost:3000
agent-browser snapshot -c > structure.txt
agent-browser screenshot full-page.png
agent-browser close

# Then compare:
# - structure.txt vs reference-snapshot.txt
# - full-page.png vs reference-desktop.png
```

### Component Isolation Test

```bash
# Test a specific component in isolation
agent-browser open http://localhost:3000/test/chevron-divider
agent-browser snapshot -i
agent-browser screenshot chevron-test.png
agent-browser scroll down 100
agent-browser screenshot chevron-scrolled.png
agent-browser close
```

### Multi-Breakpoint Capture

```bash
#!/bin/bash
URL=$1
BREAKPOINTS="320 768 1024 1440"

agent-browser open $URL

for bp in $BREAKPOINTS; do
  agent-browser evaluate "window.resizeTo($bp, 900)"
  agent-browser screenshot "screenshot-${bp}px.png"
done

agent-browser close
```

## When to Use What

| Task | Tool |
|------|------|
| Visual validation | agent-browser (this) |
| Complex JS evaluation | Playwright MCP via superUX |
| Design system extraction | superdesign MCP |
| Video recording | playwright-record-mcp (if installed) |

## Repository

GitHub: https://github.com/vercel-labs/agent-browser

## Troubleshooting

### Chromium not found
```bash
agent-browser install
# On Linux with missing deps:
agent-browser install --with-deps
```

### Session already open
```bash
agent-browser close --all
```

### Element reference not found
```bash
# Re-run snapshot to get fresh references
agent-browser snapshot
# References change when page content changes
```
