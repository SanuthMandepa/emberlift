# Data Model

Feature ID: 001-core | Postgres 16 | Drizzle ORM

Conventions used throughout:

- Primary keys are **UUIDv7**, generated on the client. Time ordered, so they index
  well, and they let an offline device create records without asking the server for
  an ID (FR-035).
- Every user owned table carries `user_id` and is **always** queried through a
  repository function that injects the authenticated user ID (FR-012).
- Timestamps are `timestamptz`, stored in UTC. The user's timezone lives on `users`.
- Money-like numbers (loads) are `numeric(6,2)`, never floats. 2.5kg increments must
  be exact.
- Soft delete via `archived_at` where history matters. Hard delete only for account
  deletion (FR-011).

---

## Entity relationships

```
users ──┬── device_sessions ── refresh_tokens (families)
        ├── push_subscriptions
        ├── reminder_schedules
        ├── custom exercises ────────────┐
        ├── programs                     │
        │      └── program_revisions     │
        │             └── workouts       │
        │                    └── prescribed_exercises ──► exercises
        ├── sessions ──► workouts        │
        │      └── logged_sets ──────────┘
        ├── personal_records ──► exercises
        ├── body_metrics
        ├── connections (user ↔ user)
        ├── invites
        └── program_shares
```

---

## Identity

### `users`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | UUIDv7 |
| `email` | citext UNIQUE NOT NULL | Case insensitive. Unique index |
| `email_verified_at` | timestamptz NULL | FR-004 |
| `password_hash` | text NOT NULL | Argon2id encoded string. Never leaves the server (FR-003) |
| `password_updated_at` | timestamptz NOT NULL | Invalidates older tokens |
| `display_name` | text NOT NULL | |
| `role` | enum(`athlete`,`coach`,`admin`) DEFAULT `athlete` | Exists in v1, UI for `coach` deferred (Constitution IX) |
| `unit_preference` | enum(`kg`,`lb`) DEFAULT `kg` | Q-002 |
| `timezone` | text NOT NULL | IANA name. Drives FR-026 and FR-050 |
| `bodyweight_kg` | numeric(5,2) NULL | For bodyweight exercise volume (FR-042) |
| `weekly_target_sessions` | smallint NULL | FR-045 |
| `failed_login_count` | smallint DEFAULT 0 | FR-008 |
| `locked_until` | timestamptz NULL | FR-008 |
| `created_at` / `updated_at` | timestamptz | |
| `deleted_at` | timestamptz NULL | Triggers purge job (FR-011) |

### `device_sessions`

One row per signed in device. Revoking a row signs that device out (FR-010).

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | |
| `user_id` | uuid FK CASCADE | |
| `family_id` | uuid NOT NULL | Refresh token family (FR-006) |
| `user_agent` / `ip_hash` | text | `ip_hash` is salted, never the raw address |
| `last_seen_at` | timestamptz | |
| `revoked_at` | timestamptz NULL | |
| `expires_at` | timestamptz NOT NULL | |

### `refresh_tokens`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | |
| `family_id` | uuid NOT NULL, indexed | All tokens descending from one sign in |
| `token_hash` | bytea NOT NULL | SHA-256 of the token. The raw value is never stored (FR-005) |
| `used_at` | timestamptz NULL | Non-null plus a presentation equals replay, revoke the family (FR-006) |
| `expires_at` | timestamptz NOT NULL | |

### `verification_tokens`

Covers email verification and password reset. `token_hash` only, single use,
`purpose` enum(`email_verify`,`password_reset`), `expires_at` (FR-009).

### `security_events`

Append only. `user_id`, `type` (`login_failed`, `token_replay`, `password_changed`,
`account_locked`, `share_revoked`), `metadata` jsonb, `created_at`. Never contains
credentials.

### `rate_limits`

`key` text PK (for example `login:user:<uuid>`), `window_start` timestamptz, `count`
int. Fixed window, swept by the same cron that sends reminders (FR-008).

---

## Exercise catalog

### `muscle_groups`

`id`, `slug`, `name`, `region` enum(`upper_push`,`upper_pull`,`legs`,`core`,`other`).
Seeded, not user editable. Powers FR-043.

