# Specification: Reps, Core Training Log

Feature ID: 001-core
Status: **DRAFT, awaiting product owner approval**
Author: Engineering
Date: 2026-09-11
Governed by: [../constitution.md](../constitution.md)

> Working name is "Reps". Final name is [NEEDS CLARIFICATION: product owner to confirm
> app name and GitHub repository name].

---

## 1. Problem Statement

A person who trains at a gym receives a workout plan from a coach. That plan changes
over time: the coach adds exercises, raises the weight, changes the number of sets.
Between sessions the athlete sometimes repeats last week's workout unchanged and
sometimes follows a revised one.

Today that plan lives in a notebook, a phone note, or the coach's head. This creates
four concrete failures:

1. **The plan and the reality drift apart.** What the coach prescribed and what the
   athlete actually lifted are never compared, so neither party knows if the program
   is working.
2. **Progress is invisible.** Without a history of load and volume, the athlete
   cannot tell whether they are getting stronger, stalling, or regressing.
3. **Weak areas go unnoticed.** Nobody counts that chest was trained eleven times
   this month and legs twice.
4. **Sessions get skipped.** Nothing prompts the athlete on the days they are meant
   to train.

## 2. Users

| Persona | Description | Primary need |
|---|---|---|
| **Athlete (primary)** | Trains 3 to 6 times a week. Follows a plan from a coach. Wants to get stronger and see it happening. | Log fast, see progress, be reminded |
| **Friend / peer** | Someone the athlete invites. Uses the app independently for their own training. | Same as athlete, plus copy a program a friend shares |
| **Coach (v1: as a peer)** | Writes plans for others. In v1 they use the app as a normal user and hand plans over by sharing a program. A dedicated coach dashboard is out of scope for v1 (Constitution, Article IX). | Author a program once, hand it to athletes |

## 3. Glossary

Terms are used with exactly these meanings throughout the project. Ambiguity here
becomes bugs later.

| Term | Meaning |
|---|---|
| **Exercise** | A movement, for example "Barbell Bench Press". Belongs to a catalog. |
| **Set** | One group of repetitions performed without rest. The user described these as "turns". |
| **Rep** | One repetition of the movement within a set. |
| **Load** | The weight used for a set, in kilograms or pounds. |
| **Program** | A reusable, named training plan, for example "Coach Push Pull Legs, Sept". Contains one or more Workouts. |
| **Workout** | A named day within a Program, for example "Day A: Push". Contains an ordered list of Prescribed Exercises. |
| **Prescribed Exercise** | An exercise inside a Workout, with target sets, target reps, and target load as written by the coach. This is the plan. |
| **Session** | One actual visit to the gym on a specific date, performed against a Workout. This is the reality. |
| **Logged Set** | One set the athlete actually completed during a Session: reps done, load used, and whether it was completed or skipped. |
| **Personal Record (PR)** | The athlete's best ever performance on an exercise, tracked as heaviest load and as best estimated one rep maximum. |
| **Volume** | Sets multiplied by reps multiplied by load, summed over a period. The primary measure of training work. |
| **Program Revision** | A versioned snapshot of a Program. When a coach changes sets or load, a new revision is created and the old one is preserved. |

## 4. User Stories

Priority: **P1** ships in v1. **P2** ships in v1 if capacity allows. **P3** is a later version.

### Epic A: Account and identity

- **A1 (P1)** As a new user, I can create an account with my email and a password, so that my training history is mine and private.
- **A2 (P1)** As a user, I can sign in on any device and see my data, so that I am not tied to one phone.
- **A3 (P1)** As a user who forgot my password, I can reset it by email, so that I do not lose my history.
- **A4 (P1)** As a user, I can sign out, and I can see and revoke my active sessions on other devices.
- **A5 (P1)** As a user, I can permanently delete my account and all my data.
- **A6 (P2)** As a user, I can export my entire training history as a file.

### Epic B: Planning the workout

- **B1 (P1)** As an athlete, I can create a Program and add Workouts to it, so that the plan my coach gave me lives in the app.
- **B2 (P1)** As an athlete, I can add exercises to a Workout with target sets, reps, and load, so that I know what I am supposed to do today.
- **B3 (P1)** As an athlete, I can pick exercises from a built-in catalog that includes a picture of the movement, so that I know what the exercise actually is.
- **B4 (P1)** As an athlete, I can create a custom exercise when the catalog does not have it.
- **B5 (P1)** As an athlete, I can reorder exercises within a Workout by dragging.
- **B6 (P1)** As an athlete whose coach changed the plan, I can edit a Workout's targets, and the app records that the plan changed on that date without destroying what I did before the change.
- **B7 (P1)** As an athlete, I can assign Workouts to days of the week, so the app knows that Monday is Push day.
- **B8 (P2)** As an athlete, I can duplicate an existing Program as a starting point for a new one.

