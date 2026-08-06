---
canonicalURL: "https://dev.innalab.com/learn/mcp/"
title: "Module 3: Building an MCP Server — Giving Claude Real-Time Data"
date: 2026-03-29
draft: false
tags: ["claude-code", "mcp", "ai-agent", "course"]
categories: ["Courses"]
description: "Build an MCP server that gives Claude Code access to weather forecasts, frost risk, soil temperature, and daylight data. All from free APIs, no keys required."
series: ["Building an AI Agent with Claude Code"]
weight: 4
ShowToc: true
---

In [Module 2](/courses/claude-code-garden/02-skill/) we gave Claude domain knowledge: gardening rules, frost date arithmetic, plant lifecycle management. Claude now knows *how* gardening works. But it doesn't know what's happening *right now*.

Ask Claude "should I start hardening my seedlings this week?" and it can calculate the ideal date from your profile. But it can't check if this week is actually warm enough. It knows your last frost date is June 15, but doesn't know that an unusual cold front is dropping temperatures to -5°C tonight.

Static knowledge needs live data. That's what MCP provides.

## What Is MCP?

MCP (Model Context Protocol) is a standard for giving AI models access to external tools and data. Instead of Claude guessing or using training data, it can call a tool, like checking the weather, and get a real answer.

An MCP server is a program that exposes tools Claude Code can use. You build it, register it with Claude Code, and Claude calls the tools when relevant. Compare:

- "Tomatoes need 20-25°C to germinate" (from skill knowledge)
- "Soil temperature at your location is currently 8°C at 6cm depth, too cold for direct sowing" (from MCP data)

## What We're Building

Eight tools, three free APIs, zero API keys:

| Tool | What it does | Data source |
|------|-------------|-------------|
| `get_forecast` | 16-day weather forecast | Open-Meteo |
| `get_frost_risk` | Frost/hard frost prediction per day | Open-Meteo |
| `get_soil_temperature` | Soil temp at 4 depths + moisture | Open-Meteo |
| `get_daylight` | Sunrise, sunset, day length | sunrise-sunset.org |
| `get_evapotranspiration` | Watering needs based on ET₀ | Open-Meteo |
| `get_soil_type` | Soil texture, pH, organic carbon | SoilGrids ISRIC |
| `garden_check` | Smart aggregator, everything at once | All sources |
| `garden_write` | Write updates back to your garden files | local skill files |

Seven of these fetch data. The eighth, `garden_write`, writes it back: it lets an agent update your `plants.md` and `journal.md` through the MCP layer instead of touching files directly. We rely on it in [Module 4](/courses/claude-code-garden/04-agent/). The data tools are the focus here.

The `garden_check` tool is the star. It calls the other tools, filters results based on season and latitude, and returns only what matters right now. In March at 63°N, you get frost risk and grow light reminders. In July, you get heat stress warnings and watering needs. No noise.

## The Data Sources

All free, all open, no registration:

**Open-Meteo** (api.open-meteo.com) does the heavy lifting. Weather forecasts, soil temperature at 4 depths, soil moisture, evapotranspiration, precipitation. 10,000 requests per day, no API key. This single source powers five of the data tools.

**sunrise-sunset.org** provides sunrise, sunset, and day length for any coordinates and date. Simple REST API, no limits stated. Critical for high-latitude gardening where day length swings from 5 hours in December to 23 hours in June.

**SoilGrids ISRIC** (rest.isric.org) has soil composition data from satellite and ground measurements. Sand/silt/clay percentages, pH, organic carbon. 5 requests per minute, no key. Useful for initial garden setup, tells you what you're working with before you amend the soil.

## Let's Build It

### Step 1: Scaffold the MCP Server

Tell Claude Code what we're building:

> "Create an MCP server project called mcp-garden. TypeScript, stdio transport. Use @modelcontextprotocol/sdk. Include Zod for input validation. The server will provide weather and soil data tools for gardening."

Claude Code will scaffold the project:

```
mcp-garden/
├── src/
│   ├── index.ts              # MCP server entry + tool registration
│   ├── tools/                # one file per tool
│   ├── clients/              # API wrappers
│   └── utils/                # validation schemas, seasonal logic
├── package.json
├── tsconfig.json
└── README.md
```

The structure follows a clean pattern: **clients** talk to APIs, **tools** use clients and format results, **index.ts** registers tools with the MCP server. One tool per file, one client per API.

### Step 2: API Clients

Three clients, one per data source. Each is a thin wrapper around `fetch`:

> "Create src/clients/open-meteo.ts, a client for the Open-Meteo API. Functions: getForecast(lat, lon, days), getFrostRisk(lat, lon, days), getSoilTemperature(lat, lon), getEvapotranspiration(lat, lon, days). Each function builds the API URL, fetches, and returns parsed JSON."

