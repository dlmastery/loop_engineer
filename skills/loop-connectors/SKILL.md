---
name: loop-connectors
description: The reach of a loop — connectors and plugins (built on MCP) that let the loop act in the tools you already use instead of only seeing the filesystem. Use when a loop needs to read/write an issue tracker, query a database, hit a staging API, post to Slack, open PRs, or bundle/share a setup as a plugin. Covers when to add a connector, treating each as a failure surface, idempotency, and least-privilege.
---

# Loop Connectors (the reach)

A loop that can only see the filesystem is a tiny loop. Connectors — built on
MCP — let the agent read your issue tracker, query a database, hit a staging
API, drop a message in Slack. Because both Claude Code and Codex speak MCP, a
connector written for one usually works in the other. Plugins bundle connectors
and skills so a teammate installs your whole setup in one go.

This is the difference between an agent that says "here is the fix" and a loop
that opens the PR, links the ticket, and pings the channel once CI is green —
by itself.

## Add a connector only when the loop must *act*

For each connector ask: does the loop need to *do* something in that system, or
just be told about it? Don't wire a connector the loop won't use — every one is
attack surface and a failure mode. Common ones:

| Need | Connector |
|------|-----------|
| Open/label/update issues, open PRs | GitHub / GitLab |
| Track work items humans also see | Linear / Jira (doubles as `loop-memory`) |
| Notify / escalate | Slack |
| Validate against real data | DB (read-only role!), staging API |

## Connectors are the loop's most common leak

This is where unattended loops fail quietly. Engineer for it:

- **Auth expires.** Tokens lapse mid-run. Detect auth failure explicitly and
  send the item to triage — never let an expired token look like "no work to do."
- **Rate limits & partial writes.** A "create issue + comment + label" sequence
  can half-succeed. Make every external write **idempotent**: use a stable key
  (e.g. the state-file item id) so a retry updates rather than duplicates, and
  check-before-create.
- **Log every external action.** The loop acts when you're not watching; the log
  is how you reconstruct what it did. Record each write in the state-file budget
  ledger (`loop-memory`).
- **Fail closed.** If a connector is down, the loop should record and stop the
  affected item, not invent a fallback that ships something wrong.

## Least privilege

- Scope each connector to the minimum: a DB connector gets a **read-only** role;
  a GitHub token gets only the repos and scopes it needs.
- Secrets go in env / a secret store, **never** in skills, agent files, or the
  state file (`harness-guardrails`).
- Prefer a bot/service identity over a personal token so the loop's actions are
  attributable and revocable.

## Packaging

When a loop's connectors + skills should be reusable across repos or shared with
a team, bundle them as a **plugin** (this repo is itself a plugin — see
`.claude-plugin/plugin.json`). The skill is the authoring format; the plugin is
how you ship it.
