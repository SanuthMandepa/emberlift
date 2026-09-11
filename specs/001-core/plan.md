# Technical Plan: Reps, Core Training Log

Feature ID: 001-core
Status: **DRAFT, follows [spec.md](./spec.md)**
Date: 2026-09-11

Every decision below cites the requirement that forces it. A decision with no
citation is an unjustified decision and should be challenged in review.

---

## 1. Stack Decisions

| Layer | Choice | Why (requirement) |
|---|---|---|
| Client and server | **Next.js 15, App Router, TypeScript strict** | One deployable unit keeps cost at zero (NFR-005). Server Components give fast first paint (NFR-001) |
| Styling | **Tailwind CSS v4 + CSS custom properties for design tokens** | Tokens make the WCAG contrast contract auditable in one file (FR-074) |
| UI primitives | **Radix UI primitives, styled by us** | Keyboard and screen reader behaviour is correct by construction (FR-074). We own the visual layer so the app does not look generic |
| Motion | **Motion (framer-motion successor)**, gated on `prefers-reduced-motion` | Constitution Article V.4 |
| Charts | **Custom SVG built on d3-scale**, no chart library | Three chart types only (FR-040, FR-042). A chart library is 80KB we do not need (NFR-001) |
| Database | **Neon Postgres, free tier** | Relational data (programs, sets, sessions) is deeply relational. Free (NFR-005). Serverless driver avoids connection exhaustion |
| Data access | **Drizzle ORM** | Typed schema, SQL-first, tiny runtime. Migrations live in the repo and are reviewable |
| Validation | **Zod**, one schema per boundary, shared client and server | Constitution Article III.5 |
| Auth | **Written in house**: Argon2id, JOSE for JWT, rotating refresh tokens | Explicit product owner requirement. See [security.md](./security.md) |
| Local persistence | **IndexedDB via Dexie**, with an outbox table | FR-035, FR-037, NFR-007 |
| Sync | **Custom outbox + pull-on-reconnect**, last write wins per Logged Set | FR-035, FR-036 |
| Service worker | **Serwist** (maintained Workbox successor) | FR-070, FR-071, offline shell |
| Push | **Web Push, VAPID, `web-push` library** | FR-050. Free and vendor independent (NFR-005) |
| Scheduler | **GitHub Actions cron** calling a signed internal endpoint every 5 minutes | Vercel Hobby cron granularity is one run per day, which cannot satisfy FR-050's 5 minute window. Actions is free |
| Transactional email | **Resend free tier** | Needed for FR-001, FR-004, FR-008, FR-009. Not used for reminders |
| Rate limiting | **Postgres backed fixed window counters** | FR-008. Avoids adding a paid Redis (NFR-005) |
| Hosting | **Vercel Hobby** | NFR-005 |
| Testing | **Vitest** (unit), **Playwright** (end to end, including an offline airplane mode run) | FR-035 cannot be verified any other way |
| Images | **Static, pre-optimized AVIF/WebP in the repo, served from the CDN** | FR-072 with no image hosting bill (NFR-005) |

### Explicitly rejected

- **Supabase / Clerk / NextAuth**: the product owner asked to own authentication. Rejecting these is a deliberate cost, paid in the discipline of [security.md](./security.md).
- **Firebase**: document store is a poor fit for set and session relations.
- **Redis**: a recurring cost, and Postgres handles our rate limit volume.
- **React Native / Expo**: requires a paid Apple Developer account for iOS distribution, violating Constitution Article VII.

## 2. Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Phone (installed PWA)                                  │
│                                                         │
│  React UI ──► Local store (Dexie/IndexedDB)  ◄─ source  │
│     │              │                            of      │
│     │              │  outbox queue              truth   │
│     │              ▼                            while   │
│     │         Service Worker (Serwist)          offline │
│     │         - app shell cache                         │
│     │         - background sync                         │
│     │         - push notification display               │
└─────┼───────────────┼───────────────────────────────────┘
      │               │  (when online)
      ▼               ▼
┌─────────────────────────────────────────────────────────┐
│  Next.js on Vercel                                      │
│                                                         │
│  Route Handlers ──► Zod validation ──► Auth guard       │
│                                            │            │
│                                            ▼            │
│                                   Repository layer      │
│                              (every query scoped by     │
│                               authenticated user id)    │
└────────────────────────────────┬────────────────────────┘
                                 ▼
                     ┌───────────────────────┐
                     │  Neon Postgres        │
                     └───────────────────────┘
                                 ▲
                                 │ signed request every 5 min
                     ┌───────────────────────┐
                     │  GitHub Actions cron  │──► Web Push ──► Phone
                     └───────────────────────┘
