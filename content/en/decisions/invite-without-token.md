---
title: "An invite is an address, not a key"
translationKey: "decision-0019"
adr: "0019"
adr_title: "The invite has no secret: identity in place of a bearer"
adr_file: "0019-invite-without-token.md"
date: 2026-09-01
weight: 19
group: "identity"
description: "An invite carries no secret. Its link is the row's id, safe to travel in events, projections and e-mails, because accepting requires signing in with the verified e-mail the invite was sent to."
related: ["0002", "0004", "0018", "0020"]
---

## What was on the table

An event's payload is replicated to four places — the event table, the
outbox, the broker's stream and the timeline projection, which is made to
be displayed. Nothing that grants access on its own may travel in an
event. The invite e-mail, produced by a consumer that only sees the event,
still needs a link the invited person — not yet a user — can open.

## The paths we weighed

**A secret token in the event.** Rejected: a credential at rest, replicated
to four places, with no revocation reaching the stream or the timeline.

**A side channel to the notifier**, outside the log. Rejected: a second
delivery mechanism, and the secret would still exist.

**A UUID indexing the token**, with only the index in the event. Rejected
on its own: if holding the index is enough to accept, the index is the
credential.

**Keep the token and also require the e-mail to match.** Rejected: it keeps
the hash of a secret nobody checks.

## What we chose, and why

**The invite has no secret.** It is addressed by its row `id`, which may
appear in events, projections, e-mails and logs, because on its own it
grants nothing.

**Accepting requires being the invitee.** Acceptance refuses when there is
no session, when the invite is no longer usable, when the signed-in
person's e-mail is not verified, and when that verified e-mail is not the
one the invite was sent to. The last two are separate errors — "confirm
your e-mail" and "this invite is not yours" send a person to do different
things — and the second never reveals whom the invite was for.

The e-mail's link is `/invites/{invite_id}`, produced by the notification
rule as data: a placeholder that cannot be resolved removes the link rather
than shipping half of one. The invite expires in fourteen days.

## What it cost

Somebody invited at one address who signs in with another — a personal
account against a corporate one — cannot accept; the message has to say
so. And an identity provider that does not report whether an e-mail is
verified makes every acceptance fail the safe way.

## Since then

The acceptance screen was built the next day, with the second factor. The
same principle — no bearer travelling in an inbox — is why the second
factor uses codes typed into the session rather than magic links.
