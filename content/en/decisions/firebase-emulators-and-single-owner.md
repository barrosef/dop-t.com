---
title: "The emulator that must not lie"
translationKey: "decision-0015"
adr: "0015"
adr_title: "Firebase emulators in the local environment; Terraform as the single owner"
adr_file: "0015-firebase-emulators-and-single-owner.md"
date: 2026-08-30
weight: 15
group: "foundations"
description: "Two lessons paid for by a sibling project, written down here before we paid for them again: the local environment runs the same SDK and the same configuration as production, and nothing is created through a console."
related: ["0001"]
---

## What was on the table

The local environment needed identity and object storage. The first proposal
was the Firebase emulator for authentication and MinIO for objects. That is
two different clients — a local S3 against GCS in production — two semantics
for signed URLs, and the oldest failure in the book: it works on my machine,
it breaks in the cloud.

A sibling project had already paid for two lessons in this area, and we did
not want to buy them a second time.

## What we chose, and why

**Firebase's Emulator Suite covers both**, authentication and storage, with
the same SDK as production, resolved by an environment variable. MinIO does
not come in; it stays as a third adapter of the object-store port for the day
a self-hosted customer has no Google Cloud.

**The emulator keeps its data across restarts.** An emulator that forgets
everything on every stop pushes developers back to the cloud for anything
that takes more than one sitting. The mechanics — export on exit, conditional
import, a grace period — are operational and live in the infrastructure spec;
what is decided here is that the local environment *must* survive a restart.

**The emulator's configuration is the deploy's configuration.** `firebase.json`,
`.firebaserc` and the rules are versioned and mounted read-only into the
emulator — the same files the deploy uses. An emulator with its own
configuration lies about production.

**Terraform is the single owner of what it manages.** This was the second
lesson from the sibling project: a resource created through the console or
the CLI is not in the state, and the next `apply` reverts or deletes it — a
feature lost in silence. So nothing is created through the console; what the
Firebase CLI publishes lives in versioned files Terraform references or
imports; and where the boundary is ambiguous — authentication providers, for
example — the infrastructure README declares one owner per resource, in an
explicit table.

## What it cost

A dependency on the Firebase CLI in every development environment. And the
ownership table has to be maintained by hand; it is what prevents the silent
loss on `apply`, and it is only as good as its last edit.

## Since then

The decision was slimmed on 2026-09-04: the operational recipe moved out of
the record and into the infrastructure spec, leaving only what is a decision.
The rule about the console was tested a week later, when the QA environment
on Google Cloud was codified in Terraform from the resources that had been
created by hand during the first deploys — every import had to produce a plan
with no changes before the environment counted as owned.
