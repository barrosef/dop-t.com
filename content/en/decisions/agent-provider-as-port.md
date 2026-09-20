---
title: "A port per model vendor, running where the credential is"
translationKey: "decision-0016"
adr: "0016"
adr_title: "The agent runtime: a port per vendor, running inside the core"
adr_file: "0016-agent-provider-as-port.md"
date: 2026-08-31
weight: 16
group: "agents"
description: "The runtime that talks to a model sees no vendor SDK: it sends a conversation through the AgentProvider port and the account's resource decides which adapter answers. It runs in the core, next to the vault, so a provider's credential never crosses the network."
related: ["0001", "0008", "0009", "0012"]
---

## What was on the table

Two questions about the runtime — the component that converses with a
model. Which vendors it can talk to: an account may hold several agent
providers, Claude and Codex among them, and the platform must not be
written against one. And where it runs: it needs the provider's
credential, which lives in the vault inside the core and is never
returned by it.

## The paths we weighed

For the vendor: **a single adapter, abstracted later** — rejected, as for
every port; **an OpenAI-compatible API as the common denominator** —
rejected, because the compatibility ends exactly where the platform needs
precision: prefix caching, tool format, token counting.

For the place: **the edge with its own vault access** — rejected, the
edge is exposed to the internet and compromising it must not expose every
account's provider keys; **an ephemeral provider token issued by the
core** — impossible, provider keys are durable; **a separate runtime
service** — a third trust boundary for the same problem; **the credential
in an environment variable** — no per-account isolation, attribution or
revocation.

## What we chose, and why

**`AgentProvider` is a port with one adapter per vendor.** The runtime
sends a conversation — a stable prefix, messages, tools — and receives a
response with usage and a stop reason; no SDK type crosses the port. The
adapter is chosen **per request, by the account's resource**, with several
active at once — the second family of ports. The router chooses the class
of model; the active adapter's catalogue resolves the concrete name, so
the cost policy holds for every vendor without knowing any.

**The runtime runs in the core.** The credential is read from the vault
and used in the same process. A turn's operations — assemble the context,
route, record usage, post a message, publish a finding, enforce the
budget — are the core's own, in-process. The edge authenticates,
aggregates and translates; running a turn is one call, and live
follow-up is the existing SSE fed by the core's events.

What no vendor can guarantee stays out of the port and is documented on
it: prefix-cache semantics, tool-call format, streaming events, stop
reasons. Two adapters and a contract suite, like every port.

## What it cost

Prefix-cache behaviour differs per vendor, so the saving of the cost
decision is per vendor. The core makes long outbound calls, so connection
budgets and timeouts are its concern. And the edge's boundary rule gained
its second half: no database, **no secret**.

## Since then

Two records — the port and the runtime's location — were consolidated on
2026-09-04. "No secret" is one of the five invariants every contributor is
briefed on, and the reason the second factor keeps its seeds in the core.
