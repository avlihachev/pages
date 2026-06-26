---
canonicalURL: "https://dev.innalab.com/learn/introduction/"
title: "Module 0: What We're Building"
date: 2026-03-29
draft: false
tags: ["claude-code", "ai-agent", "mcp", "skill", "course"]
categories: ["Courses"]
description: "Meet the garden AI agent, a Telegram assistant that monitors weather, gives planting advice, and sends proactive notifications. Built entirely with vibe coding."
series: ["Building an AI Agent with Claude Code"]
weight: 1
ShowToc: true
---

Your garden doesn't care about your schedule. Frost hits at 3 AM. Your seedlings need hardening exactly 2 weeks before transplanting. The soil temperature crossed 10°C while you were in a meeting. And every gardening website gives you advice for "Zone 5," which means nothing when you're growing tomatoes in containers on a balcony in Finland at 63°N.

This is the kind of problem AI is actually good at. Real-time monitoring, domain-specific knowledge, proactive action based on data you can't manually check every day.

So we built a garden AI agent. And we built it entirely by directing Claude Code. Zero hand-written code.

This course shows you how. The full story, in eight minutes:

{{< youtube nThuytn1tbg >}}

## What the Agent Does

The garden agent lives in Telegram. You talk to it like you'd talk to an experienced gardener who also happens to check the weather obsessively.

**Ask a question:**
> "When should I start my tomato seedlings?"

It doesn't give you a generic "6-8 weeks before last frost" answer from the internet. It knows your location (Kokkola, Finland), your last frost date (June 15), your growing setup (containers, south-facing balcony, grow lights), and your specific tomato varieties. It calculates: "Start Sungold and Black Cherry under grow lights around April 20. San Marzano needs more time, start April 13."

**Send a photo.** Snap a picture of yellowing tomato leaves and it identifies nitrogen deficiency (likely from the container soil being depleted after 6 weeks), then suggests a specific fertilizer schedule based on what stage your plants are in.

**Proactive notifications:**

You wake up to a message: "Frost warning tonight (-2°C expected at 4 AM). Your hardening seedlings should come inside. Tomorrow looks clear, resume hardening in the afternoon."

You didn't ask for this. The agent checked the forecast, cross-referenced it with your plant list, and decided you needed to know.

## The Architecture

Three layers, each building on the previous one:

```
+----------------------------------------+
|  Layer 3: AI Agent                     |
|  Autonomous: polls, decides, acts      |
|  Agent SDK + Telegram integration      |
+----------------------------------------+
|  Layer 2: MCP Server                   |
|  Real-time data: weather, soil, sun    |
|  8 tools, 3 free APIs, no keys         |
+----------------------------------------+
|  Layer 1: Skill                        |
|  Domain knowledge: gardening logic     |
|  Parametric rules, not hardcoded       |
+----------------------------------------+
```

Each layer solves a specific limitation of the one below.

**The Skill** teaches Claude gardening. Without it, Claude gives generic internet-quality advice. With it, Claude understands frost date arithmetic, plant lifecycle stages, container-specific problems, and your personal growing conditions.

**The MCP Server** gives Claude real-time data. The skill knows *rules* ("start seedlings 8 weeks before last frost"), but it doesn't know today's weather. The MCP server connects to free APIs: Open-Meteo for weather forecasts, soil temperature, and evapotranspiration; sunrise-sunset.org for daylight hours; SoilGrids for soil composition. No API keys needed.

**The Agent** makes Claude autonomous. Without it, you have to open Claude Code and ask questions. With the agent, Claude polls your Telegram bot, processes messages, and sends proactive alerts. It checks the weather every morning and warns you about frost, heat stress, or watering needs. Without you asking.

## What Makes This Different

**Parametric, not hardcoded.** Most gardening tools give you a calendar: "Plant tomatoes in May." Useless if you're in Finland, or Thailand, or a windowsill apartment. Our skill calculates everything relative to your frost dates and growing conditions. Move to a different city? Update your profile, everything recalculates.

**Each layer works alone.** The skill by itself is already useful. Just install it and Claude Code becomes a gardening expert. Add the MCP server, and it also knows today's weather. Add the agent, and it becomes proactive. You don't need all three layers to get value.

**Built with vibe coding.** The entire system (skill, MCP server, Telegram bot, agent) was built by directing Claude Code. No hand-written code. If you can describe what you want in natural language and review what Claude Code produces, you can build an AI agent.

## The Vibe Coding Approach

"Vibe coding" sounds like a meme. In practice it looks like this:

1. **You describe what you want.** In words, not code. "Build an MCP server that provides frost risk data using the Open-Meteo API."

2. **Claude Code proposes an approach.** In plan mode, it shows you the architecture, file structure, and implementation steps. You review.

3. **You approve or redirect.** "This looks good, but use Zod for validation instead of manual checks." Claude Code adjusts.

4. **Claude Code writes the code.** You review the output, test it, iterate. First version is rarely final. You refine through conversation.

5. **You end up with working software.** Software you understand, because you directed every decision. Software you can modify, because you know why each piece exists.

The companion build-along video shows this exact process: the prompts, the plan mode output, the review-and-approve cycle, the iteration. The point is to learn how to direct an AI to build real systems.

## What You'll Need

- Claude Code installed and working
- A terminal (you'll be running commands, not writing code)
- A Telegram account
- Basic comfort with the command line: `cd`, `ls`, running `npm` commands
- No programming experience required. Seriously.

## Course Roadmap

| Module | What You'll Learn |
|--------|-------------------|
| **0. What We're Building** (you are here) | See the final result, understand the architecture |
| **1. Claude Code Essentials** | CLAUDE.md, plan mode, the review-approve workflow |
| **2. Building a Skill** | Teach Claude your domain with parametric knowledge |
| **3. Building an MCP Server** | Give Claude access to real-time data from free APIs |
| **4. Building an AI Agent** | Create an autonomous agent with Agent SDK + Telegram |
| **5. Putting It All Together** | Adapt the pattern to your own domain |

Each module is a standalone article. You can follow along from start to finish, or jump to the module that interests you most.

## Source Code

Everything is open source:

- [Garden Skill](https://github.com/avlihachev/claude-skill-garden)
- [Garden MCP Server](https://github.com/avlihachev/mcp-garden)
- [Garden Agent](https://github.com/avlihachev/garden-agent)

## Next: Building a Skill

[Module 2](/courses/claude-code-garden/02-skill/) starts with the foundation. You'll create a skill that turns Claude from a general-purpose assistant into a domain expert. Fastest way to see what Claude Code can really do.
