---
title: "Verification runs in a runner that builds from source"
translationKey: "decision-0023"
adr: "0023"
adr_title: "Verification builds from source in a runner, not from an image"
adr_file: "0023-verification-runs-from-source.md"
date: 2026-09-04
weight: 23
group: "delivery"
description: "An ephemeral runner pulls the commit, builds the application from source and starts it — no image of the project is ever built, pushed or deployed. The account's cache makes the second run fast; the runner takes the demand's address while it runs."
related: ["0001", "0003", "0005", "0017"]
---

## What was on the table

Evidence of a green verification must name the commit it ran on and the
environment it ran in. Running a pod from an image would require, on
every run's critical path, translating the project's compose file into
manifests, building the application's image, pushing it to a registry and
pulling it back — a builder and a registry that exist only to move bytes
between two places in the same cluster.

## The paths we weighed

**Compose inside the sandbox.** Rejected for verification: the agent's
environment is not a clean one. It remains a candidate for a developer's
own bench.

**An ephemeral pod from a built image.** Rejected for its cost — builder,
registry, three network hops per run — not for its argument, which this
decision keeps.

**Translating compose into manifests.** Rejected: no faithful mapping for
`build`, `healthcheck`, `depends_on`, volumes and profiles.

**A runner kept warm.** Rejected until start latency is measured.

## What we chose, and why

**The verification environment is a runner:** ephemeral, it pulls the
commit, builds from source and starts the application — what every CI
runner does. **The runner's image is the platform's**, published once with
the toolchains and cached on the nodes. **Third-party dependencies are
pulled as published images**, declared by the project in a few lines;
nothing of a third party is built. **The account's cache volume is
mounted into the runner** — `node_modules`, the Go build cache, Maven —
so the first run of a project pays and the rest do not.

**The runner is separate from the sandbox**, created when a run starts and
destroyed when it ends, never kept idle. It holds the demand's address
while it runs; parallel runs of one demand queue; the address is
reconciled if a run dies. "Run this commit and give me a URL" is a port,
`VerificationRunner`, with Kubernetes and Docker adapters.

**Two triggers:** the end of development, automatically, once the
reaction-as-data process exists to decide it; and the developer asking —
in which case the run holds the environment after the checks for the
developer to use. **The demand keeps no running application** outside a
verification or a held run.

## What it cost

A runner image that is a product artifact with a release cadence, and the
version drift of customers' toolchains as a maintenance burden taken on
rather than pushed onto them. Projects declare their dependencies in a
few lines.

## Since then

The two triggers and "no running application" were added on 2026-09-04
and 2026-09-05. The port has its guarantees in a spec; the runner is not
built yet.
