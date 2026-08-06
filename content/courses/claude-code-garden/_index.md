---
canonicalURL: "https://dev.innalab.com/learn/"
title: "Building an AI Agent with Claude Code"
description: "A project-based course on extending Claude Code with skills, MCP servers, and autonomous agents. No coding experience required: learn to direct AI to build real systems."
date: 2026-03-29
draft: false
categories: ["courses"]
ShowToc: true
ShowReadingTime: false
---

## What You'll Build

A garden AI assistant that lives in Telegram: it monitors weather, gives personalized
planting advice, accepts photos, and sends proactive notifications. All built by directing
Claude Code, not writing code by hand.

## Architecture

```
Skill (domain knowledge) → MCP Server (live data) → AI Agent (autonomy)
```

Each module adds a layer. By the end, you have a fully autonomous agent.

## Modules

| # | Module | What You'll Learn |
|---|--------|-------------------|
| 0 | [What We're Building](/courses/claude-code-garden/00-introduction/) | See the final result in action |
| 1 | [Claude Code Essentials](/courses/claude-code-garden/01-essentials/) | CLAUDE.md, plan mode, effective prompting |
| 2 | [Building a Skill](/courses/claude-code-garden/02-skill/) | Teach Claude your domain with structured knowledge |
| 3 | [Building an MCP Server](/courses/claude-code-garden/03-mcp/) | Give Claude access to real-time data |
| 4 | [Building an AI Agent](/courses/claude-code-garden/04-agent/) | Create an autonomous agent with the Agent SDK |
| 5 | [Putting It All Together](/courses/claude-code-garden/05-conclusion/) | Adapt the pattern to your own domain |

## Prerequisites

- Claude Code installed and working
- A Telegram account (for the bot)
- Terminal basics (cd, ls, running commands)
- No programming experience required

## Source Code

- [Garden Skill](https://github.com/avlihachev/claude-skill-garden)
- [Garden MCP Server](https://github.com/avlihachev/mcp-garden)
- [Garden Agent](https://github.com/avlihachev/garden-agent)

---

*This course is free. If you would rather not build it yourself, the same stack is delivered as [fixed-price agent packages](https://dev.innalab.com/): public prices, written scope, no sales calls.*
