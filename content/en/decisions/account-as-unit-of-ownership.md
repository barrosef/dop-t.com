---
title: "The account owns everything"
translationKey: "decision-0002"
adr: "0002"
adr_title: "Tenancy: the account owns everything, and an organization proves itself by domain"
adr_file: "0002-account-as-unit-of-ownership.md"
date: 2026-08-29
weight: 2
group: "identity"
description: "One entity, personal or organization, owns every integration, workspace and project; every table carries its id from the first migration. An organization is created instantly and proves itself later, by a DNS record."
related: ["0009", "0019", "0020"]
---

## What was on the table

The platform is multi-tenant from the start: people with their own
authentication; organizations with members, roles and per-resource
access; both individuals and organizations owning integrations,
workspaces and projects. Two constraints shaped the model: a personal
account's integration must be private while an organization's is shared
under control — with no special case per situation — and creating an
organization must not require paperwork, while still letting it prove it
is the company it claims to be.

## The paths we weighed

**A polymorphic owner** — `owner_type` plus `owner_id` on every resource.
Rejected: two fields and a branch in every query, access rules written
twice, and a transfer that becomes a migration.

**Nestable namespaces** with inheritance at any depth. Rejected: the
hierarchy is fixed and three levels deep.

**Single-user first, tenancy later.** Rejected: retrofitting isolation is
the expensive migration.

**Everything built before returning to the product.** Rejected: it delays
the product that justifies the platform.

**Ownership validated by national ID or a power of attorney at creation.**
Rejected: friction, a registry integration, and no reference product does
it — GitHub and Google Cloud verify a domain, later.

**No verification at all.** Rejected: a shared handle namespace needs a
dispute path.

## What we chose, and why

An **`Account`**, personal or organization, is the single unit of
ownership. Everything owned carries an `account_id` and no other owner
field; a person is linked to an account by a `Membership` with a role; a
personal account is created with the user. A personal account's
integration is private because the account has one member; an
organization's is shared because it has several — no code needs to know
the difference.

The model is **complete from the first migration** — every entity carries
account, workspace and project; every call resolves an active account —
while phase one builds authentication, the personal account and the
workspace → project hierarchy. Organizations, members, roles and grants
arrive later without a migration.

An organization is **created instantly**: a name and a company
registration number, which autofills the legal name and address. It
**verifies its domain later**, optionally, by publishing a DNS TXT record.
Verification unlocks exactly three things: automatic entry for anyone
with an e-mail at that domain, the verified badge, and the right to
contest a handle someone else holds. Everything else works unverified.

## What it cost

An implicit personal account. A shared handle namespace, where
verification mitigates contention but does not remove it. A larger scope
before any product value — authentication, accounts, roles, grants.
Personal and company registration numbers in the system, with the data
protection obligations that follow. And a line worth keeping in view:
controlling a DNS zone proves control of DNS, not legal representation.

## Since then

The model was consolidated into one record on 2026-09-04. The invite and
the second factor both rest on the account being the boundary: it is what
a membership grants, and what a policy can require a second factor for.
