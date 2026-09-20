---
title: "The flow is data: typed stages, free composition, inheritance"
translationKey: "decision-0010"
adr: "0010"
adr_title: "A dynamic, typed and inheritable workflow"
adr_file: "0010-dynamic-workflow.md"
date: 2026-08-30
weight: 10
group: "delivery"
description: "Stages have a type the platform knows how to render and act on; flows compose them freely and are resolved down a chain — platform, account, workspace, project, demand. No conditionals, no rules language: a flow is data a loader reads."
related: ["0004", "0009", "0021"]
---

## What was on the table

The human↔agent development flow is configurable per account, workspace,
project and even demand. The cockpit and the agent must operate any flow
without knowing a specific one. The references sit at the extremes: task
trackers customise only statuses; CI systems are flow-as-code for a
machine; BPMN models everything at the cost of an analyst.

## The paths we weighed

**Fixed platform stages.** Rejected: accounts work differently, and typed
stages make the fixed case a particular flow.

**A workflow engine** — BPMN, Temporal. Rejected for v1: conditionals and
parallelism nobody asked for.

**Flow as code, a YAML per repository.** Rejected as the primary
interface; the audience is a developer on a screen.

## What we chose, and why

**Stages have a semantic type; flows are compositions.** The type
vocabulary belongs to the platform — context, spec, plan, implementation,
test, human validation, finalisation, generic — and the type decides the
renderer in the cockpit and the agent's behaviour. A new type is a
platform change; a new composition is not.

**The structure is deliberately small:** a name, a version, stages with a
key, a type, artifacts, a gate, and optionally actions. An artifact is a
file in the project's knowledge repository. **Stage actions** — what the
platform does when a demand enters or leaves a stage — come from a closed
vocabulary (`open_attention`, `close_attention`, `send_email`,
`provision_bench`) with flat parameters: a declaration, not a program.

**Resolution with inheritance:** platform, account, workspace, project,
demand — the nearest declared level wins, and the interface shows where
the effective flow came from. **A demand freezes the flow's version when
it starts**; stage progress is an event. A flow may be promoted to a higher
level by a holder of `manage`. Content resources like flows are open
within an organization by default; resources with credentials are closed.

## What it cost

The inheritance chain requires the "inherited from" trail. External
sharing of flows between accounts stayed out of v1, recorded as strategic.
The syntax of the executable criteria inside the spec artifact is defined
in the workflow spec.

## Since then

Artifacts got their address in the knowledge repository on 2026-09-03, and
stage actions were added on 2026-09-06 — both inside the structure, both
keeping the flow data rather than code.
