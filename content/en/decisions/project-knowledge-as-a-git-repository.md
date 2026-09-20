---
title: "The project's knowledge is a git repository the platform hosts"
translationKey: "decision-0021"
adr: "0021"
adr_title: "The project's knowledge is a git repository, hosted by the platform"
adr_file: "0021-project-knowledge-as-a-git-repository.md"
date: 2026-09-03
weight: 21
group: "knowledge"
description: "Every project has a root repository, born with it, cloned into every sandbox. Agents write by committing; every push is an event; a per-demand token opens exactly that repository; the user's own remote is a mirror."
related: ["0001", "0003", "0004", "0006", "0017"]
---

## What was on the table

Every document that is the agent's knowledge — rules, maps, memories,
demand specs and plans — must be present in every sandbox of a project
from provisioning, writable by agents, shared between demands, and
attributed per change. Sharing files is not enough: the platform must
always know who changed what.

## The paths we weighed

**A ConfigMap.** Rejected: configuration, not data; one megabyte in etcd.

**A per-demand volume filled by a loader.** Rejected: per demand,
read-only, no sharing.

**A read-write-many volume per project.** Rejected: unavailable on the
local cluster, and a filesystem has no attribution.

**Object storage mounted as a filesystem.** Rejected for text —
last-writer-wins, history without authorship; kept for binaries.

**The user's own provider as the primary.** Rejected: it requires an
integration before a project can hold knowledge, and puts the user's
credential in the sandbox's path. It survives as the mirror.

## What we chose, and why

**Every project has a root repository**, created with it in a git server
the platform runs, with a fixed layout — `rules/`, `index/`, `memory/`,
`demand/<id>/` — and a manifest the platform regenerates on every push.

**Every sandbox clones it** at `/project`, read-write. Agents read files
and write with `git commit` and `git push`; sharing between demands is
push and pull; a conflict the agent cannot resolve becomes an attention
item. Commits are attributed by [the credential decision](../organization-credential-human-authorship/).

**The sandbox's credential is a token that opens exactly one
repository** — per demand, short-lived, scoped to the project, delivered
as a projected file and never as an environment variable. It is the only
credential a sandbox holds.

**The user's remote is a push mirror:** the platform's repository stays
primary and pushes onward with the user's credential from the vault, which
the sandbox never sees. No two-way sync. A person may also clone the
platform's repository directly with their platform identity.

**Every push is an event** — author, paths, commit — regenerating the
manifest, feeding the timeline, and closing the lessons loop when an agent
commits into `memory/`. The port, `ProjectRepository`, has two adapters —
the platform's git server on Kubernetes and bare repositories behind
`git http-backend` locally — and one contract suite. **Text in git, bytes
in the object store.**

## What it cost

A stateful git server to run, with storage and backup. `git` in the
devbox image and a clone at provisioning. The sandbox contract's
guarantees for the knowledge path: readable, writable, visible to the
next sandbox after a push, persistent across resume and demands, fenced
per project.

## Since then

Built and proven on both executors — Docker and Kubernetes — on the day it
was decided. The cockpit's view of the shelf is the remaining surface.
