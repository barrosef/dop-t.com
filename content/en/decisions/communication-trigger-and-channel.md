---
title: "A notifier decides; a channel delivers"
translationKey: "decision-0018"
adr: "0018"
adr_title: "Communication: the trigger and the channel are born together"
adr_file: "0018-communication-trigger-and-channel.md"
date: 2026-08-31
weight: 18
group: "events"
description: "One consumer of the event spine decides what to notify and to whom, from a table of rules. One port per channel delivers it, and each adapter — OneSignal, SendGrid, SMTP — resolves its own templates. Use cases publish events and nothing else."
related: ["0004", "0014", "0019", "0020"]
---

## What was on the table

The platform notifies people outside the cockpit: an invite, an account
verification, a broken integration, an exceeded budget, a thread waiting
for an answer. The event spine already exists; communication is a consumer
of it. Two things had to be decided: who decides that a notice goes out,
and what shape the delivery takes across channels that differ — e-mail has
a subject and HTML, SMS has 160 characters.

## The paths we weighed

**Use cases calling a mailer directly.** Rejected: the decision "what to
notify" would be scattered over every use case that sends e-mail, with no
name and no place.

**One port for every channel.** Rejected: it would carry the union of all
channels' fields, or the lowest common denominator.

**Rendering templates in the domain.** Rejected: the port would carry HTML,
and a provider's own template editor could never be used.

**One e-mail per attention item.** Rejected: noise.

## What we chose, and why

**Two components.** The **Notifier**, a consumer in the worker, decides
what to notify and to whom from a table of rules — event type, kind,
recipient resolution, data, link path — and emits a command. A **channel
port** delivers it: `Mailer` for e-mail, `SMSer` for SMS, `Pusher` when
push exists. Use cases publish events only.

**Template index, resolution and rendering live in the adapter.** The port
speaks intent — a kind and its data — and each adapter maps it to a
template: SendGrid to a template id, SMTP and OneSignal to templates in the
repository. The contract suite makes every adapter resolve every kind the
domain emits, so a missing template is a failing test rather than a silent
non-delivery. Without a credential configured, the adapter prints instead
of sending.

**Transactional notices** — an invite, a verification — send immediately,
one per event. **Attention notices** — a blocked thread, a PR awaiting
review, an exceeded budget — hook onto the attention box and go out as a
**digest after a delay**, fifteen minutes by default, only if the item is
still open.

**Idempotency is per (event, rule, action)**, so one event may trigger
several actions. **Link paths are data:** a rule's link accepts
placeholders resolved against the notification's data; an unresolvable
placeholder removes the link. The consumer runs in the core, where the
channel credentials are.

## What it cost

A template exists per adapter, and the contract suite is what keeps them
complete. The fifteen-minute delay is a starting value to be calibrated
with telemetry.

## Since then

Link paths as data were added on 2026-09-01 for the invite; the OneSignal
adapter joined SendGrid and SMTP in September. The second factor uses the
channels and deliberately not the notifier: a code is a challenge, not a
notification.
