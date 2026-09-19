---
title: "Two languages, one owner of the truth"
translationKey: "decision-0012"
adr: "0012"
adr_title: "A core in Go, a BFF in Python, and the boundary between them"
adr_file: "0012-stack-go-core-python-bff.md"
date: 2026-08-30
weight: 12
group: "foundations"
description: "Go for the state and the transactions, Python for the edge — and a boundary in one sentence: the BFF has no database. One of its clauses did not survive contact with the code."
related: ["0001", "0013", "0016"]
---

## What was on the table

Two natures of work were asking to live in the same backend. One is domain,
state and transactions: identity, resources, flows, demands, delivery, cost —
a gRPC server, event consumers, a daemon talking to Kubernetes. The other is
conversation with AI models: long sessions, token streaming, tools,
embeddings. Forcing both into one language charges somewhere. gRPC and
concurrency in Python are uncomfortable; the agent and embedding ecosystem in
Go is shallow.

## The paths we weighed

**Everything in Python.** Continuity with what already existed, one language.
Rejected for the core: a single binary that starts instantly on scale-to-zero,
cheap concurrency, a Kubernetes daemon — that is work Go is materially better
at.

**Everything in Go.** Coherent on the backend, but the edge would lose the AI
ecosystem — the agent SDKs, the embeddings, the tokenizers — which is the
product's whole point.

**TypeScript across the backend**, one language with the frontend. Rejected
for the same reason as Go: the most complete agent libraries are in Python.

## What we chose, and why

`dop-core` in Go: domain, state, transactions, events, orchestration; it
speaks gRPC to its callers and Postgres, NATS and the Kubernetes API to its
surroundings; one binary with four modes. `dop-api` in Python: protocol, the
agent session, the conversation with the models; REST with SSE for the
cockpit, gRPC for the CLI, and a gRPC client to the core.

The boundary fits in one rule, and the rule is the decision: **the BFF has no
database.** Not even "just a quick query". Two owners of a schema is how a
boundary dies. When the edge needs to record something, it calls the core,
which writes the state and the event in the same transaction.

## What it cost

Two toolchains, two lint and test setups, two image pipelines. Every call
from the edge to the core is a network call — it needs a deadline, a retry
and idempotency, so the contract carries an `idempotency_key` on every write.
Types exist at both ends, which is only bearable because both are generated
from the same source ([the next decision](../proto-as-source-of-truth/)).

## Since then

One clause of this decision was wrong and was replaced. The original text put
the agent runtime — the piece that talks to the model — in the Python edge:
*"the core decides what; the BFF runs the conversation"*. It looked clean on
paper. On implementation it turned out that the runtime needs the model
provider's credential, and the credential lives in the vault, in the core,
which never hands a secret out. The runtime moved to the core, and the
boundary gained its second half: the BFF has no database, **and no secret**.
That story is [ADR-0016's](../agent-provider-as-port/). What stayed intact
here is the split of languages and the sentence about the database.
