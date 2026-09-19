---
title: "The invite that could not carry its own key"
translationKey: "decision-0019"
adr: "0019"
adr_title: "The invite has no secret: identity in place of a bearer"
adr_file: "0019-invite-without-token.md"
date: 2026-09-01
weight: 19
group: "identity"
description: "A token that could not appear anywhere at rest, an e-mail that needed a link, and four places every event lands. The way out was to stop having a secret at all."
related: ["0002", "0004", "0018", "0020"]
---

## What was on the table

The invite was born the usual way: an opaque token, generated once, returned
to the caller, kept in the database only as a hash. Acceptance checked the
hash and nothing else. Whoever held the token got into the account.

That is a bearer credential, and it blocked the product in a way we had not
foreseen. The token could not appear anywhere it would sit at rest — and
"anywhere" is large here, because the payload of `invite.created` travels to
four destinations: the events table (partitioned, indefinite), the outbox
(until drained), JetStream (thirty days on disk) and the timeline projection,
which keeps the whole payload and is *made to be displayed*. Putting the token
in the event would replicate a credential to four places, one of them a
screen. Leaving it out meant the notifier — which only ever sees the event —
had no way to build an acceptance link. The e-mail announced an invite and
pointed at a list the invitee, who is not a user yet, could not open.

## The paths we weighed

**The token in the event.** A credential at rest, four times over, with no
revocation reaching JetStream or the timeline.

**A side path to the notifier.** A second delivery mechanism, existing only
so as not to use the first — and the secret would still exist, in fewer
places.

**Index the token by a UUID and send the index.** The owner's proposal, and it
pointed the right way — but on its own it changes nothing: if holding the UUID
is enough to accept, the UUID *is* the credential, in the same four places.

**Keep the token, and also require the e-mail to match.** It works, and it
keeps the hash of a secret nobody checks any more: a surface with no owner.

## What we chose, and why

The owner's second half was the key: *"the e-mail has to match too."* Once
acceptance requires being the invitee, the link stops being a credential and
becomes an **address**. So the invite stopped having a secret. The token
column was dropped, the generators disappeared, and creating an invite
returns nothing — there is nothing to return. What addresses an invite is the
row's id, which travels in plain text in the event, the timeline, the e-mail
and the log, because on its own it grants nothing.

Acceptance refuses when there is no session, when the invite is not usable,
when the signed-in person's e-mail is **not verified**, and when that verified
e-mail **is not the invite's**. The last one changed the nature of the thing —
and closed a hole that had nothing to do with the token: before, *any*
authenticated user holding the link got the role granted to somebody else.
The two new refusals are separate on purpose ("confirm your e-mail" and "this
invite is not yours" send a person to do different things), and the second
one never says whom the invite was for — saying so would make the link an
oracle, and the secret would return through the back door.

The e-mail's link is still data, not code: the notification rule carries
`/invites/{invite_id}`, resolved against the same data the template receives;
a test refuses a placeholder that is not in the data, and at run time a
missing field deletes the whole link rather than shipping half of one.

## What it cost

Somebody invited at one address who signs in with another — a personal
Google account against a corporate one — cannot accept. That is correct, the
invite is for one person, but it is a new wall and its message has to be
good. And an identity provider that does not report whether an e-mail is
verified makes every acceptance fail the safe way: noisily.

## Since then

The pattern set here — no bearer travelling in an inbox — is what the second
factor ([ADR-0020](../second-factor-in-the-core/)) used the next day to
refuse magic links: a code must be typed into the session that asked for it.
The acceptance screen the link points at was still to be built when this was
decided; it is recorded in the roadmap.
