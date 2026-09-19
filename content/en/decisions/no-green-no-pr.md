---
title: "The bottleneck moved to the reviewer's desk"
translationKey: "decision-0005"
adr: "0005"
adr_title: "No green, no PR: native verification, and a merge queue per repository"
adr_file: "0005-no-green-no-pr.md"
date: 2026-08-29
weight: 5
group: "delivery"
description: "Agents already close the loop up to the pull request. Without verification before the human, parallelism only moves the queue. And after the green, three PRs tested against yesterday's main can still break production together."
related: ["0004", "0008", "0011", "0023"]
---

## What was on the table

Two failures on the same path, and they are consecutive.

**Before the human.** The research was conclusive: agents already close the
loop up to the pull request, and the flow's bottleneck had become human
review capacity. With no native verification, an executor's parallelism
only moves the queue — from development to the reviewer's desk. The product
had already fixed that a merge is a human decision; this decides what
happens before the human is called.

**After the green.** Parallelism is a requirement: three demands at once,
"regardless of the repos overlapping". Three green PRs, each tested against
the `main` of when its branch was born. The first merge invalidates the
other two — at best a text conflict, at worst a silent semantic break: one
PR removes the check the other assumed. A PR's CI does not see it.
Production does. Agent fleets turn that monthly accident into a daily one.

## The paths we weighed

**Human-only review.** The market's default, and where the fleet drowns the
reviewer.

**Auto-merge on green.** Rejected: the human gate is a non-goal fixed by the
product, and the critic does not replace responsibility.

**Optimistic merge**, in arrival order. Rejected: it is exactly the
semantic-break scenario.

**One file, one owner.** Rejected: it kills the parallelism that is a
requirement — a queue in disguise.

**Only the provider's merge queue.** Rejected as the sole route: not every
provider has one, and the cross-demand view is something the provider does
not have.

## What we chose, and why

**Before the PR, four rules in order.** Acceptance is born in the spec,
executable — a criterion that does not execute is a wish. The agent iterates
to green; no PR opens with acceptance failing, and a persistent failure
becomes a question to the human, never a broken PR. A critic reviews before
the human — an independent instance, clean context, without the history of
whoever implemented, receiving diff, spec and evidence and issuing a verdict:
the first line of defence against rubber-stamping. And the PR carries the
evidence package, so the human reviews the exception, not the rule.

**After the green, a merge queue per repository, as a domain concept.** A
green PR enters the queue; the queue reapplies each one on top of the
updated `main`, re-runs the verification and merges one at a time. Only what
is green against the real state gets in. A conflict is the agent's task
first, the human's on escalation. Overlap is detected early, before the PR,
by an orchestrator that later got a name. And the provider's native queue is
used where it exists, with the platform's orchestrating on top.

## What it cost

The critic costs tokens — a strong model, no saving here. A serialised
merge per repository means delivery latency grows with the queue, so the
position and the forecast are visible in the cockpit. Re-verification at
each position costs compute. And the exact syntax of an executable criterion
was left to the work model, not fixed here.

## Since then

The record was written as two — the rules before the PR, the queue after it
— and folded into one on 2026-09-04, because they are the same path from
green to `main`. The spec artefact the criteria live in got an address when
the project's knowledge became a repository
([ADR-0021](../project-knowledge-as-a-git-repository/)). And the question of
*where* verification runs — which this decision did not ask — went through
two answers before landing on a runner that builds from source
([ADR-0023](../verification-runs-from-source/)).
