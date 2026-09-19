---
title: "Where the second factor lives is the decision"
translationKey: "decision-0020"
adr: "0020"
adr_title: "The second factor is the platform's, with three verifiers"
adr_file: "0020-second-factor-in-the-core.md"
date: 2026-09-02
weight: 20
group: "identity"
description: "The identity provider offered MFA for free. We built our own anyway — because the provider did not cover the requirement, would not survive a swap, and could never be exercised locally."
related: ["0001", "0016", "0018", "0019"]
---

## What was on the table

The product's requirement fixed the three second factors a person can choose:
an authenticator app, e-mail, and SMS. The question was not *whether* — the
platform is born with a second factor — but *where it lives*, and there were
two candidates.

**The identity provider's.** Firebase, our first identity adapter, has MFA.
Three things made it a bad home. It does not cover the requirement: its second
factor is SMS and TOTP, and e-mail does not exist there — so half the feature
would be ours anyway, and two mechanisms would decide the same thing. It does
not survive an adapter swap: MFA semantics differ in every provider —
enrolment, recovery, what the token asserts and how — so the account's
security would depend on which vendor is wired in. And the local environment
cannot exercise it: the emulator does not do TOTP enrolment, which means
shipping a security path that never runs locally — the divergence that had
already cost this platform two authentication failures.

**The platform's.** The TOTP seed is a credential, so it belongs in the
vault, which lives in the core. The mail channel already existed. The event
log already existed. What was missing was one domain and one channel.

## The paths we weighed

**Delegate MFA to Firebase Identity Platform.** The default answer, and the
right one in a product with a single identity provider and no e-mail factor.
Neither is true here.

**Accept a factor asserted by the provider** — a token whose `amr` says `mfa`
— as equivalent to ours. Convenient: somebody with 2FA on their Google
account would not do ours. Rejected for v1 because it creates two rulers for
one decision, which this platform keeps refusing; recorded as an open item for
the day the port can normalise the assertion.

**TOTP only.** Strongest and cheapest. Rejected because it excludes whoever
does not use an authenticator app — from the very protection.

**A magic link instead of an e-mail code.** A link is a bearer travelling in
an inbox, exactly what [the invite decision](../invite-without-token/) had
just removed. A code has to be typed into the session that asked for it.

## What we chose, and why

The second factor is a domain concept of the core, with one mechanism and
three verifiers; the identity provider does the first factor and nothing
else. A factor is born `pending` and only becomes `active` when the person
returns a code — a lock nobody has tested is discovered on the day of the
sign-in that fails. An e-mail factor requires a verified e-mail. Codes are
single-use and die on the first correct answer.

The code is a **challenge, not a notification**. The notification table's own
test — "does this deserve to interrupt the person outside the platform?" —
fails in both directions: nobody is being interrupted, and a 2FA code cannot
pass through a policy with digests, delays and recipients resolved by
membership. It uses the channel and not the trigger, which is the vocabulary
of [ADR-0018](../communication-trigger-and-channel/) applied in reverse.

SMS needed a new port, `SMSer`, born with two adapters and a contract suite
like every other — narrow on purpose: no subject, no HTML, 160 characters, a
destination in E.164.

What requires a fresh step-up: signing in, writing a credential into the
vault, changing a role, inviting, revoking, deleting an account. Reading is
not gated — a challenge on every request would be theatre and would teach
people to answer without reading. Revoking a factor drops every step-up of
that person in every session. Ten recovery codes, shown once, kept hashed:
without them a lost phone becomes a support ticket, and support becomes the
bypass. Five failures cool a factor off; and because an SMS costs money, the
*send* has its own two ceilings — sixty seconds between messages and five per
hour — because a loop that never answers would otherwise cost nothing to
whoever runs it.

## What it cost

Somebody with 2FA at their identity provider does it twice until the open
item is decided. SMS is the weakest of the three — NIST discourages it — and
the only one that costs per attempt, which also makes it the abuse surface.
Locally there is no SMS delivery: the adapter prints the code. And the core
trusting the edge's session identifier stopped being a background concern,
which is the thread [ADR-0022](../the-core-verifies-its-callers/) picked up
the next day.

## Since then

It shipped end to end on the day it was decided — the TOTP verifier written
against the standard library, the two channels, the step-up gate on four
operations, the contract, the edge and the cockpit's screens.
