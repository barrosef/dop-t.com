---
title: "One database, an outbox, and a queue for what fails"
translationKey: "decision-0014"
adr: "0014"
adr_title: "PostgreSQL as the single database, and the outbox that moves its events"
adr_file: "0014-postgres-persistence.md"
date: 2026-08-30
weight: 14
group: "events"
description: "PostgreSQL holds the relational spine, the documents and the vectors. Every state change writes its event in the same transaction; a relay publishes it to NATS; a delivery that keeps failing lands in one dead-letter queue with everything needed to retry it."
related: ["0004", "0001", "0018"]
---

## What was on the table

Three natures of data — relational (accounts, memberships, grants,
projects), documental (flows, cards, event payloads) and semantic (the
memories the agents search) — and two requirements: a state change and
its event are atomic, and the processes that react to events are
decoupled from the one that wrote them.

## The paths we weighed

**MongoDB.** Rejected: relational integrity would move into the
application.

**Postgres plus a dedicated vector store.** Rejected until volume requires
it.

**Kafka as the log.** Rejected: the log lives in the database; a broker
transports.

**Google Pub/Sub.** Kept as a second adapter of the bus port, not the
default.

**Redis Streams.** Rejected: weaker guarantees, an extra service.

**Publishing without an outbox.** Rejected: an event is lost whenever the
process dies between the write and the publish.

## What we chose, and why

**PostgreSQL for everything**, the same engine in both environments.
Relational integrity is in the schema — every domain table carries the
account, and isolation is a constraint, not a convention. JSONB for flows,
cards and payloads. pgvector for the memories. The event log in an
append-only table partitioned by month. Projections start as materialised
views.

**A transactional outbox.** Every state change writes the new state and
the event in one transaction; a relay reads the outbox and publishes to
the broker — at-least-once delivery, no distributed commit.

**NATS JetStream** carries the events: one container, identical locally
and in the cloud, persistent, with replay. Consumers are idempotent and
build the projections. A delivery is retried up to five times with
backoff — one second, five, fifteen, one minute. **An exhausted delivery
goes to one dead-letter queue** carrying the full event envelope, the
consumer and the attempt history; a dedicated consumer retries it three
more times, every attempt is recorded in an error ledger, and each failure
is classified recoverable, irrecoverable or unknown — a classification
learned from the error's signature and demoted by a later success.

**Long processes are sagas** orchestrated by the core: each step an event,
failures compensating or escalating to the attention box, state in
Postgres, progress reaching the cockpit over SSE.

## What it cost

One database that also carries the event load, so partitioning, retention
and vigilance are ours. Vector search has a ceiling in Postgres. The relay
polls, adding latency between commit and publication. Every consumer must
be idempotent. And NATS is one more thing to run.

## Since then

The dead-letter path — the queue, the classification and the error ledger
— was specified in detail and built on 2026-09-13, along with the context
fields of the event envelope.
