Enable Figma MCP servers for this session. Run these commands to add them:

1. Add html-to-design:
```bash
claude mcp add html-to-design --type url --url "https://h2d-mcp.divriots.com/4c267f55-a7ce-41c3-b182-8136bb0e1268/mcp" -s user
```

2. Add ClaudeTalkToFigma:
```bash
claude mcp add ClaudeTalkToFigma -s user -- bunx claude-talk-to-figma-mcp@latest
```

After adding, restart Claude Code to load the servers.

To remove when done:
```bash
claude mcp remove html-to-design -s user
claude mcp remove ClaudeTalkToFigma -s user
```

## Prerequisites
- Run "Claude Talk to Figma" plugin in Figma (connects on port 3055)
- Have your Figma file open

$ARGUMENTS
