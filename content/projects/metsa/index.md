---
title: "Metsä Advisor: Finnish Forest Intelligence on Three Open APIs"
description: "AI agent that turns a Finnish kiinteistötunnus into a forestry report — boundary, stand inventory, timber valuation — by orchestrating three open government APIs. Built with Claude Agent SDK, MCP, Hono, and Lit."
date: 2026-04-21
tags: ["claude-agent-sdk", "mcp", "hono", "lit", "sqlite", "ai-agent", "finland", "forestry", "open-data", "fly-io"]
categories: ["projects", "ai-agents"]
---

# Metsä Advisor: Finnish Forest Intelligence on Three Open APIs

## The Problem

Finland has about 344,000 private forest holders. Most own forest as a secondary asset, not a full-time profession. They all ask the same two questions periodically: what's actually on my property, and what is it worth right now?

The data exists. Three Finnish government agencies publish it under CC BY 4.0:

- **Maanmittauslaitos** — cadastral boundaries via OGC API Features
- **Metsäkeskus** — forest stand inventory via WFS
- **Luke** — regional stumpage prices via PxWeb

The problem is the plumbing. Three protocols, three data shapes, schemas that assume you already know what you're looking at. Metsä Advisor is an agent that does the assembly.

## What It Does

Enter a kiinteistötunnus (e.g. `543-411-6-175`) and an email. The agent:

1. Calls Maanmittauslaitos for the property's boundary polygon
2. Extracts the bbox and queries Metsäkeskus WFS for every stand inside it
3. Picks the price region (Etelä-, Keski-, or Pohjois-Suomi) from the municipality code
4. Pulls current stumpage prices from Luke for that region
5. Writes a structured report: per-stand breakdown, species mix, standing volume, rough timber valuation at current prices, Metsäkeskus cutting recommendations where they exist

The report is delivered to the user's email. A Leaflet map with MML maastokartta tiles renders the property boundary and stand polygons in the browser while the agent works.

## Why It's an Agent

A script would call the three APIs in sequence and dump the numbers. Metsä reasons about them.

It maps the municipality code (first three digits of the kiinteistötunnus) to a Luke price region before deciding which price table to pull. It applies species-appropriate prices to each stand — spruce-dominated → kuusi prices, pine → mänty, birch-or-other → koivu. It flags Metsäkeskus cutting recommendations only when `CUTTINGTYPE` is populated, and translates the numeric codes into Finnish terms with English glosses.

And it refuses to extend beyond the data. No silvicultural advice, no price forecasts, no buyer recommendations — those are scoped out in the system prompt. The output shape is also locked there: property overview, per-stand table, timber valuation, disclaimers. Consistent structure means the agent produces forestry reports, not generic LLM chatter.

## Architecture

Monorepo with npm workspaces:

```text
mcp/    → MCP server, 3 tools, stdio transport
server/ → Hono HTTP + Claude Agent SDK runner
web/    → Lit 3 frontend + Leaflet
agent/  → SYSTEM_PROMPT.md
```

The Agent SDK runs as a subprocess of the Hono server, orchestrating the MCP tools on demand. Server-Sent Events stream tool calls and intermediate reasoning to the UI while the agent works — the frontend isn't guessing progress, it's watching the real event loop.

## Tech Stack

```text
Backend:
  - Hono (HTTP server)
  - Claude Agent SDK (@anthropic-ai/claude-agent-sdk)
  - Model Context Protocol (@modelcontextprotocol/sdk)
  - better-sqlite3 (email-gate ledger)
  - Resend (report delivery)

Frontend:
  - Lit 3 (web components)
  - Leaflet + MML maastokartta WMTS tiles

Data:
  - Maanmittauslaitos OGC API Features — property boundaries
  - Metsäkeskus WFS — forest stand inventory
  - Luke PxWeb — regional stumpage prices

Deploy:
  - Fly.io (Stockholm region, scales to zero)
  - 1GB persistent volume (SQLite + Agent SDK session transcripts)
```

## A Few Interesting Details

### The kiinteistötunnus has two formats

Finnish property IDs come in two shapes: the human-readable `543-411-6-175` and the API-accepted `54341100060175` (zero-padded, concatenated). Users type the first; MML accepts only the second. The tool normalizes transparently:

```ts
export function normalizeKiinteistotunnus(input: string): string {
  const stripped = input.replace(/\s/g, "");
  if (/^\d{14}$/.test(stripped)) return stripped;

  const parts = stripped.split("-");
  if (parts.length !== 4) {
    throw new Error(`Invalid kiinteistötunnus "${input}"`);
  }
  const [kunta, kyla, tila, yksikko] = parts;
  return [
    kunta.padStart(3, "0"),
    kyla.padStart(3, "0"),
    tila.padStart(4, "0"),
    yksikko.padStart(4, "0"),
  ].join("");
}
```

The municipality code falls out of the normalized form as `.slice(0, 3)`, which the pipeline later uses to pick the price region.

### MML's OGC API has a landmine

