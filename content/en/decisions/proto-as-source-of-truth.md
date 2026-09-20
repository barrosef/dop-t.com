---
title: "One contract, generated everywhere"
translationKey: "decision-0013"
adr: "0013"
adr_title: "The .proto is the contract's source of truth"
adr_file: "0013-proto-as-source-of-truth.md"
date: 2026-08-30
weight: 13
group: "foundations"
description: "One .proto per domain is the only description of the API. The core's server, the edge's client, the CLI and the cockpit's hooks are generated from it, and a breaking change fails the build instead of surfacing in production."
related: ["0012", "0022"]
---

## What was on the table

Four components — the core, the edge, the CLI and the cockpit — share one
API. Described more than once, a contract drifts: a field renamed on one
side, a type widened on another, and the mismatch shows up at run time. The
contract needed exactly one source.

## The paths we weighed

**OpenAPI as the source, gRPC generated from it.** Rejected: the core's
contract would depend on the edge's format.

**A hand-written contract at each end.** Rejected: that is drift by design.

## What we chose, and why

**One `.proto` per domain, versioned, is the only source.** From it are
generated the core's server and types in Go, the edge's client in Python,
the CLI's client, and — through the OpenAPI document the edge publishes —
the cockpit's react-query hooks and Zod schemas. Generated code is never
edited by hand; a breaking change is refused in CI.

Five conventions travel with it:

- everything live is server-side streaming, which the edge turns into SSE
  for the browser;
- identifiers are typed references (`AccountRef{id}`), never loose strings;
- every write carries an `idempotency_key`, scoped to the account that owns
  the rows — platform-level rows use a namespace of their own;
- **who is calling, in which account, travels in the call metadata**, never
  in the message body — resolved by an interceptor before any use case
  runs, and verified per [the caller decision](../the-core-verifies-its-callers/);
- a field never changes its number or its type.

## What it cost

`buf` and code generation are in CI from day one, and a contract change is
a versioned, additive change. Four repositories regenerate when the
contract moves.

## Since then

Two conventions were made precise after the first implementation: on
2026-09-02 the request messages lost a context field that the metadata had
already replaced (its number stays reserved), and on 2026-09-06 the scope of
the idempotency key was fixed to the owning account.
