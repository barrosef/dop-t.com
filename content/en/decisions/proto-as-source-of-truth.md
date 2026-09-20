---
title: "A contract that was born tripled"
translationKey: "decision-0013"
adr: "0013"
adr_title: "The .proto is the contract's source of truth"
adr_file: "0013-proto-as-source-of-truth.md"
date: 2026-08-30
weight: 13
group: "foundations"
description: "Three descriptions of the same API, none of them authoritative, drifting before there was a product. One file became the truth — and a field in it that did nothing taught us something."
related: ["0012", "0022"]
---

## What was on the table

The contract already existed in three places before any of them worked: a
`types.ts` written by hand in the frontend, an empty `openapi.yaml` with code
generation running over nothing, and the promise of a `.proto` for the core.
Three sources, none of them the authority. The drift had started before there
was a product to drift from.

## The paths we weighed

**OpenAPI as the source, gRPC generated from it.** It inverts the dependency:
the internal contract — the core's — would depend on the edge's format.
Rejected.

**Contracts written by hand at both ends.** That was the current state, and
it was the problem.

## What we chose, and why

One `.proto` per domain, versioned, is the only source. From it we generate
the core's server and types in Go, the edge's client in Python, the CLI's
client, and the frontend's types. The edge's OpenAPI document is itself a
product of the edge, which is a product of the proto.

Five conventions came with it, and two of them matter more than the others.
Everything live is server-side streaming — the edge turns it into SSE for the
browser. Every write carries an `idempotency_key`, because with events and
retries that is a requirement, not a luxury. And **who is calling, in which
account, travels in the metadata, not in the body** — resolved by an
interceptor before any use case runs. Context is cross-cutting: in the body,
every RPC would have to remember to check it, and the one that forgot would be
an isolation hole. Compatibility is checked in CI: a field never changes its
number or its type.

## What it cost

`buf` and code generation enter CI on day one. A contract change requires
discipline — additive by default — and a build now fails where a drift would
once have been a discovery in production.

## Since then

Two things happened that the original text could not have predicted, and both
are now part of the decision.

The first was a debt. The request messages used to carry a `CallContext ctx =
1` from an earlier attempt, and the server ignored it. A contract that declares
a field with no effect teaches the wrong thing: the client believes it is
scoping the call when it is not. It had already copied itself into every new
proto, the second factor's included. On 2026-09-02 the field left all twelve
protos, number 1 was reserved in every message that carried it, the message
type disappeared from `common.proto`, and two tests that used to assert the
field was in the body now assert the opposite — which is what keeps it from
coming back.

The second was a subtle one about the idempotency key's *namespace*. One table
made the key unique across all rows; another scoped it to the account. The key
is supplied by the client and stored verbatim, so a table-wide namespace lets
account B send a key account A already used, collide with a row B may not see,
and get that collision handed back. One table shipped that once. The rule that
came out of it: scope the key to whatever owns the rows, and where a level has
no owner, give it a namespace of its own — a decision the next table's author
should take deliberately, not by copying whichever neighbour they opened first.
