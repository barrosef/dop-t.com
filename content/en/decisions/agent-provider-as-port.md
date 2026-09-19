---
title: "The runtime was on the wrong side of the credential"
translationKey: "decision-0016"
adr: "0016"
adr_title: "The agent runtime: a port per vendor, running inside the core"
adr_file: "0016-agent-provider-as-port.md"
date: 2026-08-31
weight: 16
group: "agents"
description: "Two mistakes caught in the same week: a briefing that said 'call Anthropic's API', and a clean-looking split that put the piece needing the model's credential in the layer open to the internet."
related: ["0001", "0008", "0009", "0012"]
---

## What was on the table

**The vendor.** When the agent runtime — the piece that talks to the model —
was specified, the briefing said "write code that calls Anthropic's API".
That was wrong, and the owner caught it: *"the platform has to think about
isolation with multiple provider options, including agent providers; for
example Claude and Codex."* It was the ports decision applied to the place
where it is easiest to forget, because the model vendor looks like the
product and not like infrastructure. The data model had already anticipated
it and nobody had connected the dots: an agent provider was already an
integration of category `agent` — a resource with a credential — and the
cost router already separated policy (which class) from catalogue (which
concrete model).

**The place.** The stack decision had put the runtime in the Python edge:
the core decides what, the edge runs the conversation. Clean on paper. On
implementation it charged: the runtime needs the provider's credential, and
a resource's credential lives in the vault, in the core, which never hands a
secret out — by design, with a test guarding it.

## The paths we weighed

For the vendor: **a single adapter, swap later** — what the ports decision
exists to prevent; it is how the identity port spent months with one adapter
and hid an authentication bypass until somebody wrote the second. **An
OpenAI-compatible layer** as the common denominator — rejected: the
compatibility covers the simple case and leaks exactly where the platform
needs precision: prefix caching, tool format, token counting.

For the place, every way out was bad. **The edge with its own access to the
vault** — the implementation's initial recommendation, and the owner vetoed
it rightly: *"the BFF is a very insecure layer, open to the internet."*
Compromising it would hand over the agent credentials of every account; the
session that discussed this had just found a total authentication bypass in
that very layer. **The core issuing an ephemeral token** — elegant, and
impossible: an API key is durable, there is nothing short-lived to issue.
**A third service just for the runtime** — a third deployment and a third
trust boundary for the same problem. **Reading from an environment
variable** — the stopgap that had shipped: no per-account isolation, no cost
attribution, no revocation.

## What we chose, and why

**`AgentProvider` is a port with one adapter per vendor.** The runtime never
sees an SDK type: it sends a conversation — a stable prefix, messages, tools
— and receives a response with usage and a stop reason. And unlike the
boot-time ports, the adapter is chosen **per request, by the resource** —
several active at once, the way one account has a project on Jira and
another on ClickUp. The router chooses the class; the active adapter resolves
the model's name; the cost policy holds for every vendor without knowing any.

**The runtime runs in the core.** The credential never crosses a network
boundary: read from the vault, used in the same process. And what the
implementation revealed weighed as much as the security: the runtime was
already almost entirely the core's orchestration. The six things it does in
a turn — assemble context, route the model, record consumption, post a
message, publish a finding, respect the budget — are all core operations,
done from outside over gRPC. Two modules existed only because of the
boundary and disappeared: one working around an inaccessible vault, one
redoing the way back from a model's name to its class.

## What it cost

Prefix-cache semantics are not the same between vendors, and the saving
depends on them — the most expensive divergence, documented on the port.
Tool-call formats, streaming events and stop reasons diverge too; whatever
all of them cannot meet stays out. Two adapters and a contract suite from
the start, the cost this platform had already paid three times. The
provider adapters were rewritten in Go — some six hundred lines; the design
survived, the language changed. And the core now makes long external calls,
so connection budgets and timeouts became its concern.

## Since then

The edge gained an invariant stronger than "no database": **no secret.** It
is now one of the five walls the coding agents are briefed on before they
touch anything, and the second factor ([ADR-0020](../second-factor-in-the-core/))
used it two days later as the reason the TOTP seed lives in the core.
