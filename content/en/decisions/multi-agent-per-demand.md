---
title: "A subagent you can talk to"
translationKey: "decision-0007"
adr: "0007"
adr_title: "Multi-agent per demand: addressable threads and published findings"
adr_file: "0007-multi-agent-per-demand.md"
date: 2026-08-29
weight: 7
group: "agents"
description: "In the market a subagent is a black box: you dispatch it and wait. We wanted one with a thread of its own, interrogable in flight — and a way for specialists to share what they found without dumping their whole context on each other."
related: ["0004", "0006", "0008", "0017"]
---

## What was on the table

A real demand may need specialists at the same time. The case that shaped
it came from the product's own history: the main agent implements; one
subagent does a forensic reading of the database through the workspace's
MySQL tool; another combs the server's logs. The developer needs to talk to
all three without mixing their timelines, and the three need to use each
other's conversations as knowledge.

Two axes had to be kept apart. Between demands, the boundary is hard — one
demand, one microVM. Inside a demand, N agents share a sandbox and
collaborate. This decision is about the second. And there was nothing to
copy: an addressable subagent, with its own thread and interrogable in
flight, did not exist in the tools of the day.

## The paths we weighed

**A single sequential agent.** Rejected: it loses specialisation and
parallelises nothing.

**One sandbox per subagent.** Rejected: it breaks the shared workspace — the
forensic agent needs the same database the main agent brings up —
multiplies cost, and buys no isolation that matters, because the agents
cooperate.

**A single shared timeline.** Rejected: that is the problem the requirement
came to solve.

## What we chose, and why

The demand's conversation is a **set of threads**, not a timeline: `#main`
plus one per subagent, each with its own history; the developer enters one
and talks to that agent in isolation. Every subagent is born with a **card**
— purpose, tools granted, the model the router chose, a slice of the
demand's budget.

**Cross-knowledge by query, not by dump.** Threads are readable by their
siblings as a tool — read a thread, ask a question. Dumping whole timelines
into every agent's context neither scales nor stays safe; it widens the
surface for injected instructions.

**A conclusion becomes a published finding** — a structured result on the
demand's common board: "a deadlock on table X between 14:02 and 14:07,
caused by migration Y". Findings go automatically into the siblings'
context, are events, and feed the project's memory.

Both the human and the main agent may launch subagents — the agent on its
own initiative when it judges it necessary, the thread appearing immediately
for the developer to follow or step into. That initiative is a recorded
assumption, adopted for coherence with autonomy; the product may restrict it.
And the security boundary is still the demand: subagents share the microVM,
the workspace, the credential and the quota. They are collaborators, not
strangers.

## What it cost

The runtime has to support N sessions per sandbox. And more threads means
more points of attention: the attention box stops being optional and becomes
a prerequisite for scale.

## Since then

The finding turned out to be more than a UX device. When the cost decision
([ADR-0008](../llm-cost-governance/)) looked for where an agent loop wastes
money, the biggest saving was already here: logs, dumps and voluminous reads
stay quarantined in the specialist's thread, and the main agent receives the
finding. Half of the token economy came from a decision taken for a different
reason.
