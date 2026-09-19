---
title: "The techlead is an agent, and it never pauses a demand"
translationKey: "decision-0011"
adr: "0011"
adr_title: "The project orchestrator: the techlead agent"
adr_file: "0011-project-orchestrator.md"
date: 2026-08-30
weight: 11
group: "agents"
description: "Two demands touching the same files, one depending on the other's result — situations no single demand's agent can see. Somebody had to see them, and the cheapest wrong answer was to stop everything until a human decided."
related: ["0004", "0005", "0007"]
---

## What was on the table

With parallel demands in one project, cross-cutting situations appear that
no demand's agent sees on its own: two demands touching the same files, one
depending on another's result, a behaviour change in one that breaks the
other's premise. The delivery decision had foreseen that "the orchestrator
sees the overlap" without saying who the orchestrator was.

## The paths we weighed

**Static detection rules** — a path diff plus a declared dependency graph.
They stay as sensors, but they are not enough alone: interference in
behaviour does not show up in a path; it needs a semantic reading of the
specs and the diffs.

**Pause demands at risk until a decision.** Rejected with emphasis. It kills
the parallelism that is a requirement and turns detection, which is cheap,
into a block, which is expensive.

**A human techlead.** That is everybody's way today — and it is exactly the
scarce attention the platform exists to spare.

## What we chose, and why

Every project has an **orchestrator agent, the techlead of the demand
agents**. It wakes when the project has two or more active demands and
sleeps otherwise; it lives on the platform, not in any demand's sandbox, and
observes through the event log, the branches' state and the demands' flows.

**Autonomy first.** On identifying a cross-cutting situation, it plans
solutions and brings the attention box a decision prompt — ready options,
with a recommendation — never a raw alarm. The developer's choice becomes a
**coordination directive**, an event that appears in the threads involved.
The canonical example: "demand 1 depends on demand 0" — so when 0 commits
what 1 needs, 1 cherry-picks from 0's branch and carries on.

**The golden rule: an identified cross-cutting situation never pauses a
demand.** Demand 1 goes as far as it can; when the directive's condition is
met, it applies the coordination and continues. A block only exists when the
demand itself exhausts what can be done without the condition — and then it
is its own block, visible in the box.

The initial vocabulary: sequencing with a cherry-pick or rebase, a preferred
order in the merge queue, file partitioning ("2 does not touch module X until
1 merges"), cross verification. Extensible — the techlead proposes, the
vocabulary only names.

## What it cost

The techlead's model cost: observation is cheap, planning is expensive and
is routed as an investigation; it wakes on an event, not by polling. And a
coordination directive is new state between demands — it has to appear in
the timeline and in both threads, or it becomes invisible magic.

## Since then

Nothing has contradicted it, and nothing has built it yet: the techlead
belongs to the part of the platform that executes rather than models, which
the roadmap puts after the agent's tools. What has been built since — flows
as data, reactions as data — is the vocabulary a directive will be written
in.
