---
title: "Three documents disagreed, and the cheap path was to build from source"
translationKey: "decision-0023"
adr: "0023"
adr_title: "Verification builds from source in a runner, not from an image"
adr_file: "0023-verification-runs-from-source.md"
date: 2026-09-04
weight: 23
group: "delivery"
description: "A pod runs an image, and an image needs a builder and a registry on the critical path of every run. Cutting the three hops around the registry leaves what every CI runner in the world does."
related: ["0001", "0003", "0005", "0017"]
---

## What was on the table

Three documents disagreed about where a demand's application runs, and the
disagreement was ours to fix. The execution spec said an internal Docker
brings up the stack inside the sandbox. The sandbox decision said a
verification runs in an ephemeral pod, because a test inside the sandbox
runs against a dirty tree. And the write-up of a later owner decision had
put the run back in the sandbox — a sentence the owner never said.

Resolving it exposed the real cost of "a pod in the cluster". A pod runs an
*image*, so verification would need: reading the project's compose file,
translating it into manifests, building the application's image from the
commit, pushing it to a registry, and pulling it back on a node. A builder
and a registry that did not exist, on the critical path of every run.

## The paths we weighed

**Compose inside the sandbox** — the original spec. Cheapest to build.
Rejected as the verification path: the environment is the agent's, with
whatever it installed along the way, and evidence from there speaks about
that environment, not a clean one. It remains the candidate for the
developer's own bench.

**An ephemeral pod from a built image** — the sandbox decision's mechanism.
Honest environment; costs a builder, a registry and three network hops per
run. Rejected for the cost, not for the argument.

**Translating compose into manifests.** Works for the simple case and lies
for the rest: `build`, `healthcheck`, `depends_on`, volumes and profiles have
no clean equivalent, and the developer ends up debugging a manifest they
never wrote.

**A runner kept warm.** Faster to start, and it burns money while nothing
happens. Revisit with a measured latency.

## What we chose, and why

**The verification environment is a runner**: ephemeral, it pulls the
commit, builds from source and starts the application. No image of the
project is ever built, pushed or deployed. The slow sequence was never the
build — it was build, push, pull, start, three hops around a registry that
only existed to move bytes between two places in the same cluster. Cutting
it leaves pull, build, start: what every CI runner does, and what a
developer does on their own machine. It also removes the compose question
entirely: there is no stack description to read, only a repository, a
commit, and a command.

**The runner's image is ours, built once** — the toolchains live in an
image we publish and the node caches. A fat image, deliberately: one big
image cached everywhere beats a small one built per demand. **Third-party
dependencies are pulled, never built** — a database is a published image.
**The account's cache volume is mounted into the runner**, so the first run
of a project pays and the rest do not; without that, the decision does not
hold. **It is separate from the sandbox**, on purpose: the evidence is
honest about the environment, the test does not compete with the agent for
resources, and a dead runner takes nothing of the demand with it. It takes
the demand's address while it runs — one per demand, parallel runs queue —
and reconciling that address when a run dies is part of its lifecycle. And
"run this commit and give me a URL" is a port, with Kubernetes and Docker
adapters, so the two executors cannot diverge on the very thing that
produces evidence.

## What it cost

A runner image that is a product artefact of ours — versions, size, a
release cadence — and the version drift of our customers' toolchains as a
maintenance burden we took on rather than pushed onto them. The project
declares its dependencies in a few lines: a tax, the smallest available.

## Since then

The next day closed the question this decision opened: **the demand keeps
no running application.** It exists only during a verification, or while a
developer asked to look at it — the same runner, no checks, held to a
deadline. The bench is where code is written, not where it runs. The cheaper
and poorer option, chosen knowingly. Then two triggers were fixed: the end
of development, automatically, once the reaction-as-data process exists to
say so; and the developer clicking *test* and keeping the preview — which
turned "a run with no checks holds" into "a run holds when it was asked to".
The runner's port has a shape and nine guarantees in a spec; it is still not
built.
