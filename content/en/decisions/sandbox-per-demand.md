---
title: "One microVM per demand, one worktree shared by its threads"
translationKey: "decision-0017"
adr: "0017"
adr_title: "A microVM per demand, with a single shared worktree"
adr_file: "0017-sandbox-per-demand.md"
date: 2026-08-31
weight: 17
group: "delivery"
description: "The hard boundary is between accounts and between demands, so each demand gets one sandbox and its agents share one workspace. Verification never runs there: it runs in a separate environment built from a commit."
related: ["0005", "0007", "0021", "0023"]
---

## What was on the table

A demand has one main agent and several subagents. Where each of them
runs, and where the application under test runs, are separate questions
with different isolation needs: agents of one demand trust each other;
demands and accounts do not; and a test must speak about a commit, not
about whatever the agent's working tree contains at the moment.

## The paths we weighed

**One microVM per thread.** Rejected: N times the overhead to isolate
agents that cooperate.

**One git worktree per thread.** Deferred: every command would have to
know which tree it runs in; kept as the mapped evolution if two threads
edit the same files often.

**Tests inside the agent's sandbox.** Rejected: a working tree is not a
commit, and the agent's environment is not a clean one.

## What we chose, and why

**One microVM per demand.** The boundary that has to be hard is between
accounts and between demands, and the database enforces one live sandbox
per demand. **One worktree, `/workspace`, shared by the demand's threads:**
most threads read — a log, a database, source — and whoever edits is
typically the main agent.

**The knowledge boundary is the project**, not the demand: every sandbox
of a project clones the project's knowledge repository at `/project`. The
workspace boundary stays per demand.

**Verification does not run in the sandbox.** It runs in an ephemeral
runner built from a commit, holds the demand's address while it runs —
`<service>--<demand>.<domain>` — and parallel runs of one demand queue.
Isolation tiers are honest: asking for hardware isolation where the
executor cannot provide it is refused, never silently downgraded.

## What it cost

Two compute lifecycles — the demand's long sandbox and verification's
short runner. A test requires a published commit, which makes the agent's
flow edit, commit, verify. The hardware tier is exercised only on clusters
that have a microVM runtime.

## Since then

The address became per demand with queued runs on 2026-09-03, and the
verification mechanism became [the runner](../verification-runs-from-source/)
the next day.
