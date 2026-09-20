---
title: "Architecture"
translationKey: "architecture"
seo_title: "DOP architecture — a core in Go, a BFF in Python, a React cockpit, events and a sandbox per demand"
layout: "architecture"
description: "How the DOP platform is built: the components, the five invariants, the life of an event and the decision record."
components_title: "The components"
components:
  - name: "dop-core"
    stack: "Go"
    repo: "dop-core"
    role: "The core: domain, state, transactions and the event log."
    text: "One binary, four modes (serve, worker, sched, launcher). It owns the schema and the vault; it is the only source of truth, and it verifies every caller's signature."
  - name: "dop-api"
    stack: "Python"
    repo: "dop-api"
    role: "The BFF: REST and SSE for the cockpit, gRPC for the CLI."
    text: "It has no database and no secret. It translates and forwards; every write goes through the core."
  - name: "dop-app"
    stack: "React · Vite"
    repo: "dop-app"
    role: "The cockpit where the developer works."
    text: "The attention box, the tree of workspaces and projects, the demand's cockpit. It consumes a committed contract and generates its hooks and schemas from it."
  - name: "dop-infra"
    stack: "Terraform · k3d"
    repo: "dop-infra"
    role: "The platform's infrastructure."
    text: "Terraform for Google Cloud (Cloud Run, Secret Manager, Identity Platform) and a local k3d environment with Postgres, NATS JetStream and the Firebase emulators."
  - name: "sandbox"
    stack: "microVM"
    repo: ""
    role: "One per demand, with a single shared worktree."
    text: "The hard boundary is between accounts and between demands. The project's knowledge is cloned inside as a git repository; verification runs from source in an ephemeral runner."
diagrams:
  title: "The diagrams"
  lead: "Six pictures, from the whole platform down to each component. Every name in them is a package, a port or an adapter that exists in the repositories; what is planned and not built is drawn dashed."
  items:
    - id: architecture
      title: "The platform"
      caption: "The cockpit and the CLI reach the BFF; the BFF reaches the core; the core owns the state, the events and the sandboxes. Identity Platform signs the person in at the edge."
    - id: core
      title: "The core, inside — the hexagon"
      caption: "One binary, four modes, one domain. The domain never imports a driver: it declares a port, and the adapter on the right implements it. Every port has at least two adapters and one contract suite that both must pass — which is what makes the local environment and Google Cloud the same platform."
    - id: bff
      title: "The BFF, inside — two transports, one rule"
      caption: "REST with SSE for the cockpit, gRPC for the CLI, and the same use case underneath. Authorization is pinned to the use case, not to the router, so a rule cannot exist on one transport and not the other; a parity test keeps it that way."
    - id: events
      title: "Events and the flow engine"
      caption: "The pipeline: a write commits its event in the same transaction; a relay publishes it; every consumer receives it. The two blocks marked with a plus are sub-processes drawn in the next two figures — the flow engine, which decides what an event triggers, and the failure path, which covers every consumer's delivery, not one of them."
    - id: flow-engine
      title: "The flow engine"
      caption: "A flow is data: a versioned list of typed stages, each with a gate, its artifacts and its actions on enter and exit. The effective flow comes down a chain — platform, account, workspace, project, demand — where the nearest declared level wins, and a demand freezes the version it started with. When a stage advances, the stage's actions and the accumulated rules table are decided into planned actions; the executor runs them through a registry of four and records each as applied per event, rule and action, so a retry never repeats one."
    - id: resilience
      title: "The failure path"
      caption: "The same path covers every consumer. A failed delivery is redelivered with backoff up to five times; an exhausted one becomes a dead letter with the whole envelope and its attempts, published to its own subject. The dead-letter consumer re-runs the same handler three more times, then terminates the message — but every attempt, its classification and the last success are kept in the error ledger, and a signature that exhausts twice is learned as irrecoverable until a success demotes it."
    - id: app
      title: "The cockpit, inside"
      caption: "A router, ten pages, six component families, two hooks, and a generated client underneath. The contract is a committed OpenAPI file; Orval produces the react-query hooks and the Zod schemas from it; one fetch layer carries the person's token and the account on every call. Firebase Auth issues the token; the BFF answers REST and pushes the attention stream over SSE. Nothing the backend already decided is decided again here."
    - id: callauth
      title: "How the core verifies its callers"
      caption: "Every call to the core carries a signature the core can check: the person's token, forwarded whole by the BFF, or a platform assertion signed with the caller's own key when there is no person. The interceptor resolves the actor from the token and the account from the assertion, refuses when the two disagree, and lets authorization say no with a message that means something. Cloud Run IAM in the cloud and a network policy on a cluster restrict who can reach the service at all; neither replaces the signature."
    - id: external
      title: "The platform and what it talks to"
      caption: "Every external service sits behind a port, so swapping one is an adapter and a configuration value, not a rewrite. What is solid has an adapter today; what is dashed is on the roadmap."
invariants_title: "The five invariants"
invariants_lead: "The architecture's load-bearing walls. A change that breaks one is wrong even when it compiles and the tests pass."
invariants:
  - title: "The core is the only source of truth."
    text: "It owns the schema and the vault. Every write goes through it."
  - title: "The BFF has no database and no secret."
    text: "It translates and forwards; it never persists and never holds a credential."
  - title: "The .proto files are the contract."
    text: "Generated code is regenerated, never hand-edited. An incompatible change is refused on purpose."
  - title: "The cockpit consumes a committed contract."
    text: "The OpenAPI spec is downloaded and committed; the react-query hooks and the Zod schemas are generated from it."
  - title: "The core verifies its callers, and verification runs from source."
    text: "A signature on every call, never a trusted header. A verification run builds from the repository in a runner, not from an image."
event:
  title: "The life of an event"
  lead: "Every write in the core emits an event in the same transaction. What happens next is infrastructure, not the repository's problem."
  steps:
    - name: "write"
      text: "A transaction in Postgres changes state and appends the event to the outbox — one commit, or neither."
    - name: "outbox"
      text: "A relay reads the outbox and publishes to NATS JetStream. The message id is the event's id, so a retry cannot duplicate it."
    - name: "envelope"
      text: "The event travels with its context: aggregate key, actor, request id, session id, caller."
    - name: "consumer"
      text: "A consumer processes it. On failure it is redelivered with backoff, up to five times."
    - name: "dead letter"
      text: "Exhausted, the event goes to one dead-letter queue with the attempt history and its classification: recoverable, irrecoverable, unknown."
      dlq: true
    - name: "error ledger"
      text: "Every failure is recorded per (event, consumer); a signature that burns every retry twice is learned as irrecoverable, and a success demotes it."
adrs_title: "The decision record"
adrs_lead: "Every structuring decision, in the MADR format: context, decision, alternatives considered, consequences. One subject, one ADR."
---

DOP is three repositories that ship and one that pins them. The components are
independent git repositories, aggregated as submodules by the
[umbrella repository](https://github.com/barrosef/dop), which carries the
product's documentation: the PRD, the ADRs, the specs and the API collections.
