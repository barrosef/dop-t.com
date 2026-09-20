---
title: "One database, and the window we had to close"
translationKey: "decision-0014"
adr: "0014"
adr_title: "PostgreSQL as the single database, and the outbox that moves its events"
adr_file: "0014-postgres-persistence.md"
date: 2026-08-30
weight: 14
group: "events"
description: "Three natures of data, one engine. And the naive way to publish an event — write, then tell the broker — has a gap where a process can die between the two. The outbox closes it without a distributed commit."
related: ["0004", "0001", "0018"]
---

## What was on the table

Two questions, and they turned out to be one subject.

**What to store it in.** An early conversation had signalled "an adequate
database, like Mongo for example". When the backend was actually designed,
the workloads became clear, and they are of three natures: relational —
accounts, memberships, grants, projects; documental — flows, agent cards,
event payloads; and semantic — the memories the agents search.

**How to get an event out of it.** The product asked for decoupled
processes, strong resilience and atomic event-based transactions. The naive
way — write to the database, then publish to the broker — has a window: the
process dies between the two, the state changed, nobody heard. A distributed
commit solves it and charges dearly in complexity and availability.

## The paths we weighed

**MongoDB.** Good for documents, bad for the relational work that dominates
the domain; multi-tenant integrity would become the application's job — the
wrong place for it.

**Postgres plus a dedicated vector store.** Rejected on YAGNI: one more
service to operate before there is volume to justify it.

**Postgres plus Kafka as the log.** Rejected: the log lives in the database;
the broker transports, it does not hold the truth. And Kafka alone is a truck
for our load.

**Google Pub/Sub.** Managed and good, and it ties us to one cloud — it stays
as a second adapter of the bus port for whoever wants a managed one.

**Redis Streams.** Without JetStream's guarantees, and it brings a service we
do not need — there is no cache requirement.

**Publishing straight from the code.** That is the window.

## What we chose, and why

**PostgreSQL for everything.** Relational in the spine — multi-tenant
integrity is a foreign key and a constraint, not a convention, and every
domain table carries an `account_id`. JSONB for flows, cards and event
payloads. pgvector for the memories, with no extra store. The event log in an
append-only table partitioned by month. Projections start as materialised
views and become worker-fed tables only if the cost demands it. And the same
engine in both worlds: Cloud SQL on Google, CloudNativePG on a cluster.

**A transactional outbox.** Every state change writes, in the same
transaction, the new state and the event. Commit means atomic, by
construction. A relay reads the outbox and publishes to the broker —
at-least-once delivery, no two-phase commit.

**NATS JetStream** as the broker: one container, identical on k3s and on a
managed cluster, persistent, with consumer groups and replay. Consumers are
idempotent and build the projections; a poisoned message goes to the
dead-letter queue.

**Long processes are sagas** orchestrated by the core — the demand's
finalisation is the canonical one: each step an event, a failure compensates
or escalates, the state in Postgres. The person perceives it as synchronous
because the edge pushes progress over SSE; the execution survives restarts.

## What it cost

The event load concentrated in the same database, so partitioning, retention
and vigilance are ours. Vector search in Postgres has a ceiling, and when it
arrives it is extracted behind the same port. The relay polls, so there is
latency between commit and publication. Idempotency becomes an obligation
for every consumer. And NATS is one more thing to run.

## Since then

The sentence "a poisoned message goes to the DLQ" was written as if the queue
existed. On 2026-09-13 we found that it did not: on exhaustion the adapter
called `Term()`, which discards, under a log line claiming the message had
been saved. The stream's thirty-day retention was the only thing between an
exhausted event and nothing at all. The queue, the classification of
failures and the error ledger were designed and built that week, and the
record now states them as built, with a dated revision line. Writing down what you *believe*
exists is how you find out that it does not.
