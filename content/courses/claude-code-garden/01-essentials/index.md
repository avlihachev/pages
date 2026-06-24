---
canonicalURL: "https://dev.innalab.com/learn/essentials/"
title: "Module 1: Claude Code Essentials"
date: 2026-03-29
draft: false
tags: ["claude-code", "tutorial", "course"]
categories: ["Courses"]
description: "Everything you need to know about Claude Code before building with it. CLAUDE.md, plan mode, prompting patterns, and the review-approve workflow."
series: ["Building an AI Agent with Claude Code"]
weight: 2
ShowToc: true
---

Before we build anything, you need to understand the tool. Claude Code isn't a chatbot. It's an AI that lives in your terminal, reads your project files, writes code, and runs commands (with your permission). Think of it less like asking someone for advice, more like having someone sit next to you and do the work while you review.

If you've been using Claude Code for a while, skim this. If you're new, read carefully. These patterns make everything in the later modules work.

## CLAUDE.md: Your Project Instructions

Every project should have a `CLAUDE.md` file at its root. This is the first thing Claude Code reads when you start a session. Not documentation for humans. Instructions for Claude.

A minimal CLAUDE.md:

```markdown
# CLAUDE.md

## Project Overview
TypeScript Node.js app that does X.

## Commands
- npm run dev — start dev server
- npm run build — compile
- npm test — run tests
```

A good CLAUDE.md adds context Claude can't infer from the code:

```markdown
# CLAUDE.md

## Project Overview
TypeScript Telegram bot for weather monitoring. Deployed on Fly.io.

## Architecture
- src/index.ts — entry point, Express server, bot handlers
- src/services/ — weather API, user storage, scheduler
- src/bot/commands.ts — Telegram command handlers

## Environment
Required in .env: BOT_TOKEN, WEATHER_API_KEY

## Commands
- npm run dev — development mode
- npm run build — compile TypeScript
- npm start — production
- npx vitest run — tests

## Deployment
fly deploy — pushes to Fly.io, region fra, port 3000
```

Without CLAUDE.md, Claude Code spends time exploring your project structure, reading random files, trying to figure out what things do. With a good one, it starts with context and goes straight to work.

**Rule of thumb:** Put in CLAUDE.md anything you'd tell a new team member on their first day. Project structure, how to run things, where the tricky parts are.

## The Workflow: Ask, Review, Approve

Claude Code follows a permission-based workflow. You ask, it proposes, you approve.

**1. You describe what you want.**

Not in code. In plain language:

> "Add a health check endpoint at /health that returns 200 OK"

Be specific. "Add an API endpoint" is vague. "Add a GET /health endpoint that returns JSON `{status: 'ok'}`, use the existing Express app in src/index.ts" is clear.

**2. Claude Code proposes changes.**

It reads your project, figures out where the change goes, and shows you what it wants to do. You'll see the file, the diff, the reasoning.

**3. You review.**

The critical step. Read what Claude Code proposes. Does it make sense? Is it in the right file? Did it understand the requirement?

**4. You approve, reject, or redirect.**

- **Approve**: Claude Code makes the change
- **Reject**: nothing happens, try again
- **Redirect**: "Almost, but put it in a separate file" or "Use Zod for validation instead"

This loop is the entire workflow. For simple tasks, it's one cycle. For complex features, it's dozens of cycles. The skill is knowing when to approve and when to push back.

## Plan Mode

For anything non-trivial, use plan mode. Hit `Shift+Tab` or type `/plan` to enter it. Claude Code stops writing code and proposes an approach instead.

> "Build an MCP server with 7 weather tools"

In plan mode, Claude Code will:
1. Propose the file structure
2. List the tools with their inputs/outputs
3. Suggest the implementation order
4. Identify potential issues

You review the plan, adjust it, and approve. Only then does Claude Code start implementing. This prevents the most common vibe coding mistake: letting Claude Code charge ahead in the wrong direction for 10 minutes before you notice.

**When to use plan mode:**
- New features with multiple files
- Architectural changes
- Anything where the *approach* matters, not just the code

**When to skip it:**
- Bug fixes where you know exactly what's wrong
- Small changes to a single file
- Adding a test for existing code

## Memory and Context

Claude Code's memory within a session is the conversation history plus your project files. It reads files on demand, not preloaded.

This means:
- CLAUDE.md is always available (loaded at session start)
- Other files are read when Claude Code needs them or when you mention them
- Long sessions can hit context limits. The system compresses older messages automatically
- New sessions start fresh, but CLAUDE.md and memory files persist

**Memory files** (`.claude/memory/`) let Claude Code remember things across sessions: user preferences, project decisions, architectural choices. Useful for ongoing projects.

## Hooks

Hooks are shell commands that run automatically in response to Claude Code events. Before or after tool calls, when files change, etc. Configured in `.claude/settings.json`.

You don't need hooks for this course, but knowing they exist matters. Examples:
- Run linting after every file edit
- Block certain file patterns from being modified
- Auto-format code before commits

We'll see hooks in action in later modules when we set up the agent project.

## Tips for Effective Vibe Coding

These come from building the garden project. Not theory, patterns that actually worked.

**Be specific.** "Build a weather tool" gets you something generic. "Build a get_frost_risk tool that calls Open-Meteo API with lat/lon, returns per-day frost boolean where min < 0°C, uses Zod for input validation" gets you exactly what you need.

**Review before approving.** Every time. Even when you're confident Claude Code got it right. If you can't explain why a change was made, you've lost control.

**Use plan mode for anything multi-step.** 30 seconds reviewing a plan saves 10 minutes undoing wrong implementations.

**Iterate in small steps.** Don't ask Claude Code to "build the entire MCP server." Scaffold the project, add one tool, test it, add another. Each step is reviewable and works on its own.

**Let Claude Code explore first.** For existing codebases, start with: "Read the project and explain the architecture." Claude Code reads files, maps the structure, summarizes. Now it has context, and so do you.

**First version is rarely final.** Plan for 2-3 iterations on any non-trivial task. First pass gets the structure right, second fixes edge cases, third cleans up.

**Don't fight the tool.** If Claude Code keeps suggesting a different approach, consider that it might be right. It's read a lot of code. That said, you know your project and your constraints. Use judgment.

## What's Next

You now know enough about Claude Code to build real things with it. The rest of this course is hands-on:

- [Module 2: Building a Skill](/courses/claude-code-garden/02-skill/) turns Claude into a gardening expert
- [Module 3: Building an MCP Server](/courses/claude-code-garden/03-mcp/) gives Claude real-time weather data
- [Module 4: Building an AI Agent](/courses/claude-code-garden/04-agent/) makes Claude autonomous

Every module uses the same workflow: plan mode, review-approve, iterate.

<!-- {{< youtube VIDEO_ID_HERE >}} -->
