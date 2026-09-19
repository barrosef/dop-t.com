---
title: "The largest cost line nobody had written down"
translationKey: "decision-0008"
adr: "0008"
adr_title: "The LLM's cost: measuring it, capping it, routing it, and spending less"
adr_file: "0008-llm-cost-governance.md"
date: 2026-08-29
weight: 8
group: "agents"
description: "N agents, long sessions, many tenants — and not a line about what it costs. Measure it, cap it, route it; and notice that half the saving came from decisions already taken for other reasons."
related: ["0004", "0005", "0007", "0016"]
---

## What was on the table

N autonomous agents, times long sessions, times many tenants: the product's
largest variable cost — and no document had a line about it. With no
per-account measurement there is no business model. With no per-demand
budget, a pathological demand burns money in a loop. With no routing,
everything runs on the most expensive model.

Underneath all three sits the mechanic of an agent loop: **the whole
conversation is resent on every turn.** A forty-turn agent pays for its
transcript forty times. The API offers discounts of up to ninety percent
through caching and fifty through batching, but none of them is a flag; they
all require engineering discipline.

## The paths we weighed

**One strong model for everything.** Simple and expensive; it becomes a
ceiling on the margin.

**Route by prompt size.** Rejected: what matters is the nature of the task,
not its length.

**A model router alone.** Not enough: the biggest waste is resending the
transcript, and the router does not touch it.

**Compaction as the way to resume a demand.** Rejected: rebuilding the
context from events is cheaper, cleaner, and we already had the material.

## What we chose, and why

**Measure from day one.** Every model use emits a cost event — tokens,
model, demand, thread, account — into the demand's log; measurement is a
projection, not a parallel system. The event carries cache reads and cache
writes, so a recurring miss on a stable prefix is an alert, not a mystery.

**A per-demand budget with a soft cut.** On overrun the demand pauses and
asks, through the attention box; it never dies mid-way and never keeps
burning. The agent sees the ceiling and paces itself.

**A router that chooses model and effort per kind of work** — mechanical
work on the cheap class, investigation on the medium, planning and
implementation on the strong — with the table marked *draft*, to be
calibrated with telemetry. One rule limits all the others: **there is no
saving on the critic.** A strong model, maximum effort. Saving on the brake
returns the cost as a rejected pull request, the most expensive rework in
the flow.

**Cache-first.** The prompt is laid out for a stable prefix — system, tools,
context package, then the conversation — and the package is serialised
deterministically: a stable order, no timestamps, no volatile ids, because a
changed byte invalidates everything after it. An operator's intervention
enters as a message in the middle, never by editing the top.

**Dirty context does not enter the main agent.** The specialist's thread
keeps the logs and dumps; the main agent receives the finding. Where it
fits, the filter runs as code in the sandbox and only the result passes
through the model.

**Resume by reconstruction, not by replay.** A demand that resumes days
later does not resend its transcript — the cache has expired anyway. The
context is rebuilt from the package, the findings and a summary of the
trace.

**Do not use a model where code does the job.** The dossier, the metrics,
the auditing and the attention box are projections of the log, computed in
code, at zero token cost. Asynchronous work — the index after a merge,
nightly metrics — goes to the batch API.

## What it cost

The routing table is a guess until there is data — hence its draft status.
The package's deterministic serialisation is a permanent constraint on the
context subsystem. And reconstruction on resume has to be demonstrably
sufficient: if the agent "forgets" what mattered, the trace's summary is
what is weak.

## Since then

The record was written as two — one to measure and cap, one to spend less —
and folded into one on 2026-09-04, because the second had always declared
itself a complement of the first. When the agent runtime became a port per
vendor ([ADR-0016](../agent-provider-as-port/)), the split between *policy*
(which class) and *catalogue* (which concrete model) was what let the cost
rules keep holding for every vendor without knowing any of them — and the
same record warned that prefix-cache semantics differ between vendors, which
is the most expensive divergence this decision depends on.
