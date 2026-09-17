---
title: "Status"
translationKey: "status"
layout: "status"
description: "How far along the DOP platform is: the subprojects and their state, what shipped and when, what comes next."
subprojects_title: "The subprojects"
subprojects_lead: "The platform was decomposed into seven subprojects, decided in this order because each one operates inside the previous. Five are designed; two are open."
shipped_title: "Shipped"
shipped:
  - date: "2026-09-13"
    title: "Events with context, and the dead-letter queue that was missing"
    text: "Every event travels with its aggregate key, actor, request and session. An exhausted event goes to one dead-letter queue with its attempt history; failures are classified and learned."
    ref: "ADR-0014 · dop-core"
  - date: "2026-09-09"
    title: "The QA environment on Google Cloud, under the product's own domains"
    text: "Codified in Terraform, with the core and the BFF on Cloud Run, Postgres and NATS on a VM, and api.qa.dop-t.com and auth.qa.dop-t.com verified. The account chain runs end to end: sign-in, the BFF, the core, the database."
    ref: "dop-infra"
  - date: "2026-09-03"
    title: "The project's knowledge is a git repository, hosted by the platform"
    text: "A git server with per-demand tokens; the sandbox clones the project at /project read-write; every artifact is committed and every push is an event."
    ref: "ADR-0021"
  - date: "2026-09-03"
    title: "The core verifies its callers"
    text: "A signature on every call — the person's token where there is a person, a platform assertion where there is none — with the wire format pinned by the same test vector in Go and Python."
    ref: "ADR-0022"
  - date: "2026-09-02"
    title: "The second factor, end to end"
    text: "TOTP written in the standard library, e-mail and SMS verifiers behind ports, a step-up gate on four sensitive operations, and the cockpit's screens."
    ref: "ADR-0020"
next_title: "Next"
next:
  - title: "The agent's tools and the cockpit"
    text: "The piece missing for the platform to execute instead of only modelling. Provider focus: Anthropic; the multi-provider port stays."
  - title: "Hosted Claude Code as the laboratory"
    text: "The sandbox runs the unmodified binary and the developer signs in with their own subscription — the place where measuring a demand's real cost costs nothing."
  - title: "The user stories"
    text: "Only after the structure is finished. Reaction to an event as data (P-29) comes first, because writing the stories against a switch statement would mean rewriting them."
---

The platform is built in the open. This page is a summary of the
[roadmap](https://github.com/barrosef/dop/blob/main/docs/ROADMAP.md) in the
umbrella repository, which is the source of every state and date below.
