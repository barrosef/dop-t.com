---
title: "Two adapters, or it is not a port"
translationKey: "decision-0001"
adr: "0001"
adr_title: "Infrastructure behind ports with pluggable adapters"
adr_file: "0001-infrastructure-behind-ports.md"
date: 2026-08-29
weight: 1
group: "foundations"
description: "The first decision, and the one every other decision leans on: the domain never touches a vendor, and a port with a single adapter is a guess shaped like that vendor."
related: ["0012", "0016", "0020"]
---

## What was on the table

The platform had to run in two places from the start: on Google Cloud Run, and
on a Kubernetes cluster somebody else operates — k3s, Rancher, OKD. The list of
things it would need from its surroundings was going to grow: secrets,
identity, object storage, persistence, messaging. And in each place the same
need is met by a different service. Secret Manager on one side; a Kubernetes
Secret on the other.

The tempting move was to pick Google, ship, and "port later". We had watched
what that does: *later* is the moment the coupling is already everywhere, and
the first choice — made under pressure, on day one — becomes the design.

## The paths we weighed

**Couple to GCP now, abstract later.** Faster to a first result. Rejected,
because running on a laptop cluster was not a future ambition, it was a
development requirement from the first week.

**A generic multi-cloud library.** Rejected for a subtler reason: such a
library gives you the common denominator *of the library's vendors*, not of
your domain. It trades one coupling for another, and the new one is harder to
see.

**Ports and adapters — with rules attached.** This won, but only because we
were honest that "hexagonal" is usually the name of a folder. The word does
nothing by itself.

## What we chose, and why

The domain reaches infrastructure only through a port it defines itself, in
its own language: `SecretStore.get(ref)`, not `accessSecretVersion`. Adapters
are chosen in one composition root, by configuration, and the domain imports no
vendor SDK.

Three disciplines make that real rather than decorative:

1. **Two adapters per port, from day one.** The local adapter is not "for
   later" — it is the proof that the port is right. A port with a single
   adapter comes out shaped like the vendor that inspired it.
2. **One contract test suite per port**, which every adapter has to pass.
   Substitutability in fact, not in intention.
3. **Whatever an adapter cannot promise stays out of the port.** Secret
   versioning stays out (Kubernetes has none). Firebase claims never cross the
   boundary; the identity port returns a normalised principal.

We also noticed that "port" hides two different things. Infrastructure ports —
secrets, identity, storage, the bus — are chosen once, at boot, by the
environment. Provider ports — git, task managers, later the model vendors —
are chosen on every request by the account's configuration, and several are
active at once. Confusing the two families is this design's typical mistake,
so the record names them separately.

## What it cost

Two adapters for every port, written and maintained, before any of them was
strictly needed. One more indirection on every infrastructure call. And a
vendor's strongest capability becomes inaccessible to the domain by
construction — deliberately.

## Since then

The rule was tested for real, and written up on 2026-09-04; it held in the
direction we had not planned for. `SecretStore` promises read-after-write. The promise was
born from the Kubernetes adapter, where it is trivially true. The Google
adapter could not keep it: Secret Manager is strongly consistent only when you
read a version *by number*, and `latest` converges "typically within minutes,
but may take a few hours". On real GCP a `Get` right after a `Put` could answer
"this does not exist" — for a credential just written — and the local emulator
would never show it.

Loosening the promise to "eventually consistent" was rejected: it pushes the
retry logic onto every caller, who cannot tell "not yet" from "never". Caching
the value in the process was rejected: a second place where a credential
lives. **The guarantee held and the adapter paid**: it confirms the write by
version, waits for the alias to catch up, and refuses loudly if it does not.
A `Put` that succeeds while the next `Get` says "not found" is worse than a
`Put` that fails — the first produces a silently broken integration, the
second an error somebody reads.

That episode was first written up as its own ADR and later folded into this
one: it is not a new decision, it is what this decision means when it hurts.