```

**The critical property**: the phone's IndexedDB is the source of truth during a
session. The UI never awaits the network to record a set (NFR-002, FR-035). The
server is a sync target, not a dependency.

## 3. Offline And Sync Design

This is the highest risk area in the build, so it is specified rather than improvised.

1. Every mutation is written to IndexedDB **and** appended to an `outbox` table in
   the same transaction. The UI updates from IndexedDB. It never waits on fetch.
2. A background sync handler drains the outbox oldest first. Each entry carries a
   client generated **UUIDv7** primary key, so a retry is idempotent on the server
   (`ON CONFLICT (id) DO UPDATE`).
3. On reconnect the client pulls changes since its last watermark using a server
   `updated_at` cursor.
4. Conflicts are resolved per Logged Set by client timestamp, last write wins
   (FR-036). The losing value is written to `logged_sets.superseded_value` as JSON
   so nothing is silently destroyed.
5. **Test gate**: a Playwright run that starts a session, goes offline, logs twelve
   sets, terminates the browser context, reopens, finishes the session, returns
   online, and asserts every set is on the server. This test is required to pass
   before v1 ships.

## 4. Progressive Overload Rule (FR-044)

Stated explicitly so it is reviewable rather than magic. For each exercise, look at
the most recent completed Session:

- If **every** prescribed set met or exceeded its target reps, suggest a load
  increase: the smaller of 2.5% or one equipment increment (2.5kg barbell, 2kg
  dumbbell, one stack pin on a machine).
- If reps were met on some but not all sets, suggest **the same load** and state
  "match all sets before adding weight".
- If reps fell short on two consecutive sessions, suggest a **10% deload** and state
  why.
- If there is no history, suggest nothing and ask the user for a starting load.

The rule that fired is always shown in words next to the suggestion. The suggestion
is a suggestion, never an auto-applied value.

## 5. Repository Structure

```
/
├── specs/                        Spec-driven artifacts. Source of truth for intent.
│   ├── constitution.md
│   └── 001-core/
│       ├── spec.md               WHAT and WHY
│       ├── plan.md               HOW (this file)
│       ├── data-model.md         Schema
│       ├── security.md           Threat model and controls
│       └── tasks.md              Ordered, testable work units
├── design/                       Visual mockups, reviewed before build
├── src/
│   ├── app/                      Next.js routes
│   │   ├── (auth)/               Sign in, register, reset
│   │   ├── (app)/                Authenticated app shell
│   │   └── api/                  Route handlers
│   ├── components/
│   ├── db/                       Drizzle schema and migrations
│   ├── lib/
│   │   ├── auth/                 Argon2, tokens, guards
│   │   ├── sync/                 Outbox, conflict resolution
│   │   ├── analysis/             Volume, PRs, overload rule
│   │   └── push/                 VAPID, subscriptions
│   ├── local/                    Dexie schema, offline repository
│   └── styles/                   Design tokens
├── public/
│   └── exercises/                Optimized exercise images
├── tests/
│   ├── unit/
│   └── e2e/                      Includes the offline gate
└── .github/workflows/
    ├── ci.yml                    Typecheck, lint, test, audit
    └── reminders.yml             The 5 minute scheduler
```

## 6. Build Phases

Each phase ends in something demonstrable. No phase starts before the previous one
is merged and green.

| Phase | Delivers | Requirements closed |
|---|---|---|
| **0. Foundation** | Repo, CI, tokens, design system, loading screen, installable shell | FR-070, FR-071 |
| **1. Identity** | Register, verify, sign in, refresh rotation, reset, device sessions, delete account | FR-001 to FR-012 |
| **2. Catalog and planning** | Exercise catalog with images, program and workout CRUD, scheduling, revisions | FR-020 to FR-026 |
| **3. The session loop** | Start session, one tap logging, rest timer, repeat last, summary. **Online only at first** | FR-030 to FR-034, FR-038 |
| **4. Offline** | Dexie, outbox, background sync, conflict handling, the Playwright offline gate | FR-035 to FR-037, NFR-007 |
| **5. Insight** | PR detection, charts, volume, muscle balance, streaks, overload suggestions | FR-040 to FR-045 |
| **6. Reminders** | Push subscriptions, schedules, Actions cron, iOS install guidance, nudges | FR-050 to FR-055 |
| **7. Sharing** | Invites, connections, program share and copy, revocation | FR-060 to FR-063 |
| **8. Hardening** | Accessibility audit, security review, seed content, performance budget | FR-073, FR-074, NFR-001 |

Phase 3 is deliberately built online first. Building the session loop and the sync
engine at the same time is how this kind of project fails.

## 7. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| iOS push requires home screen install | Reminders silently do nothing for iPhone users | FR-054 makes this explicit in onboarding and in settings. Treated as a product requirement, not a caveat |
| Neon free tier auto-suspends when idle | Cold start delays first request | NFR-004: the UI renders from local cache and never blocks on the database |
| In house auth is a real risk surface | Account compromise | [security.md](./security.md), a dedicated hardening phase, and `/security-review` before release |
| Offline sync complexity | Lost workouts, the one unacceptable failure | Phase 4 isolated, deterministic conflict rule, mandatory Playwright gate |
| Exercise image licensing | Legal exposure | Only public domain or explicitly permissive sources. Provenance recorded in `public/exercises/CREDITS.md` |
| Scope creep from Epic F | v1 never ships | Constitution Article IX. Social is P3 |

## 8. Definition Of Done (per task)

1. Satisfies its cited requirement IDs.
2. Unit tests for logic, Playwright coverage for any user facing flow.
3. Typecheck and lint clean, no `any`, no suppressed errors.
4. Keyboard traversable, focus visible, contrast checked.
5. No secret, key, or credential added to the repository.
6. No em dash in any string, comment, or document (Constitution Article VIII).
