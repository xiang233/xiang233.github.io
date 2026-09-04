---
title: Canvas AI Agent
date: 2026-01-05 22:34:39
updated: 2026-08-31
tags:
---

{% asset_img "01_landing_light.png" "Canvas Student Agent" %}

## Overview

At the **Skandalaris Hackathon 2025 (AI Track)**, my teammates **DeYu Zhang**, **Mark Song**, and I built **Canvas AI Agent**, an LLM-powered assistant that answers questions, summarizes materials, and surfaces deadlines from contextual Canvas course data. We earned a **Runner-Up Award (Top 5 overall)** and $500 in prizes.

Since then I have kept working on my own fork, turning a demo that worked into a system I can prove works.

## The hackathon build

React + TypeScript chat frontend over WebSocket streaming; FastAPI backend with async handling, Azure OpenAI (`gpt-4.1-mini`) for query understanding and tool invocation, OpenAI Vector Stores for semantic search over course materials, and the Canvas LMS API for student-accessible resources.

## Making it measurable

The demo worked, but nothing about it was measured. I built the evaluation first and let it tell me what to fix: a factorial experiment over knowledge-base access and question wording, graded by a **different model family** (Claude judging GPT output) using a rubric that separates *fabrication* from *false refusal*.

| | Without KB | With KB |
|---|---|---|
| Fully correct | **0 / 18** | **17 / 35** |
| Grounded in course material | 0 | **25 / 35** |
| Refusals | 17 | 3 |
| Fabricated content | 0 | 3 |

That asymmetry is the point. Without retrieval the agent politely refuses — harmless-looking in a single metric; with retrieval it answers correctly far more often but occasionally invents a source. One number would have hidden both failure modes.

Real bugs fell out once the harness existed. Asking "who teaches this course?" burned fifteen identical API calls and 460K input tokens before giving up: the tool advertised `include=teachers`, requested it, then dropped the field while formatting. A model that asks for teachers and receives none concludes it passed the argument wrong — and retries forever. Separately, Canvas's announcements endpoint silently defaults to a 14-day window: 0 announcements for a concluded course, 18 with an explicit semester range.

Both share one root cause: **the model cannot question what it cannot see.** API contracts now live in tool descriptions, errors surface verbatim (`HTTP 404 ... /quizzes`, not `Resource not found`), list rendering always reports counts including zero, and identical repeated calls are short-circuited. The agent now reads the raw 404 and pivots on its own — no prompt rule needed.

{% asset_img "03_tool_trace.png" "Tool trace" %}

The same question that once looped fifteen times now resolves in two calls, with each step and its latency visible as it streams.

## The interface

{% asset_img "02_course_picker.png" "Course scope selector" %}

Course scope narrows the agent to one class; a **KB** badge marks which courses have a knowledge base built from their materials.

{% asset_img "04_course_only_rag.png" "Course-only grounding" %}

A grounding toggle switches to **course-only** mode, and answers are then labeled by provenance — *grounded in course materials*, or explicitly not found in them. That label is the interface counterpart of the attribution classes the RAG evaluation measures.

{% asset_img "05_dark_theme.png" "Dark theme" %}

## Other engineering

- **Bounded multi-turn memory** — compaction cut an eight-turn session from **438K to 166K input tokens (62%)**, recall verified intact.
- **Per-session agents** — concurrent users run in parallel with isolated context, over SSE and WebSocket sharing one event model. Worth measuring first: under ReAct with forced tool calling the model emits **zero** free-text deltas, so there is nothing token-level to stream.
- **Read-only as a mechanism, not a convention** — every tool carries a `side_effect` attribute filtered at build time, guarded by a CI assertion. Verified by re-enabling a write tool: it still cannot get in.
- **76 deterministic cases across seven suites** on every pull request, Python 3.11 and 3.13, no API keys and no network. Proven by planting a phantom tool name and watching CI go red. The expensive LLM evals stay manual: a low-change repo produces identical scheduled runs at real cost for no new signal.

## Links

- [My fork](https://github.com/xiang233/canvas_ai) — eval harness, RAG ablation, CI, streaming, MCP server
- [Team repository](https://github.com/Deyu-Zhang/canvas_ai) — hackathon build
- [Cloudflare Workers version](https://github.com/xiang233/cf_ai_canvas_agent) — Workers AI + SQLite Durable Objects
- [Demo slides](https://docs.google.com/presentation/d/1RhQeK-0zsjlltyZdZD8q8MCvjk9Y4iDnbsi4hZDU2Ak/edit?usp=sharing)
