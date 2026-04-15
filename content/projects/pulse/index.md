---
title: "Pulse: Reddit Community Discovery Agent"
description: "AI agent that finds where your customers talk on Reddit and generates ready-to-post reply drafts. Built with Claude Agent SDK and MCP."
date: 2026-04-14
tags: ["claude-agent-sdk", "mcp", "hono", "lit", "sqlite", "ai-agent", "reddit"]
categories: ["projects", "ai-agents"]
---

# Pulse: Reddit Community Discovery Agent

## The Problem

Every business wants to be where their customers are. On Reddit, that means finding the right subreddits, the right threads, and writing replies that don't get downvoted into oblivion. Doing this manually takes hours — scanning dozens of communities, reading thread context, crafting replies that actually add value.

Pulse does this automatically.

## What It Does

Enter your website URL and email. The agent takes over:

1. **Parses your site** — reads your URL, extracts what your business does, who it serves, what makes it different
2. **Builds a business profile** — product, audience, value proposition, keywords
3. **Searches Reddit** — finds relevant subreddits and fresh threads via Reddit JSON API
4. **Generates draft replies** — writes contextual, ready-to-post replies for each thread

The output is a full report: matched subreddits with relevance scores, threads with context, and draft replies you can review and post. Everything saves to a PDF.

## Why This Is an Agent, Not a Script

A script would search Reddit for keywords and return results. Pulse reasons about your business first — it reads your website, understands your product positioning, identifies your audience, and then searches for conversations where that audience is asking relevant questions.

The reply drafts aren't templates. The agent reads the actual thread, understands the conversation context, and writes replies that reference specific points from the discussion. Different threads get different replies.

Two-phase pipeline: Business Intelligence → Reddit Discovery. The first phase feeds the second — the agent's understanding of your business shapes how it searches and what it writes.

## Tech Stack

```text
Backend:
  - Hono (HTTP server)
  - Claude Agent SDK (orchestration + OAuth bridge)
  - MCP (4 tools: parse_website, search_reddit, analyze_thread, check_subreddit)
  - SQLite (business profiles, scan results, shared subreddit knowledge)
  - pdfkit (report generation)
  - Resend (email delivery)

Frontend:
  - Lit 3 (Web Components)
  - Phased loading animation (5 stages while agent works)
```

## A Few Interesting Details

**Shared knowledge base.** Every scan adds to a subreddit knowledge base — rules, posting guidelines, audience type. This grows across all clients, so the agent gets smarter about which communities match which business types over time.

**Reddit JSON API.** No scraping, no Google intermediary. Reddit exposes `.json` endpoints on every page. Free, no API key required, no captcha. `search_reddit` uses Reddit's native search; `analyze_thread` reads thread `.json` directly.

**No streaming yet.** The full pipeline takes ~60 seconds. Currently a single POST request with a loading animation. SSE streaming is a future improvement.

## Live Demo

Pulse is live as a demo on [dev.innalab.com/demos/pulse](https://dev.innalab.com/demos/pulse). Built as part of [AI Agent Development practice](https://dev.innalab.com).

Source on [GitHub](https://github.com/avlihachev/pulse).
