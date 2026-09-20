---
title: "No green, no PR — and a merge queue after the green"
translationKey: "decision-0005"
adr: "0005"
adr_title: "No green, no PR: native verification, and a merge queue per repository"
adr_file: "0005-no-green-no-pr.md"
date: 2026-08-29
weight: 5
group: "delivery"
description: "A pull request opens only when executable acceptance passes and an independent critic has reviewed it, carrying its evidence. After the green, a queue per repository re-verifies each PR against the current main and merges one at a time."
related: ["0004", "0008", "0011", "0023"]
---

## What was on the table

Agents deliver pull requests faster than humans can review them, and
parallel demands on one repository produce PRs each verified against an
outdated `main` — the first merge can silently break the others. A merge
is a human decision, by product rule; what happens before the human is
called, and after the PR is green, are the platform's to decide.

## The paths we weighed

**Human-only review.** Rejected: review capacity is the bottleneck.

**Auto-merge on green.** Rejected: the human gate is a product rule.

**Optimistic merge, in arrival order.** Rejected: semantic breaks reach
`main`.

**One file, one owner.** Rejected: it serialises the required
parallelism.

**Only the provider's merge queue.** Rejected as the sole route: not
universal, and no cross-demand view.

## What we chose, and why

**Before the pull request, four rules.** Acceptance criteria are
executable and live in the demand's spec — test suites and checks derived
from it. No PR opens with acceptance failing; the agent iterates to green,
and a persistent failure becomes a question to the human. An independent
critic — a strong model, clean context, maximum effort — reviews the diff,
the spec and the evidence before the human does. The PR carries the
evidence package, so the human reviews the exception, not the rule.

**After the green, a merge queue per repository**, as a domain concept. A
green PR enters the queue; the queue reapplies each PR onto the current
`main`, re-runs verification and merges one at a time. A conflict is first
the demand agent's task, then the human's. Overlap between demands is
detected before the PR by the project orchestrator. The provider's native
queue is used where it exists; the platform's orchestrates on top.

## What it cost

The critic costs tokens, with no saving allowed. Merges are serialised per
repository, so queue position and forecast are shown in the cockpit.
Re-verification per queue position costs compute. The syntax of the
executable criteria is defined in the workflow spec.

## Since then

Two records — the rules before the PR and the queue after it — were
consolidated on 2026-09-04. The spec artifact got an address in the
project's knowledge repository, and verification got its runner in
[the verification decision](../verification-runs-from-source/).
