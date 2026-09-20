---
title: "The local environment runs production's SDK, and Terraform owns the rest"
translationKey: "decision-0015"
adr: "0015"
adr_title: "Firebase emulators in the local environment; Terraform as the single owner"
adr_file: "0015-firebase-emulators-and-single-owner.md"
date: 2026-08-30
weight: 15
group: "foundations"
description: "Firebase's emulators provide identity and object storage locally with the same SDK and the same configuration files as production. Infrastructure is created by Terraform only, so nothing exists that the state does not know about."
related: ["0001"]
---

## What was on the table

The local environment needs identity and object storage with the same
semantics as production, so that what works on a laptop works in the
cloud. And the infrastructure must not drift between what Terraform manages
and what actually exists.

## The paths we weighed

**The Firebase Auth emulator plus MinIO for objects.** Rejected: two
different clients — a local S3 against GCS in production — and two
semantics for signed URLs.

**The Firebase Emulator Suite for both.** The choice.

## What we chose, and why

**Firebase's Emulator Suite provides authentication and storage locally**,
with the production SDK selected by an environment variable. MinIO is not
used; it remains a possible third adapter of the object-store port for a
self-hosted customer without Google Cloud.

**The emulator persists across restarts** — export on exit, conditional
import, a grace period — so a developer's data survives a session.

**The emulator's configuration is the deploy's configuration:**
`firebase.json`, `.firebaserc` and the rules are versioned and mounted
read-only into the emulator. There is no second configuration that could
disagree with production.

**Terraform is the single owner of what it manages.** Nothing is created
through the console; what the Firebase CLI publishes lives in versioned
files Terraform references or imports; where ownership is ambiguous, the
infrastructure README declares one owner per resource. A resource adopted
from outside is imported until `terraform plan` reports no changes.

## What it cost

The Firebase CLI becomes a development dependency, and the ownership table
is maintained by hand.

## Since then

The operational recipe moved out of the record into the infrastructure spec
on 2026-09-04, leaving the decision alone. The single-owner rule was applied
to the QA environment on Google Cloud in September, when every resource was
imported into Terraform state.