### Epic C: Doing the workout

- **C1 (P1)** As an athlete arriving at the gym, I see today's Workout on the home screen without searching for it, and I can start it in one tap.
- **C2 (P1)** As an athlete, I can start a Session from any Workout, not only today's, so that I can train out of order.
- **C3 (P1)** As an athlete mid exercise, I can mark a set complete in a single tap, and the reps and load are pre filled with the prescribed target.
- **C4 (P1)** As an athlete who lifted something different from the plan, I can adjust the reps and load for that set before or after marking it complete.
- **C5 (P1)** As an athlete, I can add an extra set beyond the prescribed count, or skip a prescribed set with an optional reason.
- **C6 (P1)** As an athlete, when I complete a set a rest timer starts automatically, and I am alerted when the rest period is over even if my screen is off.
- **C7 (P1)** As an athlete, I can repeat my previous Session for this Workout with its actual loads pre filled, so that "same as last time" takes one tap.
- **C8 (P1)** As an athlete with no phone signal, I can complete an entire Session, and it uploads by itself once I have signal.
- **C9 (P1)** As an athlete, I can pause and resume a Session, and if I close the app mid workout my progress is exactly where I left it when I reopen.
- **C10 (P1)** As an athlete finishing a Session, I see a summary: total volume, duration, sets completed against sets planned, and any PRs set.
- **C11 (P2)** As an athlete, I can attach a note to a set or a Session, for example "left shoulder pinched".
- **C12 (P2)** As an athlete at a barbell, I can see which plates to load for a target weight.
- **C13 (P2)** As an athlete, I get a nudge if I have been idle for longer than my rest period, so that sessions do not drag.

### Epic D: Seeing progress and finding weak areas

- **D1 (P1)** As an athlete, I can see a chart of load over time for any single exercise.
- **D2 (P1)** As an athlete, I am told immediately when I set a personal record, during the session and not afterwards.
- **D3 (P1)** As an athlete, I can see total training volume per week and per muscle group.
- **D4 (P1)** As an athlete, I can see which muscle groups I have trained least over the last 30 days, so I know what I am neglecting. This is the "areas for improvement" view.
- **D5 (P1)** As an athlete, I can see my current streak of weeks in which I hit my training target.
- **D6 (P1)** As an athlete, I am shown a suggested target for my next session on each exercise, based on what I actually did previously, so I progress instead of repeating the same weight forever.
- **D7 (P2)** As an athlete, I am warned when my weekly volume jumps far above my recent average, because that is how people get injured.
- **D8 (P2)** As an athlete, I can see adherence: the percentage of prescribed sets I actually completed, per week.
- **D9 (P2)** As an athlete, I can log body weight and measurements and chart them alongside training volume.

### Epic E: Reminders

- **E1 (P1)** As an athlete, I can set a reminder time for each training day, and receive a push notification on my phone at that time.
- **E2 (P1)** As an athlete, I can turn all reminders off, or snooze today's.
- **E3 (P1)** As an athlete on iPhone, I am told clearly during onboarding that I must add the app to my home screen for reminders to work, because that is an operating system restriction.
- **E4 (P1)** As an athlete who has not trained in longer than my usual gap, I receive a gentle nudge, not a guilt trip.
- **E5 (P2)** As an athlete mid session, I receive a notification when my rest timer ends, even with the screen locked.

### Epic F: Sharing and social

- **F1 (P1)** As a user, I can generate an invite link and send it to a friend or my coach, and when they sign up through it we are connected.
- **F2 (P1)** As a user, I can share one of my Programs, and a connected user can copy it into their own account as their own editable Program.
- **F3 (P1)** As a user, my logged Sessions are private by default and no other user can read them unless I share them.
- **F4 (P2)** As a user, I can share a progress card showing selected stats, and revoke that share at any time.
- **F5 (P3)** Friends activity feed, reactions, leaderboards and challenges.

### Epic G: Getting started

- **G1 (P1)** As a first time user, I see a branded loading screen while the app boots, so it feels like an app and not a slow web page.
- **G2 (P1)** As a first time user with an empty account, I am offered a small set of ready made starter Programs (Push Pull Legs, Upper Lower, Full Body 3 Day) so that I am not staring at a blank screen.
- **G3 (P1)** As a user, I can install the app to my home screen and open it full screen with no browser chrome.

## 5. Functional Requirements

Each requirement is testable per Constitution Article II.

### Authentication and accounts