### `exercises`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | |
| `user_id` | uuid FK NULL | **NULL means a catalog exercise visible to everyone.** Non-null means a custom exercise visible only to that user (FR-025) |
| `name` | text NOT NULL | |
| `primary_muscle_group_id` | uuid FK | |
| `secondary_muscle_group_ids` | uuid[] | For volume attribution at 0.5 weighting |
| `equipment` | enum(`barbell`,`dumbbell`,`machine`,`cable`,`bodyweight`,`kettlebell`,`band`,`other`) | Drives the plate calculator and load increments |
| `load_increment_kg` | numeric(4,2) | Smallest jump possible. Feeds the overload rule |
| `image_path` | text NULL | Static asset path (FR-024, FR-072) |
| `image_alt` | text NULL | Required when `image_path` is set (Constitution V.5) |
| `is_bodyweight` | boolean | Volume uses `users.bodyweight_kg` |

Unique index on `(coalesce(user_id, '00000000-...'), lower(name))` prevents duplicates
per owner.

---

## Planning

### `programs`

`id`, `user_id`, `name`, `description`, `source_program_id` (set when copied from a
share, FR-061), `archived_at`, timestamps.

### `program_revisions`

The mechanism behind FR-023. A Program always has exactly one current revision.

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | |
| `program_id` | uuid FK CASCADE | |
| `revision_number` | int NOT NULL | Monotonic per program |
| `effective_from` | timestamptz NOT NULL | |
| `note` | text NULL | For example "coach raised bench to 60kg" |

Unique on `(program_id, revision_number)`.

**How edits work**: editing a prescription copies the current revision's workouts and
prescribed exercises into a new revision, applies the edit there, and sets
`effective_from = now()`. Historical Sessions point at the revision in force when
they were performed, so past targets are never rewritten.

### `workouts`

`id`, `program_revision_id` FK CASCADE, `name` ("Day A: Push"), `position` smallint,
`scheduled_weekdays` smallint[] (0 to 6, FR-026), `estimated_minutes`.

### `prescribed_exercises`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | |
| `workout_id` | uuid FK CASCADE | |
| `exercise_id` | uuid FK RESTRICT | |
| `position` | smallint | Drag to reorder (FR-021) |
| `target_sets` | smallint NULL | The "number of turns" |
| `target_reps_min` / `target_reps_max` | smallint NULL | Equal values mean a fixed target, different values mean a range (FR-022) |
| `target_load_kg` | numeric(6,2) NULL | |
| `target_rest_seconds` | smallint NULL | Drives the auto rest timer (FR-034) |
| `superset_group` | smallint NULL | Same value equals performed back to back |
| `note` | text NULL | Coach cue |

---

## Doing the work

### `sessions`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | Client generated so an offline session exists immediately (FR-035) |
| `user_id` | uuid FK CASCADE | |
| `workout_id` | uuid FK SET NULL | Null allowed for an ad hoc session |
| `program_revision_id` | uuid FK SET NULL | The prescription snapshot in force (FR-030) |
| `performed_on` | date NOT NULL | User's local date, not UTC |
| `started_at` / `ended_at` | timestamptz | `ended_at` null means in progress (FR-037) |
| `status` | enum(`in_progress`,`completed`,`abandoned`) | |
| `total_volume_kg` | numeric(10,2) | Denormalized on completion, for fast charts |
| `note` | text NULL | FR-011 |
| `client_updated_at` | timestamptz NOT NULL | Conflict resolution input (FR-036) |

Index on `(user_id, performed_on DESC)`.

### `logged_sets`

