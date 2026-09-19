---
title: "The flow is data, and the screen never knew fixed stages"
translationKey: "decision-0010"
adr: "0010"
adr_title: "A dynamic, typed and inheritable workflow"
adr_file: "0010-dynamic-workflow.md"
date: 2026-08-30
weight: 10
group: "delivery"
description: "Jira customises only statuses. GitHub Actions is flow-as-code for a machine. BPMN models everything and costs an analyst. The middle — typed stages, free composition — was vacant."
related: ["0004", "0009", "0021"]
---

## What was on the table

The demand's cycle was undefined, and the product decided how it should be:
**dynamic**. The platform has a default, but each account, workspace,
project and even a single demand may work its own way. The chat screen and
the agent have to understand any flow without knowing any of them — which
means the flow is data, not code.

The references occupy the extremes. Jira customises only statuses — no
artifacts, no agent. GitHub Actions is flow-as-code for a machine, not for a
human to follow. BPMN models everything and costs an analyst. The middle
ground — typed stages plus free composition — was vacant.

## The paths we weighed

**Fixed platform stages.** The earlier design, and the old PRD's. Rejected:
accounts work differently, and the cost of dynamism fell to almost zero once
stages were typed — the static case becomes the particular case of one flow.

**A full workflow engine** — BPMN, Temporal. Rejected for v1: it buys
conditionals and parallelism nobody asked for, at a price in complexity
everybody would pay.

**Flow as code, a YAML per repository.** Rejected as the primary interface:
the audience is the developer-as-manager on a screen, not a pipeline.

## What we chose, and why

**Stages have a semantic type; flows are compositions.** The type vocabulary
belongs to the platform — context, spec, plan, implementation, test, human
validation, finalisation, generic — and the type decides the renderer on the
screen and the agent's behaviour: which artefact to produce, where to stop.
A new type is an evolution of the platform; a new composition is not.

The v1 structure is deliberately simple — a name, a version, a list of
stages with a key, a type, artefacts and a gate — with no conditionals, no
parallel stages, no rules language.

**A resolution chain with inheritance**: platform, account, workspace,
project, demand — the nearest level wins, inherited by omission, overridden
by declaration, and the interface always shows where the effective flow came
from. **The demand freezes the flow's version when it starts**; a stage's
progress is an event, and the ruler on the screen is a projection.

And a refinement of the sharing decision: a resource with a credential stays
closed by default; a content resource — a flow, a skill — is open within an
organization by default. A credential is risk; a flow is knowledge.

## What it cost

The inheritance chain requires a visible trail — "inherited from…" — or it
becomes a support ticket. External sharing of flows between accounts stayed
out of v1, recorded as strategic: it waits, it does not sleep. And the syntax
of the executable criteria inside the spec artefact stayed open.

## Since then

Two additions, both inside the structure and both tested against the line
"no rules DSL". On 2026-09-03 an artefact got an address: a file in the
project's root repository. On 2026-09-06 a stage gained *actions* — what the
platform does when a demand enters it and when it leaves — drawn from a
closed vocabulary the platform implements, with flat parameters and no
condition to evaluate. That was the smallest change that answers "provision
the bench when implementation ends" without reopening what was refused: the
moment a stage could say *when* to act rather than only *what*, the flow
would stop being data a loader reads and become a program. One limit fell out
of the idempotency gate rather than the design, and is refused loudly when a
flow is written: an action name may not repeat at the same moment on one
stage, because the second would be skipped forever, in silence.
