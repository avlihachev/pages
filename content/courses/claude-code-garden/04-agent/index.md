---
canonicalURL: "https://dev.innalab.com/learn/agent/"
title: "Module 4: Building an AI Agent — Autonomous Intelligence"
date: 2026-03-29
draft: false
tags: ["claude-code", "agent-sdk", "ai-agent", "telegram", "course"]
categories: ["Courses"]
description: "Build an autonomous AI agent that connects to Telegram, processes messages with your skill and MCP server, and sends proactive notifications."
series: ["Building an AI Agent with Claude Code"]
weight: 5
ShowToc: true
---

In Module 2, we gave Claude domain knowledge. In Module 3, we gave it real-time data. But everything still requires *you* to ask. You open Claude Code, type a question, get an answer. Nobody is opening their laptop at 3 AM to ask "is there frost tonight?"

An agent changes this. It polls for messages, processes them, and takes initiative on its own. Accessible from your phone through Telegram.

## What Makes an Agent?

A tool waits for you to call it. An MCP server waits for Claude to call it. An agent responds when asked *and* acts when it decides to.

Our garden agent:
- **Responds** to messages you send via Telegram (questions, photos, journal entries)
- **Monitors** the weather every morning and checks for frost every 4 hours
- **Notifies** you proactively when something needs attention
- **Remembers** your garden state through skill files (plants.md, journal.md)

It uses the same skill and MCP server we built in previous modules. The agent is the glue that makes them autonomous.

## Architecture

```
┌─────────────────┐         ┌─────────────────────┐
│  Telegram User   │         │   Your Machine       │
│  (phone)         │         │                      │
│  text / photo    │         │  garden-agent/       │
└───────┬──────────┘         │  ├── polling.ts      │
        │                    │  ├── agent.ts        │
        ▼                    │  ├── proactive.ts    │
┌─────────────────┐  poll   │  └── config.ts       │
│  garden_bot      │◄───────│                      │
│  (Fly.io)        │        │  Agent SDK           │
│                  │ reply   │  + Garden Skill      │
│  Telegraf +      │◄───────│  + Garden MCP        │
│  Express +       │        │                      │
│  AgentQueue      │        └─────────────────────┘
└─────────────────┘
```

Two pieces:

**The Telegram bot** (existing, on Fly.io) gets a small upgrade: a message queue and two API endpoints. When users send free text or photos, the bot queues them for the agent. Bot commands (`/location`, `/threshold`, `/status`) still work directly, no agent needed.

**The agent** (new, runs on your machine) polls the bot for messages on a short interval (5 seconds by default, configurable). For each message, it runs the Agent SDK with your garden skill as the system prompt and the MCP server for live data. Replies go back through the bot to Telegram.

Deliberate design. The bot is the always-on gateway. The agent is the intelligence layer. If the agent is offline (your machine is sleeping), the bot still works. It just can't answer free-text questions. Messages queue up and get processed when the agent comes back.

## Step 1: Bot Changes, the Message Queue

The existing bot needs three additions. Tell Claude Code:

> "Add a message queue to the garden bot for AI agent integration. Create src/services/agentQueueService.ts, an in-memory FIFO queue, max 100 messages. Each message has: id, chatId, type (text/photo), text, optional photoBase64, timestamp."

Then the API endpoints:

> "Add two API endpoints to the Express server. GET /api/agent/messages returns queued messages and clears the queue. POST /api/agent/reply sends a message back to a Telegram user. Both require an AGENT_SECRET header for auth."

And message routing:

> "In the bot command handler, route free text and photos to the agent queue. Bot commands (/start, /location, etc.) work as before. When the user sends free text, queue it for the agent. When they send a photo, download it, convert to base64, queue with caption."

Three changes, three prompts. Claude Code reads the current architecture from CLAUDE.md and fits the changes into the existing patterns.

## Step 2: Scaffold the Agent

The agent is a separate project:

> "Create a new project garden-agent. TypeScript, Node.js. Structure: src/index.ts (entry point), src/polling.ts (poll bot for messages), src/agent.ts (Agent SDK wrapper), src/proactive.ts (scheduled checks), src/config.ts (env vars). Install @anthropic-ai/claude-agent-sdk."

The agent project is independent from the bot. Different repo, different machine, different lifecycle. They communicate through HTTP.

## Step 3: Message Processing

The core loop in `polling.ts`:

1. Every few seconds (5 by default): `GET /api/agent/messages`
2. For each message: call the Agent SDK
3. Send reply: `POST /api/agent/reply`

