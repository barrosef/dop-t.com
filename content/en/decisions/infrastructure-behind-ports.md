---
title: "Every port has two adapters and one contract"
translationKey: "decision-0001"
adr: "0001"
adr_title: "Infrastructure behind ports with pluggable adapters"
adr_file: "0001-infrastructure-behind-ports.md"
date: 2026-08-29
weight: 1
group: "foundations"
description: "The domain never touches a vendor. Each piece of infrastructure is a port the domain defines, with at least two adapters and a contract suite both must pass — which is what lets the same binary run on Google Cloud and on a laptop cluster."
related: ["0012", "0016", "0020"]
---

## What was on the table

The platform runs in two kinds of place from the start: on Google Cloud
Run, and on a Kubernetes cluster somebody else operates — k3s, Rancher,
OKD. Each place offers a different service for the same need: Secret
Manager or a Kubernetes Secret; Cloud Storage or a filesystem; Identity
Platform or an OIDC provider. The domain could not be written against any
of them.

## The paths we weighed

**Couple to one cloud now, abstract later.** Faster to a first result;
rejected because running on a laptop cluster is a development requirement,
not a future ambition.

**A generic multi-cloud library.** Rejected: it abstracts the library's
vendors, not the domain, and trades one coupling for another.

**Ports and adapters, with disciplines attached.** The choice — with the
rules that keep "hexagonal" from being just the name of a folder.

## What we chose, and why

The domain reaches infrastructure only through a port it defines, in its
own vocabulary — `SecretStore.Get(ref)`, not `AccessSecretVersion`. Adapters
are bound in one composition root and selected by configuration; no
conditional on the environment exists anywhere else.

Three rules make it real:

1. **Two adapters per port, from day one** — a production one and a local
   one. The local adapter is the proof that the port is shaped by the
   domain and not by the first vendor.
2. **One contract test suite per port**, which every adapter passes.
3. **A port carries only what every adapter can guarantee.** Secret
   versions stay out; provider-specific token claims stay out; the identity
   port returns a normalised principal.

Ports come in two families, and the difference matters for the wiring:
infrastructure ports — secrets, identity, storage, the event bus — are
chosen once, at boot, by the deployment; provider ports — git hosts, task
managers, model vendors — are chosen per request by the account's
configuration, with several active at once.

And one rule for the day an adapter cannot meet a guarantee natively: **the
adapter pays; the port's guarantee is not lowered.** The secret store
promises read-after-write. Google's Secret Manager only guarantees that
when reading a version by number, so its adapter confirms the write by
version, waits for the `latest` alias to converge, and refuses with an
explicit error if it does not — rather than answering "not found" for a
credential that was just written.

## What it cost

Two adapters for every port, written and maintained before either is
strictly needed. One more indirection on every infrastructure call. A
vendor's strongest capability is unreachable from the domain by
construction. And a write to Secret Manager is slower than a write to a
Kubernetes Secret, with a failure mode the local emulator does not
reproduce.

## Since then

The read-after-write rule for Secret Manager was added on 2026-09-04. Every
later port — the agent provider, the SMS channel, the project repository,
the verification runner — was born under these three rules.
