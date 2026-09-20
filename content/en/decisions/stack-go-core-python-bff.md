---
title: "A core in Go, a BFF in Python, and one rule between them"
translationKey: "decision-0012"
adr: "0012"
adr_title: "A core in Go, a BFF in Python, and the boundary between them"
adr_file: "0012-stack-go-core-python-bff.md"
date: 2026-08-30
weight: 12
group: "foundations"
description: "Go for state, transactions, events and the agent runtime; Python for the edge that talks to the cockpit and the CLI. The boundary fits in one sentence: the BFF has no database and no secret."
related: ["0001", "0013", "0016"]
---

## What was on the table

Two natures of work share the backend. One is domain, state and
transactions: a gRPC server, event consumers, a daemon that talks to
Kubernetes, and — since the runtime moved into it — long conversations with
model providers. The other is the edge: authenticating the person,
translating protocols for the cockpit and the CLI, aggregating. Each is
comfortable in a different language.

## The paths we weighed

**Everything in Python.** One language; rejected for the core, where a
single binary that starts instantly on scale-to-zero and cheap concurrency
matter more.

**Everything in Go.** Coherent; rejected for the edge, where the Python
ecosystem for agent SDKs and embeddings is the richer one.

**TypeScript across the backend**, one language with the frontend.
Rejected for the same reason as Go.

## What we chose, and why

`dop-core` in Go: domain, state, transactions, events, orchestration and
the agent runtime; one binary with four modes — `serve`, `worker`, `sched`,
`launcher`. `dop-api` in Python: REST with SSE for the cockpit, gRPC for the
CLI, and a gRPC client to the core.

The boundary is one rule, and it is one of the five invariants every
contributor is briefed on: **the BFF has no database and no secret.** No
connection from the edge to Postgres; no credential in its process or its
configuration. When the edge needs to record something, it calls the core,
which writes the state and the event in one transaction. The sandbox, in
turn, talks only to the edge and to the platform's git server.

Every call from the edge to the core is a network call, so it carries a
deadline, a retry policy and — on writes — an idempotency key.

## What it cost

Two toolchains, two test and lint setups, two image pipelines. Types exist
at both ends, which is bearable only because both are generated from the
same contract.

## Since then

The agent runtime was placed in the core on 2026-08-31, which is also when
"no secret" joined "no database" in the boundary rule. The reasoning is in
[the agent provider decision](../agent-provider-as-port/).
