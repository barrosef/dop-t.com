---
title: "A header is not a proof"
translationKey: "decision-0022"
adr: "0022"
adr_title: "The core verifies a signature; it does not believe a header"
adr_file: "0022-the-core-verifies-its-callers.md"
date: 2026-09-03
weight: 22
group: "identity"
description: "The core believed two lines of gRPC metadata. Whoever could open a connection was any actor of any account. The only thing in the way was a network policy — a property of the deployment, not of the software."
related: ["0001", "0004", "0012", "0013"]
---

## What was on the table

The core read `x-actor-id` and `x-account-id` from the call's metadata and
believed them. Metadata is text: whoever could reach port 9090 could declare
themselves any actor of any account, with nothing beyond the header.

That had been a deliberate choice — one trust boundary, at the edge, and a
simple core. What put it on the table was the *kind* of guarantee behind it.
"Only the BFF can call" was enforced by a NetworkPolicy, and we verified on
2026-09-03, from a loose pod in the namespace, that it held: the gRPC port
blocked, the health port open. It held *today, here*. Three things made that
insufficient: it is a property of the deployment, not of the software — a
cluster whose CNI ignores network policies has no boundary at all; it fails
in silence — nothing breaks, the door is simply open; and this platform runs
agent code inside the same cluster on purpose. "Someone hostile inside the
network" is not a hypothesis here. It is a feature.

## The paths we weighed

Discussed with the owner side by side, on a canvas. **Keep as is:** cost
zero, and one misconfiguration is the whole system. **A shared secret in a
header:** cheap, and it proves only that the caller knows the secret — the
holder still claims any actor. **mTLS:** strong, no bearer, expensive in
issuance and rotation — and it proves the *connection*, while the question
was about the *claim*. **An assertion signed by the platform:** it binds the
claim, works where there is no person, and is cheap to verify. **Forwarding
the person's JWT** — the owner's proposal, and the strongest where it
applies: the signature is the identity provider's, an authority neither end
controls, and the core already had the machinery to verify it. It covers only
calls with a person behind them.

The question that decided it: not *who opened the connection*, but *who has
the authority to assert who the actor is*.

## What we chose, and why

The core verifies a signature on every call, and which signature depends on
who is calling. A call with a person carries the person's token, forwarded
whole; the core verifies it through the port it already had, and the actor
comes from the token's subject. A call with no person carries an assertion
the edge signs — caller, actor, kind, account, session, expiry — with one key
per caller, so a compromised component forges only its own calls, and the
signature covers the *claim*, so a stolen assertion is worth one actor in one
account for two minutes.

The account never comes from the token — it is not in there — always from
the assertion. When both are present and name different actors, the call is
refused: that is not a preference between sources, it is a bug or an attack.
A refused call goes on with no actor and fails at authorization, where the
message means something; failing in the interceptor would say "unauthenticated"
about a call whose real problem is that it proved nothing.

Three modes, and the default is not the strict one: `permissive` warns,
`strict` refuses, and the deployment runs strict while the code stays usable
for whoever clones it. Flipping everything at once would have broken every
caller not yet taught to sign — the contract suite included.

## What it cost

The edge changed too: it carries the raw token to forward it, and signs an
assertion on every call. The wire format is pinned by a shared test vector,
asserted in a Go test and a Python test, because two languages only agree by
accident — and a drift here does not fail loudly; it makes every call arrive
unauthenticated. One HMAC and one token verification per call, with the
signing keys and the subject lookup cached. A two-minute expiry assumes two
clocks agree, which in a cluster they do; it is written down for the day they
do not. And what is not solved: an assertion is replayable within its window,
and a compromised edge signs whatever it likes — both inherent in signing
one's own claims, both the reason the third-party token is preferred wherever
a person exists.

## Since then

On 2026-09-07, moving to Cloud Run changed the objection to mTLS. IAM invoker
permission is the mTLS-shaped answer with no issuance and no rotation: Google
signs, Google verifies, before the request reaches the process. It replaced
the NetworkPolicy — which stays in the local cluster, where there is no IAM —
and it replaced nothing decided above: three layers answer three questions
(may this caller invoke this service; which component asserts which actor;
who is the person), and none answers another's. One collision had to be
handled: Cloud Run reads its token from `Authorization`, which the person's
token already occupies, so the service token travels in
`X-Serverless-Authorization`.

Identity-Aware Proxy was weighed and refused — and an earlier draft of that
amendment was wrong about why. It is not cost: IAP's external-identities mode
is free to fifty thousand users. It is that IAP decides who may reach a
resource by IAM policy, and this platform lets anyone sign up: the policy
would admit everyone and decide nothing, while the question that matters —
which account, which role, which grants — is per-tenant state IAM has no
vocabulary for. It would also break the cockpit, a single-page app on another
origin that cannot follow a redirect to a sign-in page. Where IAP fits is a
surface only the team uses.
