---
title: "The core verifies a signature on every call"
translationKey: "decision-0022"
adr: "0022"
adr_title: "The core verifies a signature; it does not believe a header"
adr_file: "0022-the-core-verifies-its-callers.md"
date: 2026-09-03
weight: 22
group: "identity"
description: "A call with a person carries the person's token; a call without one carries an assertion signed by the platform. The network boundary is additional — Cloud Run IAM in the cloud, a network policy on a cluster — never the only guarantee."
related: ["0001", "0004", "0012", "0013"]
---

## What was on the table

The core resolves who is calling, and in which account, from the call's
metadata. A network boundary alone — only the edge may reach the gRPC port
— is a property of one deployment, and this platform runs agent code
inside the same cluster on purpose. The core needed a guarantee that
belongs to the software.

## The paths we weighed

**Trust the metadata.** Rejected: whoever can open a connection claims any
actor of any account.

**A shared secret in a header.** Rejected: it proves that the caller knows
a secret, not what the caller claims.

**mTLS.** Rejected on clusters: issuance and rotation, and it proves the
connection rather than the claim. On Cloud Run its equivalent — IAM
invoker permission — costs neither, and is used.

**An assertion signed by the platform.** Adopted for calls with no person
behind them.

**Forwarding the person's token.** Adopted for calls with a person: the
signature is the identity provider's, an authority neither end controls,
and the core already had the machinery to verify it.

**Identity-Aware Proxy.** Rejected: it authorises by IAM policy, which
cannot express per-tenant accounts, roles and grants, and it breaks a
cross-origin single-page cockpit. It fits team-only surfaces.

## What we chose, and why

**Which signature depends on who is calling.** A call with a person
carries the person's token, forwarded whole; the core verifies it through
its identity port and takes the actor from the token's subject. A call
without a person carries an assertion the edge signs — caller, actor,
kind, account, session, expiry — with one key per caller, so a compromised
component forges only its own calls, and the signature covers the claim,
so a stolen assertion is worth one actor in one account for two minutes.

**The account always comes from the assertion**, never from the token,
which does not carry it. **Token and assertion naming different actors
refuse the call.** A refused call proceeds with no actor and fails at
authorisation, where the message means something.

**Three modes:** `strict` refuses to fill in an actor without a verified
signature, `permissive` warns and proceeds, `off` trusts the metadata. The
code's default is permissive, so a clone works without an edge in front;
the deployment runs strict.

**Transport authentication is additional.** On Cloud Run the core accepts
no unauthenticated invocation and only the edge's service account may
invoke it — the invoker token travels in `X-Serverless-Authorization`
because `Authorization` carries the person's. On a cluster a network policy
restricts who reaches the port. Neither replaces the signature.

## What it cost

The edge forwards the raw token and signs an assertion on every call. One
HMAC and one token verification per call, with signing keys and the
subject lookup cached. A two-minute expiry assumes the two clocks agree.
And two things this design does not solve: an assertion is replayable
within its window, and a compromised edge signs its own claims — both
inherent, both the reason the person's token is preferred wherever a
person exists. The wire format is pinned by one test vector asserted in Go
and in Python.

## Since then

Cloud Run IAM took the place of the network policy in the managed
deployment on 2026-09-07; the cluster keeps the policy.