| ID | Requirement | Acceptance criterion |
|---|---|---|
| FR-001 | Users register with email and password | Registration with a valid, unused email and a compliant password creates an unverified account and sends a verification email within 60 seconds |
| FR-002 | Passwords are checked for strength | A password shorter than 12 characters, or appearing in a known breached password list, is rejected with a specific message saying why |
| FR-003 | Passwords are stored using Argon2id | No plaintext or reversibly encrypted password exists in any table, log, or backup. Verified by inspecting the users table after registration |
| FR-004 | Email addresses are verified | An unverified account can sign in but cannot send invites or share programs until verified |
| FR-005 | Sign in issues a short lived access credential and a long lived rotating refresh credential | Access credential expires within 15 minutes. Refresh credential rotates on every use and the previous value is invalidated |
| FR-006 | Reuse of an already used refresh credential revokes the entire token family | Replaying an old refresh token signs the user out of every device and records a security event |
| FR-007 | Authentication credentials are never readable by client side script | No auth token appears in `localStorage`, `sessionStorage`, or any non httpOnly cookie |
| FR-008 | Failed sign in attempts are rate limited | After 5 failed attempts for one account within 15 minutes, further attempts are rejected for that account regardless of source address, and the user is emailed |
| FR-009 | Password reset uses a single use, time limited token | A reset token expires in 30 minutes, works once, and does not reveal whether an email is registered |
| FR-010 | Users can list and revoke active device sessions | Revoking a device session causes the next request from that device to fail authentication |
| FR-011 | Users can delete their account | Deletion removes or irreversibly anonymizes all personal data within 24 hours and is confirmed by a re-entry of the password |
| FR-012 | Every data access is scoped to the authenticated user | A request for a resource owned by another user returns "not found", never "forbidden", and never the resource |

### Programs and workouts

| ID | Requirement | Acceptance criterion |
|---|---|---|
| FR-020 | A user can create, rename, archive and delete Programs | An archived Program no longer appears in the active list but its historical Sessions remain intact and viewable |
| FR-021 | A Workout contains an ordered list of Prescribed Exercises | Reordering persists and is reflected in the next Session started from that Workout |
| FR-022 | A Prescribed Exercise stores target sets, target reps (fixed or a range), target load, and target rest seconds | All four are optional individually; an exercise with no targets is still loggable |
| FR-023 | Editing a Prescribed Exercise creates a new Program Revision | After an edit, Sessions performed before the edit still display the targets that were in force on the date they were performed |
| FR-024 | A built in exercise catalog is available | The catalog contains at least 120 exercises, each with a name, primary muscle group, secondary muscle groups, equipment type, and an illustrative image |
| FR-025 | Users can create custom exercises | A custom exercise is visible only to its creator and behaves identically to a catalog exercise everywhere else |
| FR-026 | Workouts can be scheduled to days of the week | The home screen shows the Workout scheduled for the current day in the user's local timezone |

### Sessions and logging

| ID | Requirement | Acceptance criterion |
|---|---|---|
| FR-030 | Starting a Session snapshots the current prescription | Changing the Program after a Session has started does not alter that in progress Session |
| FR-031 | Marking a set complete requires exactly one tap when the prescribed values are accepted | Measured from the active session screen with the exercise visible |
| FR-032 | Logged Sets record reps, load, completion state, and timestamp | Values default to the prescription, or to the previous Session's actuals when "repeat last" is used |
| FR-033 | Sets can be added beyond, or skipped within, the prescription | Both are visible in the Session summary as a variance against plan |
| FR-034 | A rest timer starts automatically on set completion and is configurable per exercise | Timer continues to run accurately while the app is backgrounded |
| FR-035 | All Session writes succeed with no network connection | With the device in airplane mode, a complete Session can be logged, the app can be closed and reopened, and no data is lost. Data appears on a second device within 30 seconds of reconnecting |
| FR-036 | Conflicting offline edits resolve deterministically | Last write wins per Logged Set, using the client timestamp, with the losing value retained in an audit field |
| FR-037 | An interrupted Session is resumable | Force closing the app mid Session and reopening restores the exact scroll position, active exercise, and elapsed time |
| FR-038 | Session completion produces a summary | Summary shows duration, total volume, sets completed against planned, per exercise variance, and PRs achieved |

### Progress and insight

| ID | Requirement | Acceptance criterion |
|---|---|---|
| FR-040 | The app detects personal records on load and on estimated one rep max | A PR is surfaced within the same Session, at the moment the qualifying set is logged, not on a later screen |
| FR-041 | Estimated one rep max uses a documented formula | The formula and its limitations are stated in the UI. Epley is the default |
| FR-042 | Volume is computed per exercise, per muscle group, and per week | Volume for a set is reps multiplied by load. Bodyweight exercises use a configured bodyweight value |
| FR-043 | The app identifies under trained muscle groups | The "areas for improvement" view ranks muscle groups by total sets over the last 30 days and flags any trained fewer than a configurable threshold |
| FR-044 | The app suggests the next target per exercise | Suggestion is derived only from the user's own logged history, states the rule that produced it in plain language, and is always overridable |
| FR-045 | Streaks are computed against a user set weekly training target | A week counts toward the streak when completed Sessions meet or exceed the target. Weeks marked as deliberate rest do not break the streak |

