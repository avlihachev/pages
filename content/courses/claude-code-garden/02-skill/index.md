---
canonicalURL: "https://dev.innalab.com/learn/skill/"
title: "Module 2: Building a Skill — Teaching Claude Your Domain"
date: 2026-03-29
draft: false
tags: ["claude-code", "skill", "ai-agent", "course"]
categories: ["Courses"]
description: "Create a Claude Code skill that encodes gardening knowledge (frost dates, plant lifecycles, container growing) using parametric logic instead of hardcoded rules."
series: ["Building an AI Agent with Claude Code"]
weight: 3
ShowToc: true
---

Ask Claude Code "when should I start tomato seedlings?" and you'll get a perfectly reasonable, perfectly useless answer. "Generally 6-8 weeks before your last frost date." Thanks, Google could have told me that.

The problem isn't that Claude is wrong. It's that Claude doesn't know *you*. It doesn't know you're in Kokkola, Finland at 63°N, that your last frost date is June 15, that you grow in containers on a south-facing balcony, and that you have grow lights for the long dark spring. With that context, the answer changes completely, and it changes differently for every variety in your collection.

A skill fixes this. It gives Claude structured domain knowledge that loads automatically, every time. After this module, your Claude Code will know more about your garden than most gardening forums.

## What Is a Skill?

A skill is a directory of markdown files that Claude Code reads when invoked. That's it. No code, no API, no compilation. Just markdown.

```
~/.claude/skills/garden/
├── SKILL.md              # the brain — logic, rules, instructions
├── profile.md            # your personal parameters
├── plants.md             # your current plant list with statuses
├── journal.md            # season log
└── references/
    ├── timing.md          # germination temps, sowing lead times
    ├── containers.md      # volumes, materials, drainage
    ├── soil.md            # substrates, mixes, pH ranges
    ├── fertilizers.md     # NPK ratios, feeding schedules
    ├── pests.md           # identification and treatment
    ├── pruning.md         # pruning and training by crop
    └── common-mistakes.md # frequent errors and how to avoid them
```

Two types of files:

- **Universal** (`SKILL.md` + `references/`): the skill engine. Same for everyone. Contains gardening logic, formulas, and reference data.
- **Personal** (`profile.md`, `plants.md`, `journal.md`): your data. Created through an onboarding dialog the first time you use the skill.

This separation matters. You can share the skill (or publish it on GitHub) without sharing your personal data. And when the skill logic improves, you update the universal files without losing your garden history.

## The Key Idea: Parametric Logic

Most gardening tools work like a calendar: "Plant tomatoes in May." But May means completely different things in Helsinki vs. Houston. Every useful recommendation depends on local conditions.

Our skill doesn't hardcode dates. It uses formulas:

```
sow_indoors = last_frost_date - N weeks
transplant_outdoor = last_frost_date + buffer
hardening_start = transplant_outdoor - 14 days
```

Where `N` varies by crop (tomatoes need 8 weeks, basil needs 4), and `last_frost_date` comes from your profile. Move to a different city? Update one field in `profile.md`, and every recommendation recalculates automatically.

That gap between "useful tool" and "toy" is exactly what parametric logic closes.

## Let's Build It

### Step 1: Create the Skill Directory

Open Claude Code and tell it what we're building:

> "Create a gardening skill in ~/.claude/skills/garden/. It should have a SKILL.md for logic, a references/ directory for domain knowledge, and support personal files (profile.md, plants.md, journal.md) that get created through an onboarding dialog."

Claude Code creates the directory structure. Review it, make sure the separation between universal and personal files is there.

```bash
mkdir -p ~/.claude/skills/garden/references
```

Nothing fancy yet. The intelligence lives in what we put inside.

### Step 2: Write SKILL.md, the Brain

This is the core. SKILL.md tells Claude Code *how to behave* when the skill is active. Not a database. Instructions.

The key sections:

**How this skill works.** On every invocation, read profile.md, plants.md, journal.md. If profile doesn't exist, run onboarding.

**Onboarding.** A multi-step dialog that creates your profile. Claude Code asks one question at a time:
1. Where do you grow? (city, country, latitude)
2. What's your last frost date? (or hardiness zone, Claude can estimate if you don't know)
3. What's your setup? (containers, raised beds, greenhouse, open ground)
4. Do you have grow lights? Indoor space?
5. Experience level?

Each answer feeds into `profile.md`. This file becomes the parameter set that powers all recommendations.

**Seasonal logic.** The formulas. Expressed as rules, not code:

- *"To calculate indoor sow date: look up the crop's lead time in references/timing.md, subtract that many weeks from the user's last frost date in profile.md."*
- *"Hardening always starts 14 days before planned transplant date."*
- *"If the user has grow lights, seedling phase can start 1-2 weeks earlier."*

**Plant management.** How to track plants through their lifecycle:

`planned → sown → germinating → seedling → hardening → outdoor → harvesting → done`

When you tell Claude "I sowed the Sungold tomatoes today," it updates plants.md and adds a journal entry. When you ask "what should I do this week?", it checks every plant's status against the seasonal formulas and tells you what's next.

**Recommendations engine.** On each invocation, Claude cross-references:
- Current date vs. seasonal formulas
- Plant statuses from plants.md
- Recent journal entries
- References for specific advice

The prompt to Claude Code for SKILL.md is detailed. You're not asking Claude to "write a skill", you're specifying the behavior:

> "Write SKILL.md for the garden skill. It should: read profile/plants/journal on each invocation, trigger onboarding if profile doesn't exist, use parametric seasonal logic relative to last_frost_date from profile, track plant lifecycle states, maintain a reverse-chronological journal, and generate recommendations by cross-referencing plant status with seasonal formulas."

