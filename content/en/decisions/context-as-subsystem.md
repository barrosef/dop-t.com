---
title: "Three layers of knowledge, one curated package per demand"
translationKey: "decision-0006"
adr: "0006"
adr_title: "Context is a subsystem: a knowledge base and a package per demand"
adr_file: "0006-context-as-subsystem.md"
date: 2026-08-29
weight: 6
group: "knowledge"
description: "A project's knowledge is rules, an index of its code and a memory of past demands. Each demand receives a curated package assembled from them, and the whole shelf is cloned into the sandbox for the agent to open what it needs."
related: ["0001", "0004", "0007", "0021"]
---

## What was on the table

An agent's usefulness depends on the project's rules, the map of its code
and the memory of what previous demands found. That knowledge has to be
permissioned per account and project, reachable from the sandbox, and —
the part that decides quality — assembled per demand rather than dumped.

## The paths we weighed

**A raw document folder as the only mechanism.** Rejected: with no
curation the agent digs.

**Everything in the prompt.** Rejected: it grows with the project, not the
demand.

**An external RAG service per customer.** Deferred; the port allows it
later.

## What we chose, and why

**A knowledge base per project, in three layers:** *rules* — the
conventions the agent obeys; *index* — one map per repository: what lives
where, how to build, how to test; *memory* — findings and lessons from
past demands. Text lives in the project's root repository, under `rules/`,
`index/`, `memory/` and `demand/<id>/`; binary artifacts live in the
object store, referenced from the repository.

**A context package per demand** is assembled when the sandbox is
provisioned — the spec, the rules, the index of the repositories
involved, the relevant memories — serialised deterministically so the
prompt's prefix stays cacheable. **The package is curated; the repository
is the shelf.** The whole root repository is cloned into the sandbox with
a generated manifest, so the agent opens what it needs without paying for
it on every turn.

**Write-back at closing:** a demand's findings and lessons are committed
into `memory/`. **The index is regenerated on a merge event**, not on a
schedule.

## What it cost

Assembling the package and judging a memory's relevance is the
orchestrator's work. Per-account, per-project permissions on the storage
are covered by the repository and object-store contract suites.

## Since then

Text moved from the object store to the project's root repository on
2026-09-03, when [the knowledge repository](../project-knowledge-as-a-git-repository/)
was decided; the three layers and the package were unchanged.
