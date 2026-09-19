---
title: "Who acts, and who is named on the commit"
translationKey: "decision-0003"
adr: "0003"
adr_title: "An organization credential to act, human authorship on the commit"
adr_file: "0003-organization-credential-human-authorship.md"
date: 2026-08-29
weight: 3
group: "resources"
description: "A person's token dies when the person leaves. An organization's credential erases who asked for the change. Two problems, and one choice for each — plus a third field git had all along."
related: ["0002", "0004", "0021"]
---

## What was on the table

A user's OAuth token belongs to the person. When they leave the company,
revoke access or change their password, the integration dies and takes with
it every project that depended on it. In a personal account that is fine —
the person *is* the account. In an organization it is a time bomb.

The obvious alternative creates the opposite problem: if the platform acts
with an organization's credential, the repository shows the App's
installation, and the history stops saying who asked for the change.
Traceability is exactly what a demand's dossier promises.

## The paths we weighed

**Everything in the organization's name.** Simpler and uniform. Rejected:
the repository stops recording who ran the work, and traceability stays
trapped inside the platform — useless to whoever reads the history months
later.

**The person's credential when they have one**, falling back to the
organization's. Perfect attribution. Rejected because it contradicts the rule
that a personal account's integration is never used inside an organization,
and it reintroduces exactly the fragility this decision exists to remove.

## What we chose, and why

Two independent choices, one per problem.

**To act**, an organization uses an organization credential — a GitHub App
installed on the organization, a group access token on GitLab, a service
principal on Azure DevOps. It survives departures. A personal token is
allowed as a way out, but the interface shows whom the integration depends
on, so the risk is visible instead of discovered on the day it breaks.

**To attribute**, the push and the pull request use the organization's
credential, but every commit carries an `author` with the name and e-mail of
the developer who ran the demand, and the PR's body names who asked.

And then git's third field. **The `committer` is the thread** — the agent
that made the commit, as a platform identity, never a person. Git has two
fields for two questions: `author` answers *for whom* the work was done, and
the customer's history keeps the person; `committer` answers *who wrote it
down*, and the platform's attribution keeps the agent. A commit the platform
makes on its own — a regenerated index, a rule saved from the cockpit —
carries the platform as committer and the person who acted as author.

## What it cost

A single point of failure: if the App is uninstalled or the token revoked,
every project on that account stops. That requires monitoring the
integration's status and alerting the owner. And the push's actor and the
commit's author are different entities, which can confuse whoever reads the
provider's interface without the context.

## Since then

When the project's knowledge became a git repository hosted by the platform
([ADR-0021](../project-knowledge-as-a-git-repository/)), the question "how is
a commit attributed there?" had an answer already: these rules govern every
repository the platform writes to, and that record explicitly declines to
restate them. One rule, one place.
