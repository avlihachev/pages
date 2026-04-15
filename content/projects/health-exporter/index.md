---
title: "Health Exporter: 17-Tool MCP Server for Apple Health"
description: "MCP server that makes 5.6 million Apple Health records queryable through Claude. 17 tools covering workouts, heart rate, sleep, nutrition, and more."
date: 2026-03-01
tags: ["mcp", "typescript", "sqlite", "apple-health", "claude", "ai-agent"]
categories: ["projects", "ai-agents"]
---

# Health Exporter: 17-Tool MCP Server for Apple Health

## The Problem

Apple Health collects everything — workouts, heart rate, sleep, steps, nutrition, blood pressure, body measurements. Years of data, millions of records. But you can't ask it questions. "What was my average resting heart rate last month?" requires exporting XML, writing scripts, parsing timestamps.

Health Exporter puts all of it behind MCP tools that Claude can call directly.

## What It Does

Export your Apple Health data once. Health Exporter parses the XML, loads it into SQLite, and exposes 17 MCP tools organized by health domain:

- **Workouts** — query by type, date range, duration, calories, distance
- **Heart rate** — resting, active, trends over time, anomaly detection
- **Sleep** — duration, stages, consistency patterns
- **Nutrition** — macros, calories, micronutrients
- **Body measurements** — weight, body fat, trends
- **Activity** — steps, flights climbed, stand hours
- **Vitals** — blood pressure, respiratory rate, blood oxygen

Ask Claude natural questions: "Compare my running pace this month vs last month." "Show days where my resting heart rate was above 70." "What's my average protein intake on training days vs rest days?" Claude calls the right tools, aggregates the data, and answers.

## Scale

5.6 million health records spanning multiple years. The SQLite database handles this without issues — queries return in milliseconds. MCP tools include pagination and date filtering so Claude doesn't try to load everything at once.

## Tech Stack

```text
- TypeScript
- MCP SDK (@modelcontextprotocol/sdk)
- SQLite (better-sqlite3)
- Apple Health XML parser
```

## Why MCP

A single monolithic API endpoint would work for simple queries. But health data has too many domains — asking about sleep patterns requires different parameters than querying workout history. MCP's tool model lets each domain expose its own interface with typed parameters. Claude picks the right tool based on the question.

17 tools sounds like a lot. In practice each tool is narrow and well-defined — `query_workouts`, `query_heart_rate`, `query_sleep`, `get_nutrition_summary`, etc. Claude rarely needs more than 2-3 per question.

Built as part of [AI Agent Development practice](https://dev.innalab.com).

Source on [GitHub](https://github.com/avlihachev/health-exporter).
