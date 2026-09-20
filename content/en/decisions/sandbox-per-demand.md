---
title: "Five microVMs, not twenty — and a sentence we had to take back"
translationKey: "decision-0017"
adr: "0017"
adr_title: "A microVM per demand, with a single shared worktree"
adr_file: "0017-sandbox-per-demand.md"
date: 2026-08-31
weight: 17
group: "delivery"
description: "Where does each agent run, and where does the application under test run? The hard boundary is between accounts and demands, not between collaborating threads. The third clause of this decision did not survive."
related: ["0005", "0007", "0021", "0023"]
---

## What was on the table

A demand has one main agent and N subagents — one reading a log, another
the database, another the code. We had to decide where each of them runs,
and where the application under test runs. Three problems arrived together.

**Isolation between threads.** One microVM per thread gives strong
isolation — at the cost of twenty microVMs in a project with five demands
of four threads each, every one with its own kernel and hundreds of
megabytes of overhead.

**Ports.** Bringing the same application up more than once in one
environment means arbitrating a port and still exposing a route for a human
to look at it.

**Which code the test speaks about.** This is the one that decided it.

## What we chose, and why

**One microVM per demand.** The boundary that has to be hard is between
accounts and between demands, and that is the one the per-demand sandbox
guarantees. A demand's threads are agents of the same account working on the
same problem: mutually trusted. Spending a microVM between them would be
using a security tool to solve a coordination problem. The database already
imposed it: one live sandbox per demand.

**One worktree, shared.** A worktree per thread would solve file collision
and add real complexity — every command would have to know which tree it
runs in. The risk was accepted explicitly: two threads that *edit* the same
file trample each other. Tolerable, because most threads read — a log, a
database, source — and whoever edits is typically the main agent. If
practice shows otherwise, the way out is mapped.

**An ephemeral pod per verification run.** The argument came from our own
code: a verification run's commit is mandatory, and the refusal says
*"evidence that does not say which code it ran on is not evidence."* A test
inside the agent's sandbox runs against the dirty working tree, which is no
commit at all. A pod built from a commit tests exactly what will be merged.

## What it cost

Two kinds of compute — the agent's long sandbox and verification's short
pod — with different life cycles on purpose. A test now requires a
published commit before it runs, which changes the agent's flow to edit,
commit, verify — closer to what a human does. And two open questions were
left written down: when to provision, and where the microVM tier could be
validated at all, since the local cluster has no Kata runtime and honestly
refuses to pretend.

## Since then

The third clause did not survive; the record now states the runner and
lists the two revisions by date.
On 2026-09-03 the owner decided that the address stays per demand and
parallel verification runs *queue* — simplicity over latency inside one
demand. The write-up of that decision then added a sentence the owner had
not said: that verification "just runs in the demand's sandbox". That put
the run back on the dirty tree this very decision had refused. It was an
error in the write-up, corrected the next day by
[ADR-0023](../verification-runs-from-source/), which keeps the argument —
evidence must name a clean environment built from a commit — and changes the
mechanism: not a pod from a built image, but a runner that pulls the commit
and builds from source. What still holds here is the sandbox itself: one per
demand, one worktree shared by its threads.
