---
title: "DOP"
translationKey: "home"
eyebrow: "delivery orchestration · built in the open"
headline: "The developer and the agent build together. From the workspace to the delivered PR."
lead: "A cockpit where every demand becomes a sequence of events: the agent operates underneath, the developer follows and decides from above. Nothing happens outside the record."
why_title: "Four things that are true of DOP and of very few other tools."
values:
  - key: operators
    title: "Two operators, one cockpit"
    text: "The agent runs the demand; the developer approves the spec, gives context and decides. The division of labour is in the flow, not in a prompt."
  - key: events
    title: "Everything is an event"
    text: "Every write emits an event that carries its context — actor, request, session. A failure goes to a dead-letter queue with everything needed to replay it, not into a log nobody reads."
  - key: sandbox
    title: "A sandbox per demand"
    text: "Isolation between accounts and between demands, with the project's knowledge mounted inside as a git repository."
  - key: open
    title: "Built in the open"
    text: "Every structuring decision is an ADR in a public repository, and the code that implements it is next to it."
how:
  title: "One core that owns the truth. Everything else translates."
  points:
    - title: "A core in Go is the only source of truth."
      text: "It owns the schema, the vault and the event log. Every write goes through it, and it verifies who is calling."
    - title: "A BFF in Python holds no database and no secret."
      text: "It translates REST and SSE for the cockpit into gRPC for the core, and forwards. Nothing persists at the edge."
    - title: "The cockpit consumes a committed contract."
      text: "The .proto files are the contract; the cockpit's hooks and schemas are generated from a downloaded OpenAPI spec, never written by hand."
stands_title: "Pre-release, and honest about it."
closing: "The next demand you open could be the one the agent finishes."
---
