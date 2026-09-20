---
title: "One resource type, one way to share"
translationKey: "decision-0009"
adr: "0009"
adr_title: "The resource as the account's unit of ownership and sharing"
adr_file: "0009-resource-as-unit-of-sharing.md"
date: 2026-08-30
weight: 9
group: "resources"
description: "Integrations, skills, workflows and git flows are all resources of an account, shared by the same grants and the same invite. A new kind of resource never touches the sharing mechanism."
related: ["0002", "0010", "0016"]
---

## What was on the table

An account owns more than integrations: the agents' reusable skills, the
workflows a developer and the agents follow on a demand, and git flows —
branch governance as an artifact: a taxonomy per card type, base and
direction, release composition, hotfix back-merge. And integrations
themselves have three categories: git hosts, task managers and agent
providers. All of it is owned, versioned and shared under authorization.

## The paths we weighed

**One sharing mechanism per kind.** Rejected: the same policy written four
times, with four screens and four sets of bugs.

**Everything as an integration.** Rejected: a skill and a flow have content
and versions, not credentials; the wrong entity charges at every evolution.

## What we chose, and why

**`Resource` is the unit of ownership and sharing** — id, account, kind,
name, config, and an optional credential reference. Four kinds:
`integration` (git, task manager, agent — the only kind with a credential),
`skill`, `workflow`, `git_flow`.

**Grants are per resource:** `use` and `manage`, per user, composed in the
invite and editable at any time. A personal account's resource is private;
only an organization's is shareable; owners and admins hold an implicit
`manage`; revoking `use` does not tear down what is already configured.

**The platform publishes global resources** — providers, skills, default
flows — which an account adopts, by copy or by versioned reference, and
then governs as its own. **A project consumes the resources of the account
that owns its workspace:** its git flow parameterises the merge queue and
verification; its skills and workflow parameterise the agents' cards.

## What it cost

The grants table is generic from birth. Adopting a global resource requires
a versioning decision per kind, recorded in the resources spec.

## Since then

The workflow kind got its shape the same day, along with a refinement of
the defaults: a resource with a credential stays closed by default; a
content resource in an organization is open within the account by
default. An agent provider being a resource is what made
[a port per vendor](../agent-provider-as-port/) the natural design when the
runtime was decided.
