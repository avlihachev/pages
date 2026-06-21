---
title: "Mening Skill: language practice inside your AI agent"
description: "An open-source SKILL.md for OpenClaw and Hermes that runs daily language-learning writing practice — with long-term error-pattern memory — through your Mening account."
date: 2026-06-21
tags: ["ai-agents", "openclaw", "hermes", "agent-skills", "mening", "curl", "jq"]
categories: ["projects", "ai-agents"]
---

# Mening Skill: language practice inside your AI agent

## The Problem

Mening is a language-learning app with a specific mechanic: write one sentence a day in your target language, get it corrected by a model, and watch which mistakes recur over time. The long-term error-pattern memory is the point — it is what separates the app from a one-shot grammar checker. But the app has no agent interface. If you work inside Hermes or OpenClaw all day, reaching for a separate app breaks the flow, and the daily habit dies. The skill closes that gap: your agent becomes the interface, while the app stays the backend and the brain.

## What It Does

Three commands, one account:

- **`today`** — fetches the day's writing prompt and shows any open error patterns to keep in mind while writing.
- **`submit <text>`** — sends your sentence, gets the correction back, and surfaces which recurring patterns the submission touched.
- **`trends`** — shows your active error patterns with occurrence counts and last-seen dates, so you can see what the app has learned about your writing over time.

The differentiator is the memory. A single correction is cheap; any grammar checker does it. What Mening tracks is which errors you repeat across weeks. When you run `trends`, you are reading a longitudinal record of your specific weak spots — not a generic rulebook. The skill exposes that record directly in your agent session.

Example session from a real Swedish practice run:

```text
you:   what's my writing topic today?
agent: Beskriv en maträtt som du gillar att laga hemma och vilka ingredienser som ingår i den.
       (Describe a dish you like to cook at home and the ingredients in it.)

you:   what mistakes do I keep making?
agent: your recurring error patterns:
       - function_words · article_missing — 4×
       - morphology · word_form — 3×
       - lexis · word_choice — 3×
       - orthography · spelling — 2×
```

## How It Works

One `SKILL.md` file covers both OpenClaw and Hermes — same document, same syntax, both runtimes pick it up. The skill teaches the agent three things: how to authenticate (Bearer token from the user's Mening account), which `/api/v1` endpoints to call, and how to format the output for a terminal session.

The transport is a curl/jq helper that ships alongside `SKILL.md`. It hits the same REST API the web and iOS clients use — no separate backend, no webhooks, no polling. The agent reads `/api/v1/today`, posts to `/api/v1/submissions`, and reads `/api/v1/trends`. That's the full surface area.

Account-backed means the data is real: corrections persist, patterns accumulate, streak counts. Switching from the web app to the agent and back leaves no seam. The agent is just another client.

## Links

- Source: [github.com/avlihachev/mening-skill](https://github.com/avlihachev/mening-skill)
- Background: [The full story](https://lihachev.pro/posts/agent-or-invisible/)
- App: [mening.app](https://mening.app)
