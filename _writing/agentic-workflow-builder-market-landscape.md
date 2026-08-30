---
layout: post
title: "What the agentic workflow builder market looks like."
date: 2026-08-17
tags: [Tech, AI]
draft: true
---

I've been building a proof of concept for an agentic workflow builder (a tool for scoping, building, and testing agentic workflows without requiring in-house AI expertise). Before going further, I looked at what already exists: SimplAI, n8n, Zapier, and LangGraph. They're not competitors in the way the category name suggests — each draws a different line around how much of a workflow is code versus clicked together, and how much of the decision-making inside a step is fixed in advance versus left to a model at run time.

## What users need

The four sit on a spectrum of control versus speed. Zapier is for ops, marketing, and sales people who want something wired together in minutes without touching code. n8n is for technical teams who want that same speed with an escape hatch into real code once a workflow gets complicated. LangGraph is for developers who want full control over how an agent reasons, branches, and retries, and are willing to write code to get it. SimplAI is for enterprises in regulated industries who need governance, compliance, and an IT team to own the thing.

One need shows up across all four regardless of persona: as a workflow gets more autonomous, the person building it has to trust its output without reading every decision it made. That means real testing, not just a working demo.

<figure class="awb-figure">
  <svg viewBox="0 0 640 380" role="img" aria-labelledby="awb-spectrum-title awb-spectrum-desc" style="width:100%;height:auto;font-family:var(--font-sans);">
    <title id="awb-spectrum-title">How the four products compare on build mechanism and AI centrality</title>
    <desc id="awb-spectrum-desc">A scatter plot. X axis: clicked together to written in code. Y axis: AI bolted on to AI is the entire point. Zapier sits low and left. n8n sits mid-height and mid-left. SimplAI sits high and left. LangGraph sits high and right.</desc>

    <line x1="70" y1="40" x2="70" y2="320" stroke="var(--color-hairline)" stroke-width="1.5" />
    <line x1="70" y1="320" x2="600" y2="320" stroke="var(--color-hairline)" stroke-width="1.5" />

    <text x="70" y="345" fill="var(--color-muted)" font-size="13">Clicked together</text>
    <text x="600" y="345" fill="var(--color-muted)" font-size="13" text-anchor="end">Written in code</text>
    <text x="335" y="368" fill="var(--color-muted-strong)" font-size="13" text-anchor="middle">Build mechanism</text>

    <text x="55" y="315" fill="var(--color-muted)" font-size="13" text-anchor="end">AI bolted on</text>
    <text x="55" y="50" fill="var(--color-muted)" font-size="13" text-anchor="end">AI is the entire point</text>
    <text x="30" y="180" fill="var(--color-muted-strong)" font-size="13" text-anchor="middle" transform="rotate(-90 30 180)">AI's role</text>

    <circle cx="150" cy="270" r="7" fill="var(--color-accent)" />
    <text x="150" y="248" fill="var(--color-ink)" font-size="15" text-anchor="middle">Zapier</text>

    <circle cx="280" cy="175" r="7" fill="var(--color-accent)" />
    <text x="280" y="153" fill="var(--color-ink)" font-size="15" text-anchor="middle">n8n</text>

    <circle cx="195" cy="80" r="7" fill="var(--color-accent)" />
    <text x="195" y="58" fill="var(--color-ink)" font-size="15" text-anchor="middle">SimplAI</text>

    <circle cx="540" cy="65" r="7" fill="var(--color-accent)" />
    <text x="540" y="43" fill="var(--color-ink)" font-size="15" text-anchor="middle">LangGraph</text>
  </svg>
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">SimplAI is the odd one out: enterprise, click-configured like Zapier, but built around AI as the whole premise like LangGraph — closer to an agent platform wearing a low-code UI than a click-together automation tool.</figcaption>
</figure>

## What these solutions offer

| | Zapier | n8n | LangGraph | SimplAI |
|---|---|---|---|---|
| **Fundamentally is** | No-code SaaS automation | Node-graph automation, code escape hatches | Code framework for agent orchestration | Enterprise agentic AI platform |
| **Built by** | Clicking | Clicking + optional code | Writing code | Clicking (enterprise-configured) |
| **AI's role** | Bolted on, now maturing into agents | Native, opt-in per workflow | The entire point | The entire point, enterprise-wrapped |
| **Handles loops/cycles** | No | Limited | Yes, natively | Unclear from public docs |
| **Testing/eval** | Single dry run, no dataset | Dataset-driven evals, built in | Mature, but a separate product (LangSmith) | Named as a pillar, thin public detail |
| **Pricing shape** | Per task | Per execution (or free, self-hosted) | Free core, paid platform + eval usage | Per credit/run, enterprise custom |

