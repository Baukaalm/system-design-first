# System Design First

**Make your coding agent think like a senior systems engineer before it writes a line of code.**

[![skills.sh](https://skills.sh/b/Baukaalm/system-design-first)](https://skills.sh/Baukaalm/system-design-first)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-blue)](https://agentskills.io)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

One skill, zero configuration. Install it and every feature request gets the
treatment it would get from a strong systems architect:
real numbers, the design as it exists in your code, the design as it should
be, and a one-page brief that explains *why* before the agent builds it.
Works with Claude Code, Codex, Cursor, Copilot, Gemini CLI and any agent that
reads the open [Agent Skills](https://agentskills.io) format.

```sh
npx skills add Baukaalm/system-design-first
```

## Why

Agents are great at building what the ticket says. They are bad at noticing
what the ticket forgot to say.

> **"Add CSV export of the report."**
> The agent builds the whole file in request memory and returns it
> synchronously. Works on the test tenant. The largest tenant has 250k rows,
> the proxy times out at 30 s, two double-clicks start four exports.

Everything that went wrong was knowable in five minutes from the row counts
and the proxy config. Nobody asked, so nobody knew.

The opposite failure is just as common: ask for a dropdown filter and get an
essay about Kafka, sharding and CQRS. That noise teaches teams to skip design
entirely.

**System Design First fixes both.** The agent sizes the task (a typo fix gets
nothing, a new subsystem gets the full treatment), does the back-of-the-
envelope math, reads the real code before describing it, and writes down
every trade-off in plain language your product owner can push back on.

## What it looks like

Ask for a feature as you normally would:

> We need in-app and email notifications when a leave request is approved.
> 20k employees, 300 companies. The PM wants the bell to update "instantly".

Before touching code the agent hands you a brief. A real excerpt from a test
run:

```
## 2. Estimates
| Metric                       | Now              | +12 months | Source                  |
| Tenants / employees          | 300 / 20k        | 450 / 40k  | stated / assumed 2x     |
| Notifications + emails / day | ~5k + ~5k        | ~10k       | 1 event per 4 users/day |
| Open tabs at the 9:00 peak   | ~2k              | ~5k        | assumed 25 % of DAU     |
Conclusion: throughput is a non-issue at every layer - no Kafka, no separate
service. What shapes the design: one admin click fans out to 2,500 employees;
"instantly" means long-lived connections (polling every 5 s = 400 rps of empty
queries); Celery over Redis is at-least-once, so the email consumer must be
idempotent or retries send duplicates.

## 4. Target design
Pattern: SSE on a small ASGI process fed by Redis pub/sub (already your Celery
broker) - because traffic is one-way and 2k idle connections fit one node.
Rejected: WebSocket/Channels (extra dependency for one-way traffic), hosted
push vendor (HR data for 2k connections one node can hold), Redis unread
counter (one indexed query is enough at this volume).

## 7. Open questions for the author
- Does "instantly" mean < 2 s, or is 30 s polling acceptable? (decides Phase 2)
- Who receives ATS stage changes - candidates are external and get no bell.
```

Then it implements, step by step, according to the brief.

And when you ask it to fix a typo in a button label, you get the typo fixed.
One sentence, no brief. Proportionality is a rule, not a hope.

## Use when

The skill triggers on its own. It is pulling its weight whenever you say
things like:

- "Implement / plan / scope / estimate this feature"
- "How should we design X?" · "What's the best way to build Y?"
- "Will this scale?" · "We're going from 800 users to 50k next quarter"
- "Add notifications / export / search / real-time updates / an integration"
- "Do a design review of this endpoint before we touch it"
- "Explain how this part of the system works"
- Anything that mentions load, QPS, latency, availability, consistency,
  queues, caching or growth, even if nobody says "architecture"

## How it works

Six moves, adapted to real work where a codebase and an author with intent
already exist:

1. **Gauge the depth.** L0 trivial, L1 local change, L2 feature, L3
   subsystem. Each level has a fixed output size. Over-designing is a bug.
2. **Scope.** Functional vs non-functional requirements, what is out, what is
   assumed, and the author's own design extracted from the ticket and the
   draft code, not replaced.
3. **Estimate.** Users, peak QPS, storage growth, payload sizes, with sources.
   The numbers are then used to *kill* patterns, not just justify them.
4. **Map the current design** from the real code: components, data flow,
   state ownership, reusable primitives, inherited limits. Nothing invented.
5. **Propose the target design.** Text diagram, API contract, data model,
   each pattern with its reason and the rejected alternative, and the
   canonical system it resembles (notification system, feed fan-out, chat,
   rate limiter, export job, booking engine, ...).
6. **Deep dive, brief, build.** Idempotency, consistency, dependency failure,
   hot spots, unbounded growth, migration, security, observability. Then a
   one-page brief with fixed headings, in your language. Then the code.

## What's inside

| | |
| --- | --- |
| `SKILL.md` | The workflow, the depth gauge and the judgment rules (numbers before patterns, respect the author, read the code) |
| `references/framework.md` | Question checklists per step, the non-functional requirements list, how to extract the author's intent |
| `references/estimation.md` | Latency ladder, availability nines, QPS and storage math, single-node capacities, frontend budgets |
| `references/patterns.md` | Scaling, caching, queues, storage and consistency, real-time transport, rate limiting, IDs, reliability, observability, plus **25 canonical systems** with their building blocks and classic traps |
| `references/frontend.md` | The same pass for SPA and micro-frontend work: query caches, pagination, real-time, bundle boundaries, failure isolation |
| `references/brief-template.md` | The inline L1 check and a complete L2 brief you can hand to a reviewer |

The agent loads only what the task needs: a small change never touches the
catalog.

## Install

**Any agent, one command** (Claude Code, Codex, Cursor, Copilot, Gemini CLI,
OpenCode, ...):

```sh
npx skills add Baukaalm/system-design-first
```

**Claude Code plugin** (with updates through the marketplace):

```sh
claude plugin marketplace add Baukaalm/system-design-first
claude plugin install system-design-first@baukaalm
```

**Git clone** into every agent config dir found on the machine, or into the
tools you name:

```sh
git clone https://github.com/Baukaalm/system-design-first ~/.agents/system-design-first
~/.agents/system-design-first/install.sh            # or: ./install.sh claude cursor codex
```

**Manual.** The repository root is the skill folder; drop it into:

| Tool | User-level | Project-level |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| OpenAI Codex | `~/.codex/skills/` | `.codex/skills/` |
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |
| GitHub Copilot | `~/.copilot/skills/` | `.github/skills/` |
| Gemini CLI | `~/.gemini/skills/` | `.gemini/skills/` |
| Shared / other | `~/.agents/skills/` | `.agents/skills/` |

No skill support in your tool? Paste `adapters/AGENTS.md.snippet` into your
`AGENTS.md`, or drop `adapters/system-design-first.mdc` into
`.cursor/rules/`.

## Does it actually help?

Measured on the same model with and without the skill (three tasks, see
`evals/`):

- **Trivial fix:** identical result, no brief, negligible overhead.
- **New feature (notifications):** 3.5× more explicit numeric estimates, every
  pattern paired with a rejected alternative, open questions listed for the
  author instead of silently decided.
- **Scaling an existing service (chat, 800 → 50k users):** found a latent bug
  in the current code (an id counter that resets on restart), computed that
  polling would cost 10k rps, and kept the lead's WebSocket plan while adding
  only what the numbers demanded.

The price is roughly 1.5× tokens on large tasks. On a feature that ships to
production, that is the cheapest design review you will ever buy.

## License

MIT.