Plan mode is your friend here. Claude Code will propose the structure, you review, adjust, approve. Iterate until SKILL.md captures the behavior you want.

### Step 3: Reference Library

The references/ directory is the skill's knowledge base. Seven files, each covering a domain:

**timing.md.** The most critical reference. Contains:
- Germination temperature ranges by crop family (tomatoes: 20-25°C, peppers: 25-30°C)
- Indoor sow lead times (tomatoes: 8 weeks, basil: 4 weeks, peppers: 10 weeks)
- Grow light parameters (16h/day for seedlings, 25-30cm height, 6500K spectrum)
- Universal hardening schedule (2-week template: day 1-3 shade 2h, day 4-6 partial sun 3h...)

**containers.md.** Recommended volumes by crop (tomatoes: 15-25L, herbs: 3-5L), materials (fabric vs. plastic vs. clay), drainage requirements, insulation for cold nights.

**soil.md.** Substrate types and recipes. Seed-starting mix vs. potting mix. Components: perlite, vermiculite, peat, coir. pH ranges by crop. When to replace substrate.

**fertilizers.md.** NPK ratios by growth stage. Seedling phase: dilute balanced feed. Flowering: higher P and K. Container-specific: leaching, salt buildup, flush schedule every 4-6 weeks.

**pests.md.** Aphids, whitefly, spider mites, fungus gnats. Plus diseases: damping off, powdery mildew, blossom end rot. Symptoms, prevention, organic treatment.

**pruning.md** and **common-mistakes.md** round out the library: pruning and training notes by crop, and a checklist of the errors that trip up most container growers.

For each reference, the prompt is straightforward:

> "Write references/timing.md for the garden skill. Include germination temperature ranges for tomatoes, peppers, chili, basil, and herbs. Include indoor sow lead times as weeks before last frost. Include grow light parameters. Include a universal 2-week hardening schedule. All timing should be relative, no hardcoded dates."

Claude Code generates comprehensive reference files. Review them, check the temperature ranges, make sure the sow lead times make sense for your varieties. This is domain knowledge, so you need to sanity-check it.

### Step 4: User Files (Personalization)

The personal files get created through the onboarding dialog, but they need a defined structure. Tell Claude Code what they should look like:

**profile.md:**
```markdown
# Garden Profile

## Location
- City: Kokkola
- Country: Finland
- Latitude: 63.84°N
- Hardiness zone: 4b
- Last frost date: June 15
- First frost date: September 20

## Growing conditions
- Format: containers
- Indoor space: south-facing windowsill + grow lights
- Grow lights: LED full spectrum, 16h/day
- Outdoor space: south-facing balcony (6m²)

## Experience
- Level: intermediate
- Years: 3
```

**plants.md.** Starts empty, fills up as you add plants:

```markdown
# My Plants — 2026 Season

## Active

### Tomato — Sungold F1
- Type: indeterminate cherry
- Container: 20L fabric pot
- Sown: 2026-04-20
- Status: seedling
- Notes: strong growth under lights
```

**journal.md.** Reverse chronological, grows with every action:

```markdown
# Garden Journal — 2026

## 2026-04-20

### Sowed tomato seedlings
- Action: sowed
- Plants: Sungold F1, Black Cherry, San Marzano
- Details: 1 seed per cell, seed-starting mix, 22°C, under grow lights
- Note: San Marzano seeds are 2 years old, added extra seeds as insurance
```

## Testing the Skill

This is where it clicks. Try asking Claude Code the same question with and without the skill.

**Without the skill:**

> "When should I start tomato seedlings?"

Claude: "Most gardeners start tomato seedlings 6-8 weeks before their last frost date. In most regions, this means late February to early April..."

Generic. Could be anyone, anywhere.

**With the skill (after onboarding):**

> "When should I start tomato seedlings?"

Claude: "Based on your profile (Kokkola, last frost June 15), here's the schedule for your tomatoes:

- **San Marzano** (10-week lead time): Start **April 6** under grow lights
- **Sungold F1** (8-week lead time): Start **April 20** under grow lights
- **Black Cherry** (8-week lead time): Start **April 20** under grow lights

Hardening starts June 1 for all varieties. Begin with 2 hours of shade on the balcony, increase gradually. Your journal shows last year's Sungold did well with an April 18 start, so April 20 is safe."

Same question, completely different answer. Claude now knows your location, your varieties, your setup, and your history.

## What We Built

A skill is the simplest way to make Claude Code useful for a specific domain. No code, no servers, no APIs. Structured markdown that Claude reads every time.

What matters:
- Parametric logic: formulas relative to your conditions, not hardcoded calendars
- Separation of concerns: universal knowledge (shareable) vs. personal data (private)
- Living data: plants.md and journal.md evolve as your season progresses

But there's a limitation: the skill knows rules, not reality. It knows tomatoes need 20-25°C to germinate, but it doesn't know that tonight's forecast is -3°C. It can calculate your hardening start date, but it can't tell you if this week is actually warm enough to start.

For that, we need real-time data. That's what the MCP server provides.

In [Module 3](/courses/claude-code-garden/03-mcp/) we fix this by giving Claude access to weather forecasts, frost risk, soil temperature, and daylight data. All from free APIs, no keys required.

## Source Code

[Garden Skill on GitHub](https://github.com/avlihachev/claude-skill-garden)

<!-- {{< youtube VIDEO_ID_HERE >}} -->