A Zap is a trigger plus one or more actions. AI used to mean a single bolted-on step; Zapier's newer "Agents" take a plain-English goal and decide which tools to call across multiple steps, with an approval gate where a reviewer can edit the payload, not just approve or reject it. Zapier MCP exposes that same catalog of app integrations as MCP tools, so an external AI client can call them directly instead of going through the Zaps UI. Testing stops at a single dry run against sample data.

n8n is a visual node graph where any node can be swapped for raw JavaScript or Python. AI Agent nodes, RAG, and MCP support are built into the canvas natively, so a workflow's agentic-ness is opt-in per node rather than the whole point of the tool. It also has the most developed testing story of the no-code options: an Eval Trigger runs a workflow against a dataset and scores outputs against configurable metrics.

LangGraph is a code-first library where nodes and edges operate over a persistent state object — the only one of the four built for cycles, where an agent tries something, evaluates its own output, and loops back with adjusted state. Testing lives in a separate product, LangSmith, which is mature but adds a second tool and a second learning curve. The core library is free; the platform and eval products around it aren't. The learning curve is also the steepest of the four.

SimplAI markets itself as an "Agentic AI Operating System" for regulated enterprises moving past AI pilots into production: an Agent Builder, Workflow Builder, knowledge base, evaluation/tracing layer, and an MCP Gateway for connecting agents to enterprise tools, wrapped in a low-code enterprise UI. It's the least independently documented of the four — most of what's available is vendor material rather than real community usage, so its evaluation claims are hard to verify against actual mechanics.

Two patterns hold once you strip the marketing. First, learning curve and agentic power move together, and testing maturity is the widest gap in the market — only n8n and LangGraph, via LangSmith, have real dataset-driven evaluation, and nobody has made evaluating an agentic step as easy as building it. Second, all four have converged on MCP as how an agent calls out to arbitrary tools, Zapier and SimplAI included — a sign the connector problem is being solved once, industry-wide, rather than by each platform maintaining its own catalog.

## Gaps to explore in the project

My proof of concept works through the ideas from three earlier posts — a [canvas](/writing/agentic-workflow-builder-canvas-prototype/), a [test bench](/writing/agentic-workflow-builder-test-bench/), and an [intake questionnaire](/writing/agentic-workflow-builder-questionnaire/) — and isn't trying to compete with any of the four products above. But the comparison sharpens what's worth building out.

**Testing.** All four products underserve it relative to how much it matters once a workflow includes an LLM step. My test bench prototype treats it as a first-class citizen already — assertions and pass@k/pass^k sampling alongside the canvas, not bolted on after. Formalizing that (code-based graders for deterministic steps, model-based graders for probabilistic ones, testable before a workflow runs live) is a more specific angle than "another visual builder."

**Intake.** None of the four start from "should you build this, and what shape should it take" — they all assume you've already scoped the problem and start from an empty canvas or script. A short intake that turns a plain-language description into a priority score and a suggested starting shape is a different entry point, and it's cheap to build well since it only needs to produce a reasonable first draft.

The canvas itself — step types, drag-and-drop — is already well-solved by n8n and Zapier. One thing isn't a gap at all: connecting a step to an outside tool. MCP has already won that argument across the market, so the answer there is to follow it rather than build a competing catalog. Testing and intake are where the market is thin.

## Further reading

- [Zapier — What is a Zap?](https://help.zapier.com/hc/en-us/articles/8496309697421-What-is-a-Zap), [Request Approval / human-in-the-loop](https://zapier.com/blog/human-in-the-loop-guide/), and [Zapier MCP](https://docs.zapier.com/mcp/home)
- [n8n](https://n8n.io/) and its [evaluations for AI workflows](https://docs.n8n.io/advanced-ai/evaluations/overview/)
- [LangGraph](https://www.langchain.com/langgraph) and its [multi-agent supervisor pattern](https://github.com/langchain-ai/langgraph-supervisor-py)
- [SimplAI](https://simplai.ai/)
- Anthropic, [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — the grader/pass@k framing this project and comparison both use
