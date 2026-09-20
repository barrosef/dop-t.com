---
title: "Storage is the easy half"
translationKey: "decision-0006"
adr: "0006"
adr_title: "Context is a subsystem: a knowledge base and a package per demand"
adr_file: "0006-context-as-subsystem.md"
date: 2026-08-29
weight: 6
group: "knowledge"
description: "The requirements said 'context created by Claude' and 'the workspace's rules' with no entity, no port, no mechanism. What separates a useful agent from a useless one is what goes in and how it is assembled — not where it is kept."
related: ["0001", "0004", "0007", "0021"]
---

## What was on the table

What separates a useful agent from a useless one is context: the project's
rules, the code's map, the memory of what has already been tried. In the
requirements this showed up as "context created by Claude" and "the
workspace's rules" — with no entity, no port, no mechanism. The product's
directive was explicit about storage: secure, available, permissioned,
reachable from the microVMs, "so that the agents work in a genuinely
intelligent way".

Storage is the easy half. The half that generates intelligence is *what* is
in there and *how* it is assembled per demand.

## The paths we weighed

**A raw document folder in the sandbox.** Rejected: with no curation and no
assembly, the agent digs, and digging is what a package exists to eliminate.

**Everything embedded in the prompt.** Rejected: it blows the context window
and grows with the project, not with the demand.

**An external RAG service per customer.** Deferred: the port allows plugging
one in later; starting there is buying infrastructure before having content.

## What we chose, and why

**A knowledge base per project**, versioned, in three layers. *Rules*: the
conventions the agent obeys. *The index*: the code's map — what lives where,
how to build, how to test; without it, every demand spends its first half
hour rediscovering the repository. *Memory*: findings and lessons from past
demands, decisions, forensic readings.

**A context package per demand**, assembled when the sandbox is provisioned:
the spec, the rules, the index of the repositories involved, the relevant
memories. The agent's carry-on luggage — curated, not dumped. **A write-back
at closing**: the demand's findings and lessons go into the memory layer.
Context is a cycle, not a file. And **the index updates on a merge event**,
not on a schedule: the map follows the real `main`.

## What it cost

Curation is real work: assembling the package and judging a memory's
relevance falls to the orchestrator. And per-account storage with
fine-grained permission is one more surface for the contract tests to cover.

## Since then

The rejected "raw folder" came back, and it is not a contradiction. On 2026-09-03, [ADR-0021](../project-knowledge-as-a-git-repository/)
gave the three layers a home: the project's root repository, a git server
the platform runs, cloned into every sandbox. What had been rejected was a
folder *as a replacement* for the package. The package stayed exactly as
defined here — the curated, budgeted luggage that goes into the prompt — and
the repository was *added* as the shelf: complete, with a generated manifest
so the agent does not dig. The package is paid for on every turn; the shelf
costs nothing until a file is opened. And "over the object store" moved: text
lives in git, which gives attribution and history; the bucket keeps bytes —
diagrams, exports, what a repository is bad at.