### Reminders

| ID | Requirement | Acceptance criterion |
|---|---|---|
| FR-050 | Users can schedule a reminder per training day at a chosen local time | Notification is delivered within 5 minutes of the scheduled time |
| FR-051 | Push subscriptions are per device and revocable | Revoking on one device does not affect others. An expired or rejected subscription is removed automatically |
| FR-052 | Reminders can be globally disabled and individually snoozed | Disabling stops all delivery within one scheduling cycle |
| FR-053 | Notification permission is requested in context, never on first load | The permission prompt appears only after the user has chosen to enable a reminder |
| FR-054 | iOS users are told about the home screen requirement | Users on iOS Safari who have not installed the app see an explanation before being offered reminders |
| FR-055 | Inactivity nudges respect the user's own pattern | A nudge sends only after the user's median gap between sessions has been exceeded by 48 hours, and at most once per 72 hours |

### Sharing

| ID | Requirement | Acceptance criterion |
|---|---|---|
| FR-060 | Users generate single purpose invite links | An invite link expires after 14 days or a configurable number of uses, whichever comes first, and can be revoked |
| FR-061 | Programs can be shared to connected users, and copied | A copied Program is a fully independent record. Later edits by the original author do not propagate |
| FR-062 | Sessions and logs are private by default | A newly created account has zero outbound shares. No API path returns another user's Logged Sets |
| FR-063 | Any share can be revoked | Revocation takes effect on the next request, with no cached bypass |

### Platform and experience

| ID | Requirement | Acceptance criterion |
|---|---|---|
| FR-070 | The app is installable to a device home screen | It meets installability criteria and launches full screen with an app icon and a splash screen |
| FR-071 | A branded loading screen shows during boot | Visible from first paint until the app is interactive, with no flash of unstyled content |
| FR-072 | Exercise images are shown in the catalog, the planner, and the active session | Images are optimized, lazy loaded below the fold, and have descriptive alt text |
| FR-073 | Starter Programs are offered to empty accounts | At least three starter Programs are available and can be adopted in one tap |
| FR-074 | The interface meets WCAG 2.2 AA | Verified by automated audit plus manual keyboard traversal of every primary flow |

## 6. Non-Functional Requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-001 | Time to interactive on a mid range Android phone over 4G | Under 2.5 seconds on first visit, under 1 second on repeat visit |
| NFR-002 | Set logging interaction latency | Under 100 milliseconds, local only, no network in the path |
| NFR-003 | Availability | Best effort. Free tier hosting. Offline capability is the mitigation, not redundancy |
| NFR-004 | Database cold start tolerance | The UI must never block on a cold database. Reads render from local cache first |
| NFR-005 | Running cost | 0.00 per month at up to 100 users (Constitution, Article VII) |
| NFR-006 | Supported browsers | Last 2 versions of Chrome, Safari, Firefox, Edge. iOS Safari 16.4 and above for push |
| NFR-007 | Data durability | No acknowledged write is lost. Offline queue survives app termination and device restart |
| NFR-008 | Privacy | No third party analytics or advertising scripts |

## 7. Out Of Scope For v1

Recorded so they are not silently reintroduced:

- Coach dashboard with athlete roster and remote program assignment
- Activity feed, reactions, leaderboards, challenges (Epic F5)
- Nutrition, calorie, or macro tracking
- Wearable, Apple Health, or Google Fit integration
- Video form capture or analysis
- Cardio and distance based training as a first class type
- Payments, subscriptions, premium tiers
- Native App Store or Play Store distribution
- Multi language support

## 8. Open Questions

These block the plan being finalized. Per Constitution Article I they must be answered, not guessed.

| ID | Question | Owner | Status |
|---|---|---|---|
| Q-001 | Final app name and GitHub repository name, and public or private visibility | Product owner | **OPEN** |
| Q-002 | Default unit: kilograms or pounds | Product owner | Proposed: kilograms, user switchable |
| Q-003 | Should a deliberate "rest week" be a first class concept in streak calculation | Product owner | Proposed: yes, FR-045 assumes it |
| Q-004 | Is a weekly training target set by the user, or inferred from their schedule | Product owner | Proposed: inferred from scheduled days, user overridable |

## 9. Success Criteria

v1 is successful when, without touching a notebook:

1. The product owner logs four consecutive weeks of real gym sessions in the app.
2. Zero sessions are lost to connectivity or crashes.
3. The product owner can answer "am I stronger than last month, and what am I neglecting" from the app in under 15 seconds.
4. At least one invited friend independently logs a session of their own.

---

**Approval**

This spec is not implemented until the product owner signs off below and the open
questions in section 8 are closed.

- [ ] Product owner approval
