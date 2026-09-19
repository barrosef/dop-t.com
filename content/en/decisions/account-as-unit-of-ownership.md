---
title: "One owner, and a company proves itself by DNS"
translationKey: "decision-0002"
adr: "0002"
adr_title: "Tenancy: the account owns everything, and an organization proves itself by domain"
adr_file: "0002-account-as-unit-of-ownership.md"
date: 2026-08-29
weight: 2
group: "identity"
description: "Three questions arrived together — who owns things, when tenancy gets built, how an organization proves it is one — and answering them apart had produced three documents. One model answered all three."
related: ["0009", "0019", "0020"]
---

## What was on the table

Three questions arrived at once, and the first attempt answered them in three
separate records.

**Who owns things.** An individual owns integrations, workspaces and
projects; so does an organization. The requirements carried a tension: an
integration "can be seen and manipulated only by that user", yet a member of
an organization "has controlled access to all" of its integrations. We needed
a model in which both sentences are true at the same time, with no special
case per situation.

**When tenancy gets built.** The earlier documentation fixed the product as
single-user, many projects in parallel, no RBAC — one developer's local tool.
The direction had changed: users with their own authentication, organizations
with members, roles, per-integration access, running on a cluster or in the
cloud.

**How an organization proves it is one.** The original requirement asked, at
creation, to validate whether the signed-in person's national ID is the
company's owner or holds a power of attorney — citing GitHub and Google Cloud
as references for fluidity, with the instruction "do not invent, do not make
it hard". So we went to check what those references actually do. GitHub
creates an organization instantly, free, with no ownership check at all; the
verification comes later, is of the *domain* (a TXT record), and earns a
badge. Google Cloud requires a verified domain — also through DNS. Neither
asks for an ID or a power of attorney. The requirement's two halves pulled in
opposite directions.

## The paths we weighed

**A polymorphic owner** — `ownerType: user | org` plus an id on every
resource. It models the requirement's text literally. Rejected: every query
needs two fields and a branch, access rules get written twice, and moving a
resource from a person to an organization becomes a migration instead of an
update. GitLab had this model and migrated away from it.

**Nestable namespaces** — a generic tree with inheritance at any depth.
Rejected on YAGNI: the hierarchy asked for is fixed and three levels deep.

**Stay single-user, add tenancy later.** Faster to the product. Rejected:
retrofitting isolation is among the most expensive migrations there are —
every query written without an account filter is a potential leak, and the
cost grows with the code.

**Build all of it before returning to the product.** Rejected for the
opposite reason: it delays the product that justifies the platform.

**Validate ownership by national ID.** Rejected: it needs an integration with
a corporate registry, handles a power of attorney badly (a document a human
must read), and creates friction exactly where fluidity was asked for.

**Free creation with no verification at all.** Rejected because it leaves the
handle dispute unanswered — and the shared namespace makes that dispute
inevitable.

## What we chose, and why

An **`Account`**, personal or organization, is the single unit of ownership.
Everything owned carries an `accountId` and nothing else; a person is linked
to an account through a `Membership` with a role; a personal account is born
with the user. The requirements' tension dissolves by construction: a
personal account's integration is private because the account has one member;
an organization's is shared because it has several. No code needs to know the
difference.

The model is **born complete from the first migration** — every entity
carries account, workspace and project; every call resolves an active account
— while only authentication and the personal account are built now.
Organizations arrive later *with no migration*, because the schema already
expected them. And phase one exercises tenancy for real: every query filters
by account from day one; the account simply is always personal.

An organization is **created instantly** — a name and a registration number
that autofills the legal name — and **verifies its domain later**, optionally,
by publishing a TXT record. Verification unlocks deliberately few things:
automatic entry for anyone with an `@domain` e-mail, the verified badge, and
the right to contest a handle somebody else took. Everything else works
without it.

## What it cost

An implicit personal account the person never asked for. A shared handle
namespace: if somebody takes `acme` as a personal account, the Acme
organization cannot — mitigated by verification, not removed. A larger scope
before any product value: authentication, accounts, roles, grants. Personal
and company registration numbers enter the system, with the data-protection
obligations that implies. And a line we wrote down so nobody mistakes it:
controlling a DNS zone does not prove legal representation. If a contractual
obligation ever needs that, it is another decision, in another layer.

## Since then

On 2026-09-04 the three records were folded into one — the two absorbed
numbers were retired, and on 2026-09-17 the whole sequence was renumbered so
the gaps would not read as mistakes. The invite ([ADR-0019](../invite-without-token/))
and the second factor ([ADR-0020](../second-factor-in-the-core/)) both lean
on the account being the boundary: it is what a membership grants, and what a
policy can require a second factor for.
