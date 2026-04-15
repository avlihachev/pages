---
title: "Garden Agent: Autonomous Monitoring for Your Garden"
description: "AI agent that proactively monitors weather, soil conditions, and plant schedules — sends Telegram alerts before problems happen."
date: 2026-03-15
tags: ["claude-agent-sdk", "mcp", "telegram", "ai-agent", "monitoring"]
categories: ["projects", "ai-agents"]
---

# Garden Agent: Autonomous Monitoring for Your Garden

## The Problem

Gardening in Nordic conditions means short growing seasons and fast-changing weather. A late frost can kill transplants overnight. A dry spell needs watering before the soil dries out completely. By the time you check the forecast manually, it's often too late.

Garden Agent watches conditions continuously and alerts you before problems happen.

## What It Does

The agent runs autonomously on a schedule. It checks:

- **Weather forecast** — temperature drops, frost risk, rain predictions
- **Soil conditions** — temperature, moisture estimates based on weather patterns
- **Plant schedules** — sowing windows, transplant timing, harvest readiness

When something needs attention, it sends a Telegram message: "Frost warning tonight (−2°C). Cover your tomato transplants." or "No rain forecast for 5 days — water raised beds tomorrow."

Not reactive alerts after the fact. Proactive warnings with enough lead time to act.

## Why It's an Agent

A simple weather bot checks temperature and sends an alert when it drops below a threshold. Garden Agent reasons about context — it knows what you're growing, what stage the plants are in, and what conditions matter for each crop at each stage.

Tomato seedlings in week 2 after transplant need frost protection at +3°C. Established potatoes are fine until −1°C. The agent applies different thresholds based on what's actually in the ground.

It combines data from multiple sources (weather APIs, soil temperature models, planting calendars) and makes a judgment call about what's worth alerting on. Not every temperature drop matters — only the ones that affect your specific plants.

## Tech Stack

```text
- Claude Agent SDK (autonomous reasoning + scheduling)
- MCP Garden server (7 tools: weather, soil, frost risk, daylight, evapotranspiration)
- Telegram Bot API (alert delivery)
```

## The MCP Garden Server

The agent's data comes from a dedicated MCP server with 7 tools:

- `get_forecast` — multi-day weather prediction
- `get_frost_risk` — frost probability with confidence levels
- `get_soil_temperature` — estimated soil temp at planting depth
- `get_soil_type` — local soil characteristics
- `get_evapotranspiration` — water loss rate for irrigation planning
- `get_daylight` — sunrise, sunset, day length for light-sensitive crops
- `garden_check` — combined health check across all monitored plants

Each tool returns structured data the agent can reason about — not raw numbers, but context-rich responses with thresholds and recommendations.

Built as part of [AI Agent Development practice](https://dev.innalab.com).

Source on [GitHub](https://github.com/avlihachev/garden-agent).