The prompt pattern is the same for each client. You specify the API, the functions you need, and Claude Code generates the fetch wrappers.

Key thing to review: **error handling**. APIs go down. Claude Code will add try/catch by default, but make sure it returns partial data with warnings rather than failing completely. A forecast without soil data is still useful.

### Step 3: Register Tools

Each tool gets a name, a description, an input schema (Zod), and a handler function. The descriptions matter. Claude Code reads them to decide which tool to call.

> "Register get_forecast as an MCP tool. Input: lat (number), lon (number), days (1-16, default 7). Description: 'Get weather forecast for the next N days. Returns daily min/max temperature, precipitation, wind, and cloud cover.' Handler calls the Open-Meteo client and formats the result."

Repeat for each tool. The aggregator `garden_check` is more interesting: it calls the other tools and applies seasonal logic:

> "Create the garden_check tool. It calls get_forecast, get_frost_risk, get_soil_temperature, get_daylight, and get_evapotranspiration for the given location. Then it filters results based on season and latitude: include frost risk in spring/fall at high latitudes, heat stress in summer, daylight warnings when day length is short, grow light reminders when day length is under 14h during spring. Return a structured overview with an alerts array."

The seasonal logic is the intelligence layer. Without it, `garden_check` would dump all data every time. With it, you get relevant alerts: frost warnings in March, watering reminders in July.

### Step 4: Connect to Claude Code

Build and register:

```bash
cd mcp-garden
npm run build
claude mcp add garden-mcp node $(pwd)/dist/index.js
```

That's it. Claude Code now has access to all 8 tools. Ask a question that needs weather data and watch it call the right tool automatically.

## Testing

Once registered, test with real questions:

> "What's the frost risk for my location this week?"

Claude Code should call `get_frost_risk` with your coordinates from the skill profile. You'll see the tool call in the terminal output. MCP tools are visible, not hidden.

> "Is the soil warm enough to transplant outside?"

Claude Code calls `get_soil_temperature`, checks the surface and 6cm readings against the skill's reference data (tomatoes need 10°C+ at 6cm depth), and gives a specific answer.

> "Give me a garden status update."

This triggers `garden_check`, the aggregator. You get a weather summary, frost alerts if relevant, daylight info, soil temperatures, and seasonal recommendations. All in one response.

**Without MCP:** "Tomatoes generally need soil temperatures above 10°C for transplanting. You can check your local weather service."

**With MCP:** "Soil temperature at your location: 8°C at 6cm depth, 5°C at 18cm. Too cold for transplanting. Forecast shows warming trend next week, expected 12°C at 6cm by Friday. I'd wait until next weekend. No frost risk in the 7-day forecast."

## Skill + MCP Integration

When both the skill and MCP server are active, Claude Code combines them automatically. The skill provides the rules ("transplant when soil > 10°C"), the MCP provides the data ("soil is currently 8°C"), and Claude makes the decision ("wait until Friday").

Add a section to your SKILL.md to make this explicit:

```markdown
## MCP Integration

If garden MCP tools are available, use them for:
- Real weather data instead of formula-based estimates
- Frost alerts for proactive warnings
- Watering recommendations based on evapotranspiration data
- Soil type detection during onboarding
- Daylight monitoring for grow light reminders
```

The skill knows to fall back to formulas when MCP isn't available. Graceful degradation: both layers work alone, but they're better together.

## What We Built

An MCP server with 8 tools that give Claude real-time gardening intelligence and a way to write results back. No API keys, no costs, no external dependencies beyond free web APIs. Two npm packages: the MCP SDK and Zod.

The pattern: find free data sources, wrap them in MCP tools with clear descriptions, add an aggregator that filters by context, connect to Claude Code. One command.

Works for any domain. Finance: wrap stock APIs and currency exchanges. Home automation: wrap your smart home API. Content management: wrap social media analytics.

But we still have a gap. Skill + MCP gives Claude knowledge and data, only when you ask. You have to open Claude Code, type a question, wait for an answer. Nobody checks the weather at 3 AM when frost is about to hit.

In [Module 4](/courses/claude-code-garden/04-agent/) we add the missing piece: an autonomous agent that polls your Telegram bot, processes messages, and sends proactive notifications without being asked.

## Source Code

[Garden MCP Server on GitHub](https://github.com/avlihachev/mcp-garden)

<!-- {{< youtube VIDEO_ID_HERE >}} -->

---

*This course is free. If you would rather not build it yourself, the same stack is delivered as [fixed-price agent packages](https://dev.innalab.com/): public prices, written scope, no sales calls.*
