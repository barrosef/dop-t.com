---
title: "Measure, cap, route — and design the prompt for the cache"
translationKey: "decision-0008"
adr: "0008"
adr_title: "The LLM's cost: measuring it, capping it, routing it, and spending less"
adr_file: "0008-llm-cost-governance.md"
date: 2026-08-29
weight: 8
group: "agents"
description: "Every model call emits a cost event; every demand has a budget that pauses rather than kills; a router picks the class of model per kind of work — never cheaper for the critic. And the prompt is laid out so the expensive part is cached."
related: ["0004", "0005", "0007", "0016"]
---

## What was on the table

Model usage is the product's largest variable cost. Without measurement
per account there is no business model; without a budget per demand, a
demand can burn money in a loop; without routing, everything runs on the
most expensive model. Underneath: an agent loop resends the conversation
on every turn, and the discounts the APIs offer — prefix caching,
batching — require the prompt to be designed for them.

## The paths we weighed

**One strong model for everything.** Rejected: a ceiling on the margin.

**Routing by prompt size.** Rejected: the task's nature decides.

**A router alone.** Rejected: it does not address the transcript being
resent.

**Compaction as the way to resume a demand.** Rejected: rebuilding the
context from events is cheaper.

## What we chose, and why

**Measure.** Every model use emits a cost event — tokens, cache reads and
writes, model, demand, thread, account — into the demand's log; a
recurring cache miss on a stable prefix is an alert.

**Cap.** A per-demand budget with a soft cut: on overrun the demand pauses
and opens an attention item; the agent is told its ceiling and paces
itself. A subagent's card carries its slice.

**Route.** A router chooses a class and an effort per kind of work —
cheap for mechanical work, medium for investigation, strong for planning
and implementation — and the active provider's catalogue resolves the
concrete model. **The critic is never routed cheaper**: a strong model at
maximum effort, because saving on the brake returns as a rejected pull
request.

**Design for the cache.** The prompt is laid out as system, tools, context
package, then the conversation; the package is serialised
deterministically so the prefix stays byte-identical. Raw tool output
stays in the specialist's thread; filters run as code where possible;
findings use structured outputs. A resumed demand rebuilds its context
from the package, the findings and a trace summary rather than resending
the transcript. Projections are computed in code, never by a model;
non-interactive work goes to the batch API; tools are loaded on demand.

## What it cost

The routing table is a starting point until telemetry calibrates it. The
package's deterministic serialisation is a permanent constraint on the
context subsystem. Reconstruction on resume must be shown to be sufficient.

## Since then

Two records were consolidated into this one on 2026-09-04. The split
between policy (which class) and catalogue (which model) is what lets the
rules hold for every vendor behind
[the agent provider port](../agent-provider-as-port/).
