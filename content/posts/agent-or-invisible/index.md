---
title: "Ship a skill or be invisible to the agent"
description: "Self-hosted agents like OpenClaw and Hermes can only act on your app if you ship them a skill. Here's the ~40-line one I wrote for Mening, and why every app now needs one."
date: 2026-06-21
tags: ["ai-agents", "openclaw", "hermes", "agent-skills", "mening", "developer-tools"]
categories: ["posts", "ai-agents"]
ShowToc: true
---

your users are starting to live inside an agent. if your app isn't a skill it can load, you don't exist to it.

More of the day now starts in one place. A person opens OpenClaw or Hermes, types what they want done, and the agent figures out how to do it. If you've shipped a skill, it calls your product. If you haven't, it reaches for whatever competitor did, or it skips the job and moves on. You never find out it happened. There's no bounce on your analytics for a request that was never made to you.

## The agent is the new front door

OpenClaw and Hermes didn't go viral because people wanted another chat window. They went viral as the place tasks get run from. OpenClaw is past 100k stars on GitHub; Hermes is somewhere around 140k. Those aren't demo numbers. People are wiring these things into their actual work and letting them act.

So the integration you used to weigh (is it worth building, will anyone use it) isn't a decision anymore. It's the price of being callable at all. The agent already expects a way to reach the tools its user mentions. If you're not one of them, you're a name the user has to remember to leave the agent and go visit by hand. Most won't.

## What it takes is small

The whole thing is one `SKILL.md`, around forty lines.

The header carries everything both platforms need to load it: `prerequisites` for Hermes (the env var with the API token, plus `curl` and `jq`), `metadata.openclaw` for OpenClaw (same requirements in its own shape). The body is plain instructions in English telling the agent when to call which command and how to read the JSON back. Underneath sits a short shell helper: `curl` with a bearer token, `jq` to build the request. That's it. No SDK, no client library, no new service to host. Mening already had the API; the skill is a thin description of how to drive it.

What that buys, from inside the agent:

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

That last answer is the whole reason Mening exists. Correction is a commodity; every tool gives it away. Remembering which mistakes you repeat, over weeks, is not. And now an agent can read those patterns back to the user mid-conversation, without anyone opening the app.

## The honest part

I'll be straight about who's actually running these agents: developers. The OpenClaw and Hermes crowd is people who self-host their tooling and write shell helpers for fun. That is not where my language learners are. Nobody learning Swedish is reaching for a self-hosted agent framework to ask for today's writing prompt. So no, this didn't open some new flood of signups, and I didn't expect it to.

I built it anyway, and not out of optimism. A few missed signups I could live with. What I couldn't live with is being unreachable from a surface people increasingly act through. A tool the agent can't call is a tool the agent forgets exists, and that gap doesn't stay the same size. It widens as more of the day moves behind the agent. Cheap to close now, expensive to notice later.

Forty lines against that math is not a hard call.

## Close

The skill is open and MIT-licensed: [github.com/avlihachev/mening-skill](https://github.com/avlihachev/mening-skill). The product it talks to is [mening.app](https://mening.app). Copy the pattern for your own app if it's useful. The API is the only part that's yours.

The thesis hasn't moved. Your users are starting to live inside an agent. If your app isn't a skill it can load, you don't exist to it.
