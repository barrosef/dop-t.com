---
title: "One second factor, three verifiers, in the core"
translationKey: "decision-0020"
adr: "0020"
adr_title: "The second factor is the platform's, with three verifiers"
adr_file: "0020-second-factor-in-the-core.md"
date: 2026-09-02
weight: 20
group: "identity"
description: "The identity provider does the first factor only. The second — authenticator app, e-mail or SMS — is a domain of the core, so it is the same for every identity provider, runs locally, and keeps its secrets in the vault."
related: ["0001", "0016", "0018", "0019"]
---

## What was on the table

The product offers three second factors: an authenticator app (TOTP),
e-mail and SMS. The second factor had to behave identically whichever
identity provider is wired in, be exercisable in the local environment,
and keep its secrets where the platform keeps secrets.

## The paths we weighed

**The identity provider's MFA.** Rejected: it offers no e-mail factor,
its semantics differ per provider — enrolment, recovery, what the token
asserts — and the local emulator cannot exercise it.

**Accepting a factor the provider asserts** (a token whose `amr` says
`mfa`) as equivalent to ours. Deferred: two rulers for one decision; open
for the day the identity port can normalise the assertion.

**TOTP only.** Rejected: it excludes whoever has no authenticator app.

**Magic links** instead of an e-mail code. Rejected: a bearer travelling
in an inbox. A code is typed into the session that asked for it.

## What we chose, and why

**The second factor is a domain of the core.** A factor is born `pending`
and becomes `active` only when the person returns a valid code — enrolment
proves possession. The TOTP seed is a reference into the vault and is
never returned after enrolment; an e-mail factor requires a verified
e-mail.

**Three verifiers:** TOTP per RFC 6238 (30-second step, six digits, a
±1 window); a six-digit code valid for ten minutes over e-mail; the same
code over SMS. A code dies on the first correct answer.

**A code is a challenge, not a notification.** It uses the channel ports
directly and never the notifier — no digest, no delay, no recipient
resolved by membership. SMS needed its own port, `SMSer`, narrow by design
(160 characters, an E.164 destination), with two adapters and a contract
suite like every other.

**Step-up is recorded per user and session**, with an expiry. It is
required to sign in when a factor is active, to write a credential into
the vault, to change a role, invite or revoke, and to delete an account.
Reads are not gated. Revoking a factor drops every step-up of that person
in every session. Ten recovery codes, shown once, stored hashed.

**Limits:** five consecutive failures cool a factor off; because an SMS
costs money, sending has its own ceilings — one message per minute and
five per hour, per factor. An organization may require a second factor of
its members; the requirement gates operating in that account, not the
person's personal one.

## What it cost

Someone with MFA at their identity provider verifies twice until the
deferred item is decided. SMS is the weakest of the three and the only one
that costs per attempt, which makes it the abuse surface; the account
policy can disable it. Locally there is no SMS delivery — the adapter
prints the code.

## Since then

Shipped end to end on the day it was decided. The session identifier the
step-up relies on has been verified by the core since
[the caller decision](../the-core-verifies-its-callers/) the next day.
