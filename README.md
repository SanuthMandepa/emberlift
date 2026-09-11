# Reps

A training log for people who follow a coach's plan. Plan the workout, log what you
actually lifted against what was prescribed, get reminded on training days, and find
out what you are neglecting.

Built as an installable web app (PWA) so it runs on any phone, works with no signal
in the middle of a gym, and costs nothing to host.

> **Status: specification phase.** No application code exists yet, by design. See
> [How this project is built](#how-this-project-is-built) below.

---

## How this project is built

This repository follows **spec-driven development**. Intent is written down, reviewed
and approved before anything is implemented. The rule is enforced by
[`specs/constitution.md`](specs/constitution.md), Article I.

The chain is:

```
constitution.md   Principles that override everything. Amendments are logged.
       |
       v
spec.md           WHAT and WHY. Written in the user's language.
                  Zero technology choices. Every requirement has an ID
                  and a testable acceptance criterion.
       |
       v
plan.md           HOW. Stack, architecture, phases. Every decision cites
                  the requirement that forces it.
       |
       v
tasks.md          Ordered, individually testable units of work. Each task
                  names the requirement IDs it closes.
       |
       v
code              Only ever written to satisfy a task.
```

The point is not ceremony. It is that when you are four weeks in and wondering why
the refresh token rotates, or why sessions are written to IndexedDB before the
network, the answer is written down and traceable to a requirement.

### The documents

| Document | What it settles |
|---|---|
| [`specs/constitution.md`](specs/constitution.md) | Non-negotiable principles: spec before code, security requirements, the gym floor as the design target, accessibility, data ownership, zero running cost |
| [`specs/001-core/spec.md`](specs/001-core/spec.md) | The product. Glossary, user stories, 60+ numbered requirements with acceptance criteria, explicit out-of-scope list, open questions |
| [`specs/001-core/plan.md`](specs/001-core/plan.md) | The technical plan. Stack decisions with justifications, rejected alternatives, architecture, offline sync design, the progressive overload rule, build phases, risks |
| [`specs/001-core/data-model.md`](specs/001-core/data-model.md) | Every table, column and index, and why the program revision model preserves training history |
| [`specs/001-core/security.md`](specs/001-core/security.md) | Threat model, Argon2id parameters, token rotation and replay detection, authorization enforcement, headers, rate limits, verification gates |
| [`specs/001-core/tasks.md`](specs/001-core/tasks.md) | 80+ ordered tasks across 9 phases, with the critical path marked |
| [`design/mockups.html`](design/mockups.html) | Ten screen designs, the design tokens, and the requirement IDs each screen satisfies |

## What it does

- **Plan.** Build the program your coach gives you: workouts, exercises, target sets,
  reps, load and rest. When the coach changes something, the app writes a new revision
  instead of overwriting, so a session from August still shows August's targets.
- **Log.** One tap per set. Prescribed values are pre-filled, so "did what it said"
  is a single thumb press. Change reps or weight when reality differs. Add a set,
  skip a set, repeat last week's session exactly.
- **Work offline.** Gyms have terrible reception. The phone is the source of truth
  during a session. Nothing waits on the network, and nothing is lost.
- **Beat it.** Personal records are announced the moment you set them. The next
  target on every lift is suggested from your own history, and the app tells you in
  plain words which rule produced the suggestion.
- **Improve.** Volume per muscle group over 30 days, so the fact that you have done
  42 sets of chest and 8 of legs stops being invisible.
- **Show up.** Push notifications on training days, a rest timer that alerts with the
  screen off, and a nudge when you have been away longer than your own usual gap.
- **Share.** Invite friends or your coach. They get their own account and their own
  training. Sharing a program hands over a copy. Your logs stay private.

## Stack

| | |
|---|---|
| App | Next.js 15, TypeScript strict, Tailwind v4 |
| Database | Neon Postgres, Drizzle ORM |
| Auth | Written in house: Argon2id, Ed25519 JWTs, rotating refresh tokens with replay detection |
| Offline | IndexedDB via Dexie, outbox queue, background sync |
| Push | Web Push with VAPID. No third party push service |
| Scheduler | GitHub Actions cron |
| Hosting | Vercel |

Every one of these choices is justified against a requirement in
[`plan.md`](specs/001-core/plan.md) section 1, including the ones that were rejected.

Total running cost: nothing. That is a constitutional requirement, not a happy
accident.

## Design

Ten screens, the palette, the type scale and the colour discipline are in
[`design/mockups.html`](design/mockups.html). Open it in a browser.

The short version: a dark instrument panel, tabular numerals you can read at arm's
length, and a single volt accent spent only on what is live right now and what you
just beat.

## Current state

Nothing is implemented. The specification is drafted and awaiting approval, and four
open questions in [`spec.md`](specs/001-core/spec.md) section 8 are still open.

When the spec is approved, work starts at task 0.1 in
[`tasks.md`](specs/001-core/tasks.md).

## Conventions

- **No em dashes** anywhere in this repository. Constitution, Article VIII.
- Secrets never enter git. `.env` is ignored from the first commit.
- A requirement that cannot be tested is not a requirement.
