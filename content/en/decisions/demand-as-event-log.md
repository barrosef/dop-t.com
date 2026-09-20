---
title: "Every write is an event; everything else is a projection"
translationKey: "decision-0004"
adr: "0004"
adr_title: "The demand is an event log; everything else is a projection"
adr_file: "0004-demand-as-event-log.md"
date: 2026-08-29
weight: 4
group: "events"
description: "Every write in the core emits an immutable event carrying who, what, when and in which context. The dossier, the timeline, auditing, replay and metrics are read from that log — none of them is written separately."
related: ["0003", "0014", "0008"]
---

## What was on the table

Five needs each ask for a record of what happened on a demand: debugging
(replaying a demand step by step), auditing (who authorised this push,
with which credential), the dossier generated stage by stage, security
forensics, and metrics — interventions, rework, time to green. One record
has to serve all five.

## The paths we weighed

**One store per consumer** — a dossier table, an audit trail, a metrics
pipeline. Rejected: several writes, guaranteed divergence, and no replay.

**An unstructured text log.** Rejected: auditing in a multi-tenant system
needs fields, not grep.

## What we chose, and why

**Every write in the core emits an immutable event** into an append-only
log, in the same transaction as the state change. The envelope carries the
event's id, the account, the aggregate it belongs to and a readable
aggregate key (`account-created`, `pr-delivered`), the type, the payload,
the time — and the call's context: who acted (user, agent or platform),
the request, the session, the caller.

**Everything else is a projection.** The dossier, the timeline, auditing,
replay, metrics and the attention box are reads of the log, never writes
of their own. An action with no event is a defect.

## What it cost

The discipline of emitting everywhere. Volume that grows with the fleet,
handled by partitioning and retention in the persistence layer.

## Since then

The envelope grew its context fields — aggregate key, actor, request,
session, caller — on 2026-09-13, with the dead-letter path that carries the
full envelope when a consumer fails. Findings, cost metering and
notifications are event types rather than mechanisms, and
[caller verification](../the-core-verifies-its-callers/) exists so that an
event's authorship is worth something.
