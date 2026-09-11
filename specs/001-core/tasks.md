# Tasks

Feature ID: 001-core | Derived from [plan.md](./plan.md)
Status: **Not started. Blocked on spec approval (Constitution Article I).**

Each task is independently testable and names the requirements it closes. A task is
not done until it meets the Definition of Done in `plan.md` section 8.

Legend: `[ ]` not started, `[~]` in progress, `[x]` done, `[!]` blocked.

---

## Phase 0: Foundation

| # | Task | Closes |
|---|---|---|
| [ ] 0.1 | Scaffold Next.js 15 + TypeScript strict + Tailwind v4, commit lockfile | - |
| [ ] 0.2 | CI workflow: typecheck, lint, unit tests, `npm audit`, secret scan | T10, T11 |
| [ ] 0.3 | Design tokens from `design/mockups.html` into `src/styles/tokens.css`, with a contrast assertion test | FR-074 |
| [ ] 0.4 | Base components: button, field, card, chip, set row, bottom nav | - |
| [ ] 0.5 | App manifest, icons, Serwist service worker, offline app shell | FR-070 |
| [ ] 0.6 | Boot screen, no flash of unstyled content | FR-071 |
| [ ] 0.7 | Security headers in middleware, with an automated header test | Section 5 of security.md |
| [ ] 0.8 | Neon project, Drizzle config, first migration, `.env.example` | - |

## Phase 1: Identity

| # | Task | Closes |
|---|---|---|
| [ ] 1.1 | `users` table and Drizzle schema | - |
| [ ] 1.2 | Argon2id hash and verify, with documented parameters and transparent rehash | FR-003 |
| [ ] 1.3 | Password policy including HIBP k-anonymity check with offline fallback | FR-002 |
| [ ] 1.4 | Register endpoint, Zod validated, timing-equalized response | FR-001, T8 |
| [ ] 1.5 | Email verification tokens, hashed at rest, single use, Resend delivery | FR-004 |
| [ ] 1.6 | Ed25519 keypair, JWT issue and verify via JOSE, `__Host-` cookies | FR-005, FR-007 |
| [ ] 1.7 | Refresh rotation with token families and replay revocation | FR-006 |
| [ ] 1.8 | Postgres rate limiter, applied per the table in security.md | FR-008, T13 |
| [ ] 1.9 | Password reset, single use, 30 minute expiry, no account disclosure | FR-009 |
| [ ] 1.10 | Device session list and revoke UI | FR-010 |
| [ ] 1.11 | Account deletion with password re-entry and a purge job | FR-011 |
| [ ] 1.12 | Repository layer with a mandatory actor, plus the ESLint rule banning direct `db` import | FR-012, T6 |
| [ ] 1.13 | **Gate:** cross-user access test returns 404 for every owned resource type | FR-012 |
| [ ] 1.14 | **Gate:** refresh replay test proves the family is revoked | FR-006 |
| [ ] 1.15 | Sign in and register screens per mockup 02 | - |

## Phase 2: Catalog and planning

| # | Task | Closes |
|---|---|---|
| [ ] 2.1 | `muscle_groups` and `exercises` schema, with the per-owner unique index | - |
| [ ] 2.2 | Source, optimize and commit 120+ exercise images with alt text and `CREDITS.md` | FR-024, FR-072 |
| [ ] 2.3 | Seed script for muscle groups and the exercise catalog | FR-024 |
| [ ] 2.4 | Exercise picker with search, muscle filter and images | FR-024 |
| [ ] 2.5 | Custom exercise creation, visible only to its owner | FR-025 |
| [ ] 2.6 | `programs`, `program_revisions`, `workouts`, `prescribed_exercises` schema | - |
| [ ] 2.7 | Program and workout CRUD | FR-020 |
| [ ] 2.8 | Prescribed exercise editor: sets, rep range, load, rest, superset group | FR-022 |
| [ ] 2.9 | Drag to reorder, touch and keyboard accessible | FR-021 |
| [ ] 2.10 | Copy-on-write revision creation, with a test proving old sessions keep old targets | FR-023 |
| [ ] 2.11 | Weekday scheduling in the user's timezone | FR-026 |
| [ ] 2.12 | Plan editor screen per mockup 06 | - |
| [ ] 2.13 | Three starter programs, adoptable in one tap | FR-073 |

## Phase 3: The session loop (online only)

| # | Task | Closes |
|---|---|---|
| [ ] 3.1 | `sessions` and `logged_sets` schema with the `(user_id, exercise_id, performed_at)` index | - |
| [ ] 3.2 | Start session, snapshotting the revision and pre-seeding `planned` set rows | FR-030 |
| [ ] 3.3 | One-tap set completion, 48px target, optimistic UI | FR-031, FR-032 |
| [ ] 3.4 | Inline adjust of reps and load | FR-032 |
| [ ] 3.5 | Add set beyond plan, skip set with reason | FR-033 |
| [ ] 3.6 | Rest timer, per-exercise duration, accurate while backgrounded | FR-034 |
| [ ] 3.7 | Repeat last session with actuals pre-filled | C7 |
| [ ] 3.8 | Resume an interrupted session at the exact position | FR-037 |
| [ ] 3.9 | Session summary: duration, volume, variance against plan | FR-038 |
| [ ] 3.10 | Today screen per mockup 03, active session per mockup 04 | C1 |
| [ ] 3.11 | Playwright: full session happy path | - |

## Phase 4: Offline

