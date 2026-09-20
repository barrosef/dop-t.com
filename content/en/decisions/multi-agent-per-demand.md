---
title: "Threads you can address, findings you can publish"
translationKey: "decision-0007"
adr: "0007"
adr_title: "Multi-agent per demand: addressable threads and published findings"
adr_file: "0007-multi-agent-per-demand.md"
date: 2026-08-29
weight: 7
group: "agents"
description: "A demand's conversation is a set of threads — the main agent and one per specialist — each with its own history and card. Specialists share what they found as structured findings, not by copying their whole context into each other."
related: ["0004", "0006", "0008", "0017"]
---

## What was on the table

A demand may need several agents at once: one implementing, one reading a
database forensically, one combing logs. The developer needs to talk to
each without mixing their timelines, and each needs to use what the others
found. Inside a demand the agents share a sandbox and cooperate; between
demands the boundary is hard.

## The paths we weighed

**A single sequential agent.** Rejected: no specialisation, no
parallelism.

**One sandbox per subagent.** Rejected: it breaks the shared workspace the
specialists need, multiplies cost, and isolates agents that cooperate.

**One shared timeline.** Rejected: separate timelines are the requirement.

## What we chose, and why

**The demand's conversation is a set of threads:** `#main` plus one per
subagent, each with its own history; the developer addresses one thread
at a time. **Every subagent has a card** — purpose, tools granted, the
model the router chose, a slice of the demand's budget.

**Cross-thread knowledge is by query, not by dump.** Threads are readable
by their siblings through tools; no timeline is copied into another's
context. **A conclusion is a published finding** — a structured result on
the demand's board that enters the siblings' context, is an event, and is
written to the project's memory.

Subagents are launched by the human, through the chat, or by the main
agent on its own initiative — an assumption the product may restrict. The
security boundary stays the demand: subagents share the sandbox, the
workspace, the credential and the quota.

## What it cost

The runtime supports several sessions per sandbox, and the attention box
becomes a prerequisite for scale rather than an option.

## Since then

The quarantine of raw tool output in the specialist's thread is also the
largest saving in [the cost decision](../llm-cost-governance/): the main
agent receives the finding, never the dump.
