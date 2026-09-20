---
title: "The organization acts; the person is the author; the agent is the committer"
translationKey: "decision-0003"
adr: "0003"
adr_title: "An organization credential to act, human authorship on the commit"
adr_file: "0003-organization-credential-human-authorship.md"
date: 2026-08-29
weight: 3
group: "resources"
description: "Pushes and pull requests use the organization's credential, so nothing breaks when a person leaves. Every commit names the developer as author and the agent thread as committer, so the history still says who asked and who wrote."
related: ["0002", "0004", "0021"]
---

## What was on the table

Two questions that pull in opposite directions. A person's OAuth token
belongs to the person and dies with their access — in an organization,
every project depending on it would stop. An organization credential
survives departures but, used alone, erases from the repository's history
who requested the change.

## The paths we weighed

**Everything in the organization's name.** Rejected: no human trail in the
repository; traceability trapped inside the platform.

**The person's credential when available, the organization's otherwise.**
Rejected: it keeps the dependency on a person's token and contradicts the
rule that a personal account's integration is never used in an
organization.

## What we chose, and why

Two independent choices, one per question — and a third field git already
had.

**To act**, an organization uses an organization credential: a GitHub App
installation, a GitLab group access token, an Azure DevOps service
principal. A personal token is permitted as a fallback, and the interface
shows whom the integration depends on.

**To attribute**, every commit's `author` is the developer who ran the
demand, and the pull request's body names who requested it.

**And the `committer` is the thread** — the agent that made the commit, as
a platform identity, never a person. Git's two fields answer two
questions: for whom the work was done, and who wrote it down. A commit the
platform makes on its own carries the platform as committer and the acting
person as author.

These rules govern every repository the platform writes to — the
customer's code repositories and the project's knowledge repository alike —
and are stated in one place.

## What it cost

A single point of failure per organization: a revoked App or token stops
every project of that account, so the integration's status is monitored
and the owner alerted. And the push actor and the commit author are
different identities, which can surprise whoever reads the provider's
interface without the context.

## Since then

When the project's knowledge became a git repository hosted by the
platform, its commits followed these rules unchanged.
