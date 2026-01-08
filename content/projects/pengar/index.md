---
title: "Pengar: Why I Built Yet Another Finance App"
description: "Finance app that tracks by life areas, not categories. Multi-currency, project-based savings, selective tracking. Built with Lit and Supabase."
date: 2025-01-08
tags:
  [
    "typescript",
    "nodejs",
    "lit.dev",
    "web-components",
    "supabase",
    "postgresql",
    "vite",
    "personal-finance",
  ]
categories: ["projects", "development"]
---

# Pengar: Why I Built Yet Another Finance App

## The Graveyard of Abandoned Apps

I've tried every finance app. YNAB, Mint, spreadsheets, random apps with promising reviews. The pattern is always the same: two weeks of enthusiasm, meticulous categorization of coffee purchases, then silence.

The apps weren't broken. The whole approach was.

These tools are built for accountants. They assume you want to track every transaction, feel guilty when you "overspend," and spend evenings reconciling bank balances.

I just wanted clarity. Not a second job.

## The Question That Changed Everything

Traditional budgeting asks: _which account did money leave?_

I started asking: _which part of my life did I invest in?_

Categories like "Food & Dining" or "Entertainment" tell you _what_ you bought. They reveal nothing about your actual life.

When I reorganized tracking around **Interest Areas** — Health, Learning, Family, Side Projects — I found $200/month going to hobbies I'd lost interest in. Traditional categories had hidden this completely.

Same transactions. Different insight.

## What I Built

**Pengar** (Swedish for "money") has four ideas:

**Interest Areas** instead of categories. Your structure, not some accountant's taxonomy.

**Projects** instead of budgets. "Save $3,000 for Japan" beats "Don't spend more than $500 on entertainment." Progress bars filling up, not warnings turning red.

**Multi-currency** as default. Salary in EUR, spending in SEK, savings in USD. Everything converts automatically. If you've lived in multiple currencies, you know this pain.

**Selective tracking.** Log what matters, skip what doesn't. No reconciliation guilt.

## Tech Stack

```
Frontend:
  - Lit (Web Components, not React)
  - Shoelace (design system)
  - Vite

Backend:
  - Node.js + Express
  - TypeScript
  - Zod validation

Database:
  - PostgreSQL via Supabase
  - Row Level Security
  - Docker for local dev
```

Lit over React was deliberate. Web Components feel closer to the platform, smaller bundles, and honestly — refreshing to build without React ecosystem complexity.

## The Philosophy

The $4 coffee doesn't change how you live. The $400/month on a hobby you're not sure about? Worth tracking.

Most finance apps fail because they demand too much. Pengar asks for minimum: transactions that matter. Nothing else.

No bank sync is a feature, not a limitation. Manual entry means intentional tracking. That's the point.

## What I Learned

Building this reminded me why side projects work:

**Constraints breed creativity.** No bank sync forced intentional design.

**Simple beats comprehensive.** Every feature adds friction.

**Solve your own problem.** Best product validation there is.

---

Live at [pengar.money](https://pengar.money). Free tier, $3/month for unlimited.

Built with Lit, Shoelace, Express, Supabase. Hosted on fly.io and Cloudflare Pages.
