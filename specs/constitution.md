# Project Constitution

Version 1.1.0 | Ratified 2026-09-11 | Last amended 2026-09-11

This document governs every decision in this project. A spec, plan, or pull request
that violates an article here is rejected, not debated. Amendments require a dated
entry in the Amendment Log at the bottom.

---

## Article I: Spec Before Code

No implementation begins without an approved spec.

The order is fixed:

1. **Constitution** (this file). Principles and non-negotiables.
2. **Specification** (`spec.md`). What we are building and why. Written in the
   language of the user, not the language of the framework. Contains zero
   technology choices.
3. **Plan** (`plan.md`). How we will build it. Stack, architecture, data model,
   API contracts. Every decision traces back to a requirement in `spec.md`.
4. **Tasks** (`tasks.md`). Ordered, individually testable units of work. Each task
   names the requirement IDs it satisfies.
5. **Implementation**. Code that satisfies a task. Nothing else.

Any requirement that is ambiguous is marked `[NEEDS CLARIFICATION: question]` in the
spec and must be resolved by the product owner before the plan is written. Guessing
is forbidden.

## Article II: Every Requirement Is Testable

A requirement that cannot be verified is not a requirement, it is a wish.

Each functional requirement carries a stable ID (`FR-001`) and at least one
acceptance criterion phrased as an observable outcome. "The app should be fast" is
rejected. "A logged set appears in the session history within 100ms of the tap,
without a network round trip" is accepted.

## Article III: Security Is A Requirement, Not A Feature

The following are non-negotiable and apply from the first commit:

1. Passwords are hashed with **Argon2id**. Never MD5, SHA family, or plain bcrypt
   without justification. Parameters are documented in `security.md`.
2. Secrets never enter the repository. Not in code, not in config, not in a commit
   that gets amended later. `.env` is git-ignored on day one.
3. All authentication state lives in **httpOnly, Secure, SameSite cookies**. Tokens
   are never placed in `localStorage`.
4. Every database query that reads user-owned data is scoped by the authenticated
   user ID at the data access layer. Authorization is never left to the UI.
5. All input crossing a trust boundary is validated against a schema before use.
   Validation happens on the server even when it also happens on the client.
6. Rate limiting is applied to every unauthenticated endpoint.
7. Dependencies are pinned and audited. A known critical CVE blocks release.

## Article IV: The Gym Floor Is The Design Target

This app is used standing up, one handed, sweating, with a phone in a chalky grip,
on 1 bar of signal, between sets, under a 90 second clock.

1. Logging a completed set must be reachable in **one tap** from the active session
   screen.
2. Primary tap targets are at least **48x48 CSS pixels** and positioned in the lower
   half of the screen, within thumb reach.
3. The app must remain fully functional with **no network connection**. Writes queue
   locally and sync when connectivity returns. Losing a workout log is treated as a
   Sev 1 defect.
4. No interaction required during a set may take longer than **2 seconds** to
   complete.

## Article V: Accessibility Is Not Optional

1. Text contrast meets **WCAG 2.2 AA** (4.5:1 body, 3:1 large text and UI).
2. Every interactive element is keyboard reachable and has a visible focus state.
3. Color is never the sole carrier of meaning. A personal record is marked with an
   icon and a label, not only a green tint.
4. All animation respects `prefers-reduced-motion`.
5. Every image and icon that conveys meaning has a text alternative.

## Article VI: The User Owns Their Data

1. A user can export every record we hold about them in a machine readable format,
   on demand, without contacting support.
2. A user can permanently delete their account and all associated data.
3. Nothing a user logs is visible to another user unless the user explicitly and
   specifically shares it. Sharing defaults to off, always.
4. We collect no analytics that identify an individual without consent.

## Article VII: It Must Cost Nothing To Run

The entire production stack runs on free tiers. A proposal that introduces a
recurring cost must be rejected or explicitly approved by the product owner with the
monthly figure stated. This constraint is a feature: it forces simplicity.

## Article VIII: Writing Style

1. **No em dashes.** Not in product copy, not in code, not in comments, not in
   commit messages, not in pull request descriptions, not in any document in this
   repository. Use a comma, a colon, a period, or parentheses. This covers the
   character itself and its HTML entity forms.
   En dashes in numeric ranges (8 to 12 reps written as a range) are permitted.
2. Prose is plain and direct. No marketing voice in the product UI.
3. Error messages tell the user what happened and what to do next.

Enforced by `.github/workflows/guard.yml`, which fails the build on any violation.

## Article IX: Progressive Enhancement Of Scope

Version 1 ships a complete, useful product for one person training alone. Social
features, coach dashboards, and integrations are additive. The data model
accommodates them from the start (roles, ownership, sharing tables exist), but the
UI for them is not built until the core is proven in real gym sessions.

## Article X: Authorship And Attribution

This project is authored by Sanuth Mandepa. Tools used to produce it are not credited
in it.

1. **No AI is ever a contributor.** Claude, or any other assistant, must never appear
   as a commit author, a commit co-author, a committer, a name in the GitHub
   contributor graph, or an entry in any `AUTHORS`, `CONTRIBUTORS`, `package.json`
   author field, or license header.
2. **No attribution trailers in commit messages.** No `Co-Authored-By` line naming an
   assistant, no "Generated with" line, no "AI-assisted" note, no robot emoji.
3. **No attribution in pull request descriptions** or release notes.
4. Every commit is authored and committed under the human's own name and email.
5. This is not about concealment. It is about responsibility. Whoever commits the
   code has reviewed it and owns it, and a tool credit would blur that.

Enforced by `.github/workflows/guard.yml`, which scans the full commit history and
fails the build on any violation.

---

## Amendment Log

| Date | Version | Change |
|------|---------|--------|
| 2026-09-11 | 1.0.0 | Initial ratification. |
| 2026-09-11 | 1.1.0 | Added Article X (Authorship And Attribution). Strengthened Article VIII.1 to cover HTML entity forms and pull request descriptions. Both are now enforced in CI by `guard.yml`. |
