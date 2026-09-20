---
title: "A techlead agent per project, and it never pauses a demand"
translationKey: "decision-0011"
adr: "0011"
adr_title: "The project orchestrator: the techlead agent"
adr_file: "0011-project-orchestrator.md"
date: 2026-08-30
weight: 11
group: "agents"
description: "When a project has two or more active demands, an orchestrator agent watches for overlap, dependencies and interference, plans a solution and brings the developer a decision — without stopping any demand while it waits."
related: ["0004", "0005", "0007"]
---

## What was on the table

Parallel demands in one project produce situations no single demand's
agent observes: two branches touching the same files, one demand
depending on another's result, a behaviour change in one that breaks the
other's premise. Something has to see across demands, and detecting must
not become blocking.

## The paths we weighed

**Static rules only** — a path diff plus a declared dependency graph. Kept
as sensors; insufficient for interference in behaviour, which needs a
semantic reading of specs and diffs.

**Pausing demands at risk until a decision.** Rejected: it serialises the
parallelism the platform exists for.

**A human techlead.** Rejected: that is the attention the platform is
meant to spare.

## What we chose, and why

**Every project has an orchestrator agent** — the techlead of the demand
agents. It wakes when the project has two or more active demands, runs on
the platform rather than in a sandbox, and observes the event log, the
branches' state and the demands' flows.

**It plans before it asks.** On detecting a cross-cutting situation it
produces options with a recommendation and opens an attention item, never
a raw alarm. The developer's choice becomes a **coordination directive**,
an event visible in the threads involved: sequencing with a cherry-pick or
rebase between branches, a preferred order in the merge queue, file
partitioning, cross verification. The vocabulary is extensible.

**A detected situation never pauses a demand.** The demand proceeds as far
as it can; when the directive's condition is met it applies the
coordination and continues. A block exists only when the demand itself has
exhausted what can be done, and is then its own attention item.

## What it cost

Observation is cheap and event-driven; planning is routed as an
investigation. Directives are new state between demands and must appear in
the timeline and in both threads.

## Since then

The techlead belongs to the part of the platform that executes rather than
models, which the roadmap places after the agent's tools; flows and
reactions as data are the vocabulary a directive will be written in.
