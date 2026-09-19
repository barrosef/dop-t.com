---
title: "Four things to share, one way to share them"
translationKey: "decision-0009"
adr: "0009"
adr_title: "The resource as the account's unit of ownership and sharing"
adr_file: "0009-resource-as-unit-of-sharing.md"
date: 2026-08-30
weight: 9
group: "resources"
description: "Sharing existed for integrations only. Then skills, workflows and git flows asked for the same thing — and a mechanism per type would have been the mistake the account decision had just avoided."
related: ["0002", "0010", "0016"]
---

## What was on the table

The sharing model existed for one thing: integrations, with `use` and
`manage` grants composed in the invite. Then other things an account owns and
wants to share under authorization showed up. **Skills** — the agents'
reusable capabilities. **Human-agent workflows** — how a developer and the
agents collaborate on a demand. **Git flows** — branch governance as an
artifact: a taxonomy per card type, base and direction, release composition,
hotfix back-merge, policies — a concept lifted from real governance in
production, where the card's type decides the branch's prefix, base and flow.
And integrations themselves gained a third category: **agent providers**,
Claude and Codex among them.

A sharing mechanism per type would repeat the mistake the account decision
had just avoided: the same policy implemented N times, diverging.

## The paths we weighed

**One sharing mechanism per type.** Rejected: the same policy written four
times, with four screens and four bugs.

**Everything as an "integration".** Rejected: a skill and a flow have no
credential; they have a version and content. Forcing them into the wrong
entity would charge for it on every evolution.

## What we chose, and why

**`Resource` is the unit of ownership and sharing** — id, account, kind,
name, config, and an optional credential reference. The initial kinds:
`integration` (git, task manager, and now `agent`; the only one with a
credential), `skill`, `workflow`, `git_flow`.

The grant became per resource — `use`/`manage`, per user, composed in the
invite, editable at any time — and the existing rules did not change, they
generalised: a personal account's resource is private; only an organization's
is shareable; owners and admins hold an implicit `manage`; revoking `use` does
not tear down what is already configured.

The platform, at level zero, offers a catalogue of global resources —
providers, skills, default flows — which an account *adopts* and then governs
as its own. And a project consumes the resources of the account that owns its
workspace: the git flow attached to a project parameterises the merge queue
and the verification; the skills and the workflow parameterise the agents.

## What it cost

The grants table generalised, and the schema was born that way. Adopting a
global resource requires a versioning decision — copy or reference — per
type, recorded in the resources spec.

## Since then

The workflow resource got its shape the same day
([ADR-0010](../dynamic-workflow/)), which also refined the access default per
kind: a resource with a credential stays closed; a content resource in an
organization is open within the account by default. A credential is risk; a
flow is knowledge. And when the agent runtime was redesigned
([ADR-0016](../agent-provider-as-port/)), the fact that an agent provider was
already a resource — Claude and Codex as two rows, not two versions of the
code — was what made a port per vendor the natural shape rather than a new
idea.
