---
canonicalURL: "https://dev.innalab.com/learn/conclusion/"
title: "Module 5: Putting It All Together"
date: 2026-03-29
draft: false
tags: ["claude-code", "ai-agent", "course"]
categories: ["Courses"]
description: "The Skill + MCP + Agent pattern for any domain. How to adapt what we built to your own projects."
series: ["Building an AI Agent with Claude Code"]
weight: 6
ShowToc: true
---

We started with a problem: managing a garden with unpredictable weather, location-specific timing, and dozens of plants each with their own schedule. We ended with an AI agent in Telegram that knows your garden, checks the weather, and wakes you up when frost is coming.

More importantly, we built it by directing Claude Code. No hand-written code. The entire system (skill, MCP server, Telegram bot integration, autonomous agent) was created through conversation.

Let's step back and look at what we actually built, and how to use the same pattern for something else.

## The Three-Layer Pattern

```
+----------------------------------------+
|  Agent: acts on its own                |
|  Polls, decides, notifies              |
+----------------------------------------+
|  MCP Server: provides live data        |
|  APIs, sensors, external systems       |
+----------------------------------------+
|  Skill: encodes knowledge              |
|  Rules, formulas, domain expertise     |
+----------------------------------------+
```

**Skill** = what you know. Domain expertise encoded as parametric rules. Not "plant tomatoes in May" but "start seedlings N weeks before last_frost_date, where N depends on variety." No code, just markdown.

**MCP** = what's happening now. Real-time data from external sources. Weather, prices, sensor readings, API responses.

**Agent** = what to do about it. The skill says "bring seedlings inside if frost is expected." The MCP says "frost tonight." The agent sends you a notification at 6 PM.

Each layer works independently. A skill alone already makes Claude more useful. Add MCP for data-aware advice. Add the agent for proactive behavior. Start simple, add layers when you need them.

## Adapting to Your Domain

The pattern works anywhere you have domain knowledge + live data + a reason for proactive action.

### Personal Finance Agent

**Skill:** Budgeting rules. Categories and spending limits. Investment principles. Tax deadlines and rules for your country. Personal goals (save X by date Y).

**MCP:** Bank API (account balances, recent transactions). Currency exchange rates. Stock/crypto prices. Tax authority calendars.

**Agent:** Daily spending summary. Alert when approaching budget limits. Notify about unusual transactions. Remind about tax deadlines. Weekly financial review.

### Home Automation Agent

**Skill:** Room preferences (temperature, lighting). Energy-saving rules. Security protocols. Schedules (work hours, sleep, away).

**MCP:** Smart home API (thermostats, lights, sensors). Energy prices (spot market). Weather forecast. Calendar API (when you're away).

**Agent:** Optimize heating based on energy prices + weather. Turn off lights in empty rooms. Security alerts. Pre-heat before you arrive. Monthly energy report.

### Content Management Agent

**Skill:** Brand voice guide. Content calendar. Platform rules (character limits, hashtag strategies). Engagement patterns.

**MCP:** Social media APIs (post metrics, trending topics). Analytics dashboards. Competitor monitoring. RSS feeds for industry news.

**Agent:** Draft content based on trending topics. Schedule posts at optimal times. Alert on engagement spikes. Weekly performance report. Reply suggestions for comments.

## Key Principles

From building the garden project, things that apply broadly:

**Parametric over hardcoded.** Don't encode "do X on date Y." Encode "do X when condition Z is met." Conditions adapt to different users, locations, and circumstances.

**Graceful degradation.** Each layer should work without the others. MCP server down? The skill still gives good advice from reference data. Agent offline? The bot still handles commands.

**Start with the skill.** Fastest win, zero infrastructure. Add MCP when you need live data. Add the agent when you need proactive behavior.

**Build something you care about.** Gardening, finance, cooking, fitness. Motivation matters when you hit the inevitable debugging sessions. And a real domain produces a real portfolio piece, not a tutorial project nobody would use.

**Free data sources exist for everything.** Open-Meteo, sunrise-sunset.org, SoilGrids. No API keys, no costs. Most domains have equivalents. Government weather data, financial market APIs, public transit APIs, food nutrition databases. Search before building.

## The Vibe Coding Takeaway

The technical pattern is useful. The meta-skill is more important: **you can build complex systems by directing Claude Code.**

The entire garden project (three repos, multiple API integrations, a Telegram bot, an autonomous agent) was built through conversation. Plan mode for architecture, review-approve for implementation, iteration for refinement.

If you can clearly describe what you want, you can build it. The bottleneck is clarity of thought, not coding ability.

## Resources

**This course:**
- [Module 0: What We're Building](/courses/claude-code-garden/00-introduction/)
- [Module 1: Claude Code Essentials](/courses/claude-code-garden/01-essentials/)
- [Module 2: Building a Skill](/courses/claude-code-garden/02-skill/)
- [Module 3: Building an MCP Server](/courses/claude-code-garden/03-mcp/)
- [Module 4: Building an AI Agent](/courses/claude-code-garden/04-agent/)

**Source code:**
- [Garden Skill](https://github.com/avlihachev/claude-skill-garden)
- [Garden MCP Server](https://github.com/avlihachev/mcp-garden)
- [Garden Agent](https://github.com/avlihachev/garden-agent)

**Official docs:**
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)
- [MCP specification](https://modelcontextprotocol.io/)
- [Agent SDK](https://github.com/anthropics/claude-code/tree/main/packages/claude-agent-sdk)

## What's Next

This course covers the foundation. Possible directions:

- Multi-agent systems, multiple agents collaborating on a task
- Deployment: running agents on cloud servers instead of your Mac
- Prompt optimization for skill instructions
- Monitoring: logging, error tracking, cost management for production agents

If you build something using this pattern, I'd love to hear about it. Tag me on [X](https://twitter.com/avlihachev) or [Threads](https://threads.net/@lihachev).

<!-- {{< youtube VIDEO_ID_HERE >}} -->

---

*This course is free. If you would rather not build it yourself, the same stack is delivered as [fixed-price agent packages](https://dev.innalab.com/): public prices, written scope, no sales calls.*