The Agent SDK call in `agent.ts` is where everything comes together:

> "Create the agent wrapper. Use Agent SDK's query() function. System prompt: load SKILL.md plus profile.md from the garden skill directory. MCP server: connect garden-mcp via stdio. Allowed tools: Read and all garden MCP tools. Updates to plants.md and journal.md go through the garden_write MCP tool, not the raw Write tool. Working directory: ~/.claude/skills/garden/."

Why route writes through `garden_write` instead of the Write tool? It keeps the agent's file changes inside one auditable, validated path. The agent reads freely, but every change to your garden files goes through the single tool we built for it in Module 3.

When someone sends "when should I start hardening my tomatoes?", the agent:
1. Reads SKILL.md (gardening rules)
2. Reads plants.md (your tomato varieties and their current status)
3. Calls garden MCP's get_forecast (checks next week's weather)
4. Combines everything into a specific, personalized answer
5. Sends it back through Telegram

Photos work the same way: save the base64 to a temp file, pass it to the agent as an image, delete after processing.

## Step 4: Proactive Notifications

This is what makes an agent truly useful. Two scheduled checks:

**Morning check (08:00 daily):**

> "Add a morning check cron job. At 08:00, call the Agent SDK with the prompt: 'Check weather forecast, seasonal calendar, and plant statuses. Report only critical items. Return empty string if nothing urgent.' If the response is non-empty, send it to the user via the bot's reply endpoint."

The agent uses MCP tools (garden_check, get_frost_risk) and reads plants.md. It decides what's worth reporting. A warm day with no issues? Silence. Frost tonight with seedlings still outside? Alert.

**Frost emergency (every 4 hours):**

> "Add a frost emergency check that runs every 4 hours. This one doesn't use the LLM, it calls get_frost_risk directly via MCP. If hard frost (-2°C or colder) is predicted within 48 hours, send an alert through the bot. This is cheap, no tokens spent."

Two levels: the morning check uses the full LLM for nuanced analysis. The frost check is a direct API call, fast and free, runs even if your LLM budget is exhausted.

**What I actually run.** In production I disabled the morning check. The code stays, the cron line is commented out. The LLM-free frost check already covers the case that matters, and a daily 8 AM model call wasn't earning its tokens. Build the morning check, run it for a week, then decide whether it pays for itself. Most scheduled LLM jobs don't. That economics is the spine of the [case-study video](/courses/claude-code-garden/00-introduction/).

**Spam protection:** Max one proactive message per day (except emergency frost alerts). Nobody wants a chatty bot.

## Graceful Degradation

This is a core design principle. When the agent is offline:

- All bot commands work normally (`/status`, `/location`, `/toggle`)
- Weather threshold alerts still fire (those live in the bot, not the agent)
- Free text and photos queue up (max 100 messages)
- Bot tells the user "agent offline, message queued"
- When the agent starts, it processes the queue chronologically
- Messages older than 24 hours are skipped as stale

You can literally delete the agent project and the bot works exactly as it did before. Zero coupling. The agent is a pure add-on.

## Testing

**Manual test (message flow):**
```bash
# in one terminal: start the agent
cd garden-agent && npm run dev

# in Telegram: send a message to your bot
"when should I start hardening my tomatoes?"

# watch the agent terminal: poll → process → reply
# check Telegram: personalized answer arrives
```

**Manual test (proactive notifications):**

Trigger the morning check manually (or set the cron to run in 1 minute). Send a frost alert test by temporarily lowering the threshold.

**Manual test (offline behavior):**

Stop the agent. Send messages via Telegram. Verify they queue up. Start the agent. Watch it process the backlog.

## What We Built

The full stack:

| Layer | What | Why |
|-------|------|-----|
| **Skill** | Domain knowledge | Claude knows gardening |
| **MCP** | Real-time data | Claude knows today's weather |
| **Agent** | Autonomy | Claude acts without asking |

Each layer works alone. Together they produce something none of them could individually: a personalized, data-aware, proactive garden assistant accessible from your phone.

The pattern (skill for knowledge, MCP for data, agent for autonomy) isn't specific to gardening. In [Module 5](/courses/claude-code-garden/05-conclusion/) we look at how to adapt it to your own domain.

## Source Code

- [Garden Bot](https://github.com/avlihachev/garden-bot)
- [Garden Agent](https://github.com/avlihachev/garden-agent)

<!-- {{< youtube VIDEO_ID_HERE >}} -->