The API docs mention a `crs` query param to force a coordinate reference system. Passing it returns a 500. The default — WGS84 with `[lon, lat]` GeoJSON order — is what the rest of the pipeline expects, so the fix is to never send the param at all. A one-line comment in code saves the next person a debugging hour:

```ts
// default CRS (WGS84 [lon, lat] GeoJSON order)
// do NOT pass crs param, it triggers 500
```

### Hectares from polygon coordinates

MML returns a GeoJSON polygon but no area value. Computing area from lat/lon needs a projection. For Finnish forest parcels (a few km at most), an equirectangular approximation at the polygon's average latitude is accurate enough:

```ts
const avgLat = ring.reduce((s, [, lat]) => s + lat, 0) / ring.length;
const latRad = (avgLat * Math.PI) / 180;
const mPerLonDeg = (Math.PI / 180) * EARTH_RADIUS_M * Math.cos(latRad);
const mPerLatDeg = (Math.PI / 180) * EARTH_RADIUS_M;
```

Cross-checked against MML's Karttapaikka area display: within 0.1 ha on a 23 ha parcel. Good enough.

### The WFS bbox query is over-inclusive

Metsäkeskus's WFS accepts only bounding boxes, not arbitrary polygons. A 22 ha irregular property has a bbox that covers 30–70 ha of forest, including stands on neighbouring parcels. Clipping to the exact boundary would need either a different API (Metsään.fi, bank-ID gated, owner-only) or client-side polygon intersection against every stand.

The pragmatic fix is to let the agent report the over-inclusion honestly. The system prompt explicitly instructs:

> When the total stand area is larger than the property area, explicitly note this to the user.

The numbers stay useful; the caveat preserves trust. Hiding the gap would make the agent look smarter and be wrong.

### Token budget discipline

A single property can return 14,000+ lines of raw polygon coordinates across its stands. Feeding that to the agent is expensive and useless — the LLM reasons about numeric attributes (volume, age, species), not boundary vertices. The MCP tool strips geometry from the agent payload and pre-computes summary totals:

```ts
// drop geometry for the agent payload — polygon coordinates blow the
// token budget (14k+ lines for a single property) and the agent only
// reasons about numeric/categorical attributes.
const lean = stands.map(({ geometry: _geometry, ...rest }) => rest);
```

The frontend gets the full geometry from a separate channel for map rendering. Same data, two consumers, two shapes.

### Email gate in SQLite

Each email can run three distinct property analyses (lifetime). The table is trivial:

```sql
CREATE TABLE analyses (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT NOT NULL,
  kiinteistotunnus TEXT NOT NULL,
  ip TEXT DEFAULT '',
  created_at INTEGER NOT NULL,
  UNIQUE(email, kiinteistotunnus)
);
```

The `UNIQUE(email, kiinteistotunnus)` constraint enforces "no duplicate analyses" and also lets us count distinct properties per email in a single indexed query. Admin email bypasses both the uniqueness check and the quota — useful for demos without logging out.

### Session resume across Fly deploys

The Agent SDK persists conversation sessions to disk. On Fly, the container restarts on each deploy, which normally wipes that state and destroys any prompt-cache advantage on follow-up questions. Mounting a 1GB persistent volume at `/data` — with the Agent SDK's session directory at `/data/.claude/projects` and the SQLite gate at `/data/db/metsa.db` — survives restarts.

A user asking a follow-up after a redeploy hits the cache on the long system prompt and tool-definition preamble. For an agent with a 3KB system prompt and three MCP tools, that's the difference between cents and dimes per follow-up.

### Prompt-injection defence

User input (kiinteistötunnus, follow-up questions) is embedded inside an XML-ish framing that the agent uses to distinguish user turn from system boundary. A user typing `</user>` could try to close the frame and inject new instructions. The server escapes angle brackets before passing user text to the SDK:

```ts
private sanitize(s: string): string {
  return s.replace(/</g, "&lt;").replace(/>/g, "&gt;");
}
```

Low-effort defence, but it forecloses the entire category of structural injection without depending on model-level defences holding up.

### Tests hit real APIs

MCP tool tests are live integration tests against real government APIs. No mocks. If Metsäkeskus's WFS schema changes (it has, twice during development) the tests fail loudly instead of hiding it behind a fake fixture that silently diverges from reality. Tests that need `MML_API_KEY` use `describe.skip` when the key is absent, so CI without secrets still passes the rest.

Real-API tests are slow and flaky. They also catch the bugs that matter — schema drift, auth changes, 500s on quiet params. For a project whose entire value depends on three external APIs behaving consistently, mocks would have been lying to me.

## Live Demo

Live at [metsa.innalab.com](https://metsa.innalab.com). Try `543-411-6-175` — 22.8 ha in Nurmijärvi, mixed pine/spruce with several Metsäkeskus cutting recommendations on file.

Built as part of [AI Agent Development practice](https://dev.innalab.com).

Source on [GitHub](https://github.com/avlihachev/metsa).