| # | Task | Closes |
|---|---|---|
| [ ] 4.1 | Dexie schema mirroring the server tables the client needs | - |
| [ ] 4.2 | UUIDv7 generation on the client | - |
| [ ] 4.3 | Local-first repository: UI reads and writes IndexedDB only | NFR-002, NFR-004 |
| [ ] 4.4 | `outbox` table, written in the same transaction as the mutation | FR-035 |
| [ ] 4.5 | Background sync drain, oldest first, with retry and backoff | FR-035 |
| [ ] 4.6 | Idempotent server upsert keyed on client UUID | NFR-007 |
| [ ] 4.7 | Pull-since-watermark reconciliation | FR-035 |
| [ ] 4.8 | Conflict resolution, last write wins, loser kept in `superseded_value` | FR-036 |
| [ ] 4.9 | Sync state indicator in the UI (synced, pending, failed) | - |
| [ ] 4.10 | **Gate:** Playwright offline test. Start session, go offline, log 12 sets, kill the context, reopen, finish, reconnect, assert every set reached the server | FR-035, NFR-007 |

## Phase 5: Insight

| # | Task | Closes |
|---|---|---|
| [ ] 5.1 | `personal_records` and history tables | - |
| [ ] 5.2 | PR detection on set completion, computed locally so it fires offline | FR-040 |
| [ ] 5.3 | Epley estimated 1RM, with the formula stated in the UI | FR-041 |
| [ ] 5.4 | Volume calculation per exercise, muscle group and week, with bodyweight handling | FR-042 |
| [ ] 5.5 | Per-exercise load chart, custom SVG, emphasized endpoint | D1 |
| [ ] 5.6 | Weekly volume chart | FR-042 |
| [ ] 5.7 | Muscle balance ranking and under-training flags | FR-043 |
| [ ] 5.8 | Progressive overload rule per `plan.md` section 4, with the reason shown in words | FR-044 |
| [ ] 5.9 | Streaks against the weekly target, rest weeks do not break them | FR-045 |
| [ ] 5.10 | Progress screen per mockup 07, improvements screen per mockup 08 | D4, D6 |
| [ ] 5.11 | Unit tests for every analysis function, including the deload branch | - |

## Phase 6: Reminders

| # | Task | Closes |
|---|---|---|
| [ ] 6.1 | VAPID keypair, `push_subscriptions` schema | - |
| [ ] 6.2 | Subscribe and unsubscribe, permission requested only in context | FR-051, FR-053 |
| [ ] 6.3 | Auto-remove subscriptions on 404 or 410 | FR-051 |
| [ ] 6.4 | `reminder_schedules` and `reminder_deliveries` with the idempotency key | FR-050 |
| [ ] 6.5 | Signed `/api/cron/reminders` endpoint, constant-time secret comparison | T12 |
| [ ] 6.6 | GitHub Actions workflow on a 5 minute schedule | FR-050 |
| [ ] 6.7 | Timezone-correct due calculation, with a DST boundary test | FR-050 |
| [ ] 6.8 | Global disable and per-day snooze | FR-052 |
| [ ] 6.9 | iOS install guidance shown to uninstalled iOS Safari users | FR-054 |
| [ ] 6.10 | Inactivity nudge based on the user's own median gap | FR-055 |
| [ ] 6.11 | Rest timer and idle notifications from the service worker | E5, C13 |
| [ ] 6.12 | Reminders screen per mockup 09 | - |

## Phase 7: Sharing

| # | Task | Closes |
|---|---|---|
| [ ] 7.1 | `invites`, `connections`, `program_shares` schema | - |
| [ ] 7.2 | Invite generation, 128 bits of entropy, hashed at rest, expiry and use cap | FR-060, T9 |
| [ ] 7.3 | Invite redemption during signup, creating the connection | F1 |
| [ ] 7.4 | Program share and deep copy into the recipient's account | FR-061 |
| [ ] 7.5 | Revocation effective on the next request | FR-063 |
| [ ] 7.6 | **Gate:** test proving no API path returns another user's logged sets | FR-062 |
| [ ] 7.7 | Partners screen per mockup 10 | - |

## Phase 8: Hardening and release

| # | Task | Closes |
|---|---|---|
| [ ] 8.1 | Full keyboard traversal of every primary flow | FR-074 |
| [ ] 8.2 | Automated accessibility audit, zero violations | FR-074 |
| [ ] 8.3 | Screen reader pass on the active session screen | FR-074 |
| [ ] 8.4 | Performance budget enforced in CI against NFR-001 | NFR-001 |
| [ ] 8.5 | Data export as JSON | Constitution VI.1 |
| [ ] 8.6 | Run `/security-review`, resolve or document every finding | security.md section 10 |
| [ ] 8.7 | Verify git history contains no secret | T10 |
| [ ] 8.8 | Repo-wide check for em dashes in shipped strings | Constitution VIII.1 |
| [ ] 8.9 | Deploy to Vercel, verify the Actions cron reaches production | - |
| [ ] 8.10 | Install on a real iPhone and a real Android, log one real session on each | Success criteria |

---

## Critical path

`0.1 -> 0.8 -> 1.6 -> 1.12 -> 2.6 -> 3.2 -> 3.3 -> 4.3 -> 4.10 -> 5.2 -> 8.10`

Task 4.10 is the single highest risk gate in the project. If offline sync is not
provably correct, the product fails its one unacceptable failure mode: losing a
workout you already did.
