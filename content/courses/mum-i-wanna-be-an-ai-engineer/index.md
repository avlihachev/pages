---
title: "Mum, I wanna be an AI Engineer: the production half of building with agents"
description: "A course for engineers who already build agents and want the production half of the job: inference economics, enforced harnesses, evals, observability, retrieval, security, serving. Graded exercises, a tutor bounded by failing tests, lessons that re-check themselves."
date: 2026-10-23
draft: false
categories: ["courses"]
tags: ["ai-agents", "claude", "llm-engineering", "evals", "observability", "rag", "python"]
ShowToc: true
ShowReadingTime: false
cover:
    image: "cover.png"
    alt: "Mum, I wanna be an AI Engineer: a software factory, built as a course"
    relative: true
---

The course lives at **[factory.lihachev.pro](https://factory.lihachev.pro)**. This page is the map.

## Who it is for

Engineers who already run agents and want to be the person who runs the factory: the one who knows what a token costs, which harness rule stops an agent from merging garbage, how to tell a regression from noise, and what blocks a change from reaching production.

## What it is

Eight modules of lessons with graded exercises. Every exercise is a Python package with a starter that fails on purpose, tests that decide pass or fail, and a `miubais` CLI that starts, checks and resets it. A tutor built on the Agent SDK explains a failing test from its output and your diff; a leak filter keeps it from handing out the solution.

The repository is also the proof of the method: lessons are pinned and dated, numbers in the text come from recorded runs, and a freshness pipeline opens review branches when a vendor page or a price moves.

## Modules

| # | Module | What you leave with |
|---|--------|---------------------|
| M0 | Inference economics | Tokens, KV cache, prompt caching, cost levers, structured outputs, context management |
| M1 | Enforced harness | Hooks, permissions, subagents, durable sessions, failure taxonomy and caps |
| M2 | Evals as engineering | Error analysis, judges and agreement, datasets, agent evals, CI gating |
| M3 | Observability and cost | Traces and spans, GenAI semantic conventions, self-hosted Langfuse, routing, latency |
| M4 | Retrieval as a component | When not to build RAG, chunking, hybrid search, reranking, retrieval metrics, citations |
| M5 | Security and compliance | Threat model for agent factories, indirect injection and provenance, permissions and secrets, remote MCP and OAuth, red teaming with promptfoo, the EU AI Act |
| M6 | Serving and platform | Open-weight landscape and licences, serving internals, running locally, Kubernetes for inference, TTFT/TPS/cost, fine-tuning |
| M7 | Capstone: pre-review | A read-only review agent over a Go, TypeScript, proto and migrations repository |

Each module ends with an interview-angle lesson: the questions a hiring panel asks about that part of the job, and the answers the exercises prepare.

## Start

```bash
git clone https://github.com/avlihachev/miubais
cd miubais
uv sync
uv run miubais doctor
uv run miubais start m0-cost-lab
uv run miubais check m0-cost-lab
```

The last command fails at stage 01: the starter raises on purpose, and the lessons tell you what to build.

## Book

The lessons are also a book, [Mum, I wanna be an AI Engineer](/projects/mum-i-wanna-be-an-ai-engineer/), in paperback and Kindle. The book is reading-only: code appears as illustration with its output, the exercises stay on the site.

## Source

- [github.com/avlihachev/miubais](https://github.com/avlihachev/miubais): code under MIT, lessons under CC BY-NC-SA 4.0.
