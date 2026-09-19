---
title: "A channel without a trigger is a diffuse trigger"
translationKey: "decision-0018"
adr: "0018"
adr_title: "Communication: the trigger and the channel are born together"
adr_file: "0018-communication-trigger-and-channel.md"
date: 2026-08-31
weight: 18
group: "events"
description: "The risk was never a mailer with nothing to fire it. It was the opposite: if the trigger is not designed, every use case that sends an e-mail becomes one, with no name and no place."
related: ["0004", "0014", "0019", "0020"]
---

## What was on the table

The platform needs to tell people things outside the cockpit: an invite, an
account verification, a broken integration, an exceeded budget, a thread
waiting for an answer.

A sibling project solves this by writing a document into a collection, which
triggers a function, which sends through SendGrid. Three things from there
transfer: the local rehearsal with no key (it prints instead of sending), the
state kept on the record, and templates versioned in the repository. The
trigger does not transfer — here the event spine already exists, and the
spec says communication is *a consumer of the event spine, not a system
apart*.

## What we chose, and why

The owner put it in one sentence: *"A Mailer living without the Notifier
would be like a bullet that could be fired without the trigger."* The point
is not that the channel *can* live alone. It is that the trigger exists
either way: if nobody designs it, the invite use case calls the mailer
directly and *becomes* the trigger, with no name and no place, spread over as
many use cases as send e-mail. The risk was never a channel without a
trigger. It was a **diffuse** trigger.

So: an event consumer, the **Notifier**, decides *what* to notify and *to
whom*; it produces a command — kind, recipient, data — and the **Mailer**, a
channel port, fires it. The invite use case knows neither; it publishes an
event and that is all.

**The port is per channel, not one for everything.** E-mail has a subject,
HTML and attachments; push has a title, a badge and a deep link; SMS has 160
characters and no formatting. One port would carry the union of everything
or the lowest common denominator. So `Mailer` now, `Pusher` and `SMSer` when
push and SMS exist; one vendor may implement several.

**The adapter is big: index, resolution and sending.** This corrected the
initial proposal, which rendered templates in the domain. The port speaks
intent — "an invite was created, to this address, with this data" — and each
adapter decides what that becomes: SendGrid maps the kind to a template id;
SMTP renders locally from the repository's files. Rendering in the domain
would have looked cleaner and been worse: the platform could never use a
provider's template editor, and the port would carry a blob of HTML. It is
the same split the cost router makes — policy here, catalogue there. One
consequence needed a test: a kind may exist in the policy and have no
template in the vendor, and that fails in silence. The contract suite makes
every adapter resolve every kind the domain can emit.

**Two real adapters, SendGrid and SMTP.** SMTP is the self-hosted path, and
it is what *forces* local template resolution — proof that the port speaks
intent and not a vendor's template id.

**Transactional and attention are different.** An invite fires immediately,
always, one per event. A blocked thread or a PR waiting for review already
has a map — the attention box — and the e-mail hooks onto the box, not onto
raw events. And then the risk is spam: one e-mail per item makes an inbox
useless. So a **digest with a delay**, fifteen minutes by default: the item
opens, waits, and only becomes an e-mail if it is still open. Fifteen is an
informed guess, to be calibrated with telemetry.

**Idempotency is per (event, rule, action)**, not per event. With one action
per event, a key on the event alone works; the day one event triggers many
actions, it would discard the second as a duplicate — silently, by design.
The key was born composite for a future that was already planned.

## What it cost

SendGrid's visual editor is lost for the SMTP templates, which are files. The
same notice's template lives in two places while both adapters exist. And the
fifteen minutes is a guess: a notice too urgent waits, one too trivial
annoys, and only data settles it.

## Since then

The vocabulary earned its keep twice. The invite decision
([ADR-0019](../invite-without-token/)) changed the link the rule carries, and
nothing else. The second factor ([ADR-0020](../second-factor-in-the-core/))
used the *channel* and deliberately not the *trigger*: a code is a challenge,
not a notification, and it must not pass through digests and delays. A third
adapter — OneSignal, the one the record had pencilled in as "future" — was
written in September, and the e-mail verification flow was proven up to the
provider's door.
