---
title: "Five needs, one log"
translationKey: "decision-0004"
adr: "0004"
adr_title: "The demand is an event log; everything else is a projection"
adr_file: "0004-demand-as-event-log.md"
date: 2026-08-29
weight: 4
group: "events"
description: "Debugging, auditing, the dossier, security, metrics — five things that each wanted a record of what happened. Writing the same data five times would have guaranteed they disagreed."
related: ["0003", "0014", "0008"]
---

## What was on the table

Five distinct needs each asked for a record of what happened on a demand.
**Debugging**: reproducing, step by step, a demand that went wrong.
**Auditing**: in an organization, answering "who authorised this push, with
which credential?" — the question the credential decision had just made
askable. **The dossier**: the requirements asked for it "generated at
runtime, stage by stage", not assembled at the end. **Security**: forensics
and detection when malicious content tries to divert the agent. **Metrics**:
human interventions, rework, time to green.

Building five mechanisms is writing the same data five times and watching
them diverge.

## The paths we weighed

**One store per consumer** — a dossier table, an audit trail, a metrics
pipeline. Rejected: a triple write, guaranteed divergence, and replay never
arrives.

**An unstructured textual log.** Rejected: it is neither queryable nor
projectable. Auditing in a multi-tenant system needs fields, not grep.

## What we chose, and why

**Every action on a demand emits an immutable event** into an append-only
log: when, who — human, agent or subagent —, what, which credential, a
summary of the input, the result. The log is the demand's spine, and the
dossier, the timeline, auditing, replay and metrics are **projections** of
it: reads, never writes of their own.

It set a rule for the persistence decision before it was taken: the event log
is a first-class citizen of storage, and whatever holds documents serves the
projections; it does not replace the log.

## What it cost

The discipline of emitting everywhere. An action with no event is a bug, not
a detail. And volume: the log grows with the fleet, so retention and
compaction became a question for the persistence decision.

## Since then

This is the decision the rest of the platform kept cashing in. The
subagents' findings and the cost metering became two more event types rather
than two mechanisms. The attention box became a projection. The invite's
security analysis ([ADR-0019](../invite-without-token/)) was made possible by
knowing exactly the four places an event lands. And the caller-verification
decision ([ADR-0022](../the-core-verifies-its-callers/)) was argued from
here: an event's authorship is only worth what the actor behind it is worth.

In September the events themselves grew up. The envelope gained its context
— the aggregate's key, the actor, the request, the session, the caller — and
the dead-letter queue that [ADR-0014](../postgres-persistence/) had described
as existing was finally built, after we found that an exhausted event had
been discarded under a log line claiming it was saved.