The highest volume table. Everything else exists to produce these rows.

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | UUIDv7, client generated, idempotent upsert key |
| `session_id` | uuid FK CASCADE | |
| `user_id` | uuid FK CASCADE | Denormalized so FR-012 scoping needs no join |
| `exercise_id` | uuid FK RESTRICT | |
| `prescribed_exercise_id` | uuid FK SET NULL | Null for an extra set beyond plan (FR-033) |
| `set_index` | smallint NOT NULL | 1 based, within the exercise |
| `reps` | smallint NULL | |
| `load_kg` | numeric(6,2) NULL | Always stored in kg. Display converts |
| `rpe` | numeric(3,1) NULL | Optional effort rating |
| `status` | enum(`completed`,`skipped`,`planned`) | `planned` rows are pre-seeded from the prescription so one tap flips to `completed` (FR-031) |
| `skip_reason` | text NULL | FR-005 |
| `is_warmup` | boolean DEFAULT false | Excluded from volume and PRs |
| `performed_at` | timestamptz | |
| `client_updated_at` | timestamptz NOT NULL | FR-036 |
| `superseded_value` | jsonb NULL | The losing side of a conflict. Nothing is silently destroyed |

Index on `(user_id, exercise_id, performed_at DESC)`. This single index serves PR
detection, the per exercise chart, and the overload rule.

### `personal_records`

Derived, but materialized because FR-040 requires detection **during** the session
and a full history scan is too slow on a phone.

`id`, `user_id`, `exercise_id`, `type` enum(`max_load`,`estimated_1rm`,`max_reps_at_load`),
`value` numeric, `reps`, `load_kg`, `logged_set_id` FK, `achieved_at`.
Unique on `(user_id, exercise_id, type)` holding the current best, with superseded
records kept in `personal_record_history` for the progress chart.

### `body_metrics`

`id`, `user_id`, `measured_on` date, `bodyweight_kg`, `measurements` jsonb, `note`.
FR-009 (P2).

---

## Reminders

### `push_subscriptions`

`id`, `user_id`, `endpoint` text UNIQUE, `p256dh_key`, `auth_key`, `user_agent`,
`created_at`, `last_success_at`, `failure_count`. A 404 or 410 from the push service
deletes the row (FR-051).

### `reminder_schedules`

`id`, `user_id`, `workout_id` NULL, `weekday` smallint, `local_time` time,
`kind` enum(`workout`,`inactivity`), `enabled` boolean, `snoozed_until` timestamptz.

### `reminder_deliveries`

`id`, `schedule_id`, `scheduled_for` timestamptz, `sent_at`, `status`.
Unique on `(schedule_id, scheduled_for)`, which makes the cron idempotent. A cron run
that fires twice cannot double-notify.

---

## Sharing

### `invites`

`id`, `inviter_user_id`, `code_hash` bytea, `max_uses` smallint, `use_count`,
`expires_at`, `revoked_at`. Only the hash is stored, matching the token pattern
(FR-060).

### `connections`

`id`, `user_a_id`, `user_b_id`, `status` enum(`pending`,`accepted`,`blocked`),
`created_via_invite_id`. Ordered pair constraint `user_a_id < user_b_id` plus a
unique index makes a duplicate connection impossible.

### `program_shares`

`id`, `program_id`, `owner_user_id`, `shared_with_user_id` NULL,
`visibility` enum(`link`,`connection`), `token_hash` NULL, `revoked_at`.

**Copy semantics (FR-061)**: accepting a share performs a deep copy of the program,
its current revision, workouts and prescribed exercises into the recipient's account
with new IDs and `source_program_id` set. There is no live link. The original author
cannot later see or change the copy, and cannot see the recipient's logs.

---

## Local (IndexedDB, Dexie)

Mirrors the server tables the client needs offline, plus:

### `outbox`

`seq` auto increment PK, `entity_type`, `entity_id` uuid, `operation`
(`upsert`/`delete`), `payload` json, `client_updated_at`, `attempts`, `last_error`.

Drained oldest first. Because every payload carries a client generated UUID and the
server upserts on conflict, replaying the queue is safe (plan.md section 3).

---

## Data retention and deletion

| Data | Retention |
|---|---|
| Training records | Until the user deletes them |
| `security_events` | 180 days |
| `reminder_deliveries` | 30 days |
| Expired tokens | Purged nightly by the cron |
| Deleted account | Hard delete of all rows within 24 hours (FR-011) |

## Seed data

- ~40 muscle groups and regions
- 120+ catalog exercises with images, alt text, equipment and load increments (FR-024)
- 3 starter programs: Push Pull Legs, Upper Lower, Full Body 3 Day (FR-073)
