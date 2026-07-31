# Initiation
Create the project's folder structure only — no code yet.

This is a monorepo for both frontend and backend. /docs already exists with goal-tracker-spec.md and the agent-*.md briefing files — leave it as is. Create:

/goal-tracker
  /frontend
  /backend
  claude-log.md
  README.md
  .gitignore

Empty claude-log.md with just a one-line header. README.md should say only what this repo is and that /frontend and /backend are separate apps — no setup instructions yet, those come with their own slices.

Don't initialize Flutter or Quarkus/Spring projects yet — that's the next two slices. Just the two folders and those few files.

Present your plan before creating anything, per the Working Agreement.

Create CLAUDE.md in backend and frontend directories, referencing `docs/agent-backend-engineer.md` `docs/agent-frontend-engineer.md`

# Scaffolding
## Backend
initialize the backend project in /backend — no feature code yet.

Pick Java Quarkus and scaffold the project inside /backend.
Basic folder structure: resource/, service/, repository/ — empty except for a one-line README comment in each explaining what belongs there.
docker-compose.yml (repo root) that spins up a local PostgreSQL instance.
App connects to Postgres using environment variables — no hardcoded credentials. Include .env.example.
One endpoint: GET /health returning 200, no auth. That's the only thing you need to prove works end-to-end this slice.
Add a short "Backend" section to the root README.md with the exact commands to run it locally (docker-compose up, then run the app, then hit /health).

No entities, no JWT/auth logic, no business endpoints — those are separate slices. Just prove the app boots and can talk to Postgres.

Present your plan before running anything, per the Working Agreement.

## Frontend
initialize the Flutter project in `/frontend` — no feature code yet.
- `flutter create` (or equivalent) inside `/frontend`, targeting mobile + web.
- Add the state-management dependency you'll use going forward (pick one — Riverpod or Provider — and say which in your plan and why).
- Basic folder structure: `ui/`, `state/`, `repository/`, `api_client/` — empty except for a one-line README comment in each explaining what belongs there.
- Confirm it runs: default counter-app screen is fine as the placeholder, don't build anything Goal-Tracker-specific yet.
- Add a short "Frontend" section to the root `README.md` with the exact commands to run it locally.

No design system, no navigation shell, no mock data, no real screens — those are separate slices. Just prove the Flutter project boots on both a mobile emulator and web.

Present your plan before running anything, per the Working Agreement.


# Login Feature

Ordered as: Backend (B1–B4) → Frontend (F1–F4) → Review. Paste each one individually, in order — don't combine them, and don't start the next until the current one is approved and logged, per the Working Agreement.

All slices assume `/docs/goal-tracker-spec.md` and the relevant `agent-*.md` briefing have already been read.

---

## Backend

### B1 — User entity & migration

Implement just the `User` table — no endpoints yet.

- Create the `User` entity exactly matching the spec's data model (§4.1): `id` (PK), `google_id` (nullable), `username`, `profile_pic` (nullable — string, per §6.8's preset-avatar/Google-URL model, not a file upload), `language`, `email`, `created_at`, `updated_at`, `last_login_at`.
- Flyway/Liquibase migration to create the table. No seed data.
- Repository layer with basic `findById`, `findByGoogleId`, `save` — no service/resource layer yet, this slice is data-layer only.
- Unit test: repository can save and retrieve a user with `google_id = NULL` (guest) and one with a `google_id` set, confirming the nullable column actually behaves as nullable at the DB level, not just in the entity class.

**Out of scope**: `User_Years` (that's created later, when a user actually adds a year — not part of login), any endpoint, any auth logic.

Present your plan (migration approach, exact column types/constraints) before writing code.

---

### B2 — Google login endpoint

Implement `POST /login` for the **Google flow only**.

- Accept a Google ID token from the client (the client obtains this via Google Sign-In SDK — verifying that flow itself is a frontend concern, not this slice).
- Verify the token server-side against Google's public keys (don't trust an unverified token — this is a real auth boundary, treat it with the security rigor in `agent-backend-engineer.md` §4).
- Look up `User` by `google_id`:
  - If found: update `last_login_at`, issue a JWT.
  - If not found: create a new `User` (from the verified token's claims — email, name, picture URL), then issue a JWT.
- JWT payload should carry at minimum the user's internal `id` — no sensitive data beyond what's needed for auth checks on other endpoints.
- Do **not** implement guest login or the migration flow in this slice — Google-only.
- Tests: valid token → 200 + JWT; invalid/expired/tampered token → 401; second login with the same `google_id` updates `last_login_at` rather than creating a duplicate user.

Present your plan (which Google token verification library, exact JWT claims, expiry duration) before writing code.

---

### B3 — Guest login + logout

- `POST /login` (guest variant, or a separate flag/param on the same endpoint — your call, state it in your plan): creates a new `User` with `google_id = NULL` and no email, issues a JWT the same way as B2. Every guest login creates a *new* guest user — there's no "returning guest" concept at this stage (device-level guest persistence, if any, is a frontend concern).
- `POST /logout`: since JWTs are stateless, decide and document whether this is a no-op (client just discards the token) or whether you're implementing token blacklisting/revocation — don't silently pick the more complex option without flagging the tradeoff in your plan.
- Tests: guest login issues a valid JWT scoped to a `google_id = NULL` user; that user can hit an authenticated test endpoint successfully.

**Out of scope**: guest → Google migration (that's B4).

---

### B4 — Guest → Google migration

This is the most business-rule-dense part of login — re-read spec §6.2 in full before starting, don't work from memory of it.

- New endpoint (design it — not explicitly named in the spec's API table, so propose one, e.g. `POST /users/migrate-guest`) that: given an authenticated guest session and a Google ID token, reassigns the guest's data to the Google account per §6.2's rules:
  - If the Google account has no existing data: reassign `user_id` on the guest's rows to the Google account, delete the guest `User` row.
  - If **both** accounts have data: reassign `user_id` on all migrating rows; if a `Yearly_Goals.goal_name` collides within the same `year_id`, **block that item's migration** and return enough detail for the client to show the "please rename first" prompt from §6.2 — don't silently overwrite or auto-rename.
  - If both accounts have a `User_Years` row for the same year: keep one, repoint all its children (`Yearly_Goals`, `Activity_Log`, `Bucket_List`) to the surviving id, delete the now-empty duplicate.
- The whole operation is **transactional** — any failure rolls back completely, per §6.2's "if error occurs, rollbacks" rule.
- Tests: no-conflict migration; goal-name-collision case (verify it blocks and reports correctly, doesn't partially apply); duplicate-`User_Years`-for-same-year case (verify child rows all repoint to the surviving id).

Present your plan (exact endpoint shape, exact collision-detection query) before writing code — this is the slice most worth getting a second look on before implementing, given the transaction complexity.

---

## Frontend

### F1 — Login screen (static)

Build the screen itself, no real auth wired up yet.

- Match spec §1 exactly: centered layout, emoji icon (default 🚀, using the Fluent Emoji asset per the design system) above "Goal Tracker" title, one-line tagline, "Continue with Google" (filled/primary) and "Continue as guest" (outlined/secondary) buttons.
- Buttons are non-functional for this slice — tapping either can navigate to the existing nav-shell placeholder from Slice 0, but don't call any real or mock auth logic yet.
- Use the design-system tokens from Slice 0 (colors, typography casing) rather than hardcoding styles here.

---

### F2 — Google Sign-In integration

- Wire up the Google Sign-In SDK for Flutter, get a real ID token on "Continue with Google" tap.
- Call the real `POST /login` (B2) with that token; on success, store the returned JWT in secure storage (already wired in Slice 0) and navigate into the app shell.
- Handle failure states (network error, Google auth cancelled by user, backend rejects token) with appropriately calm messaging — no alarming red error dialogs for a cancelled sign-in, that's a normal user action, not an error.
- Test on both a mobile emulator and web, since Google Sign-In's flow differs by platform (native SDK vs. web OAuth redirect) — don't assume the mobile implementation "just works" on web.

---

### F3 — Guest login

- "Continue as guest" calls `POST /login` (guest variant, B3), stores the resulting JWT the same way as F2, navigates into the app shell.
- No local-only/offline-account concept beyond what B3 already provides — a guest login always talks to the backend to get a real (if `google_id`-less) account and JWT, consistent with B3's design.

---

### F4 — Guest → Google migration UI

- When a guest user taps "Connected with Google" (or an equivalent entry point — check Settings §3.2) and signs in with Google while guest data exists, show the **"Import your guest data?"** prompt from spec §6.2.
- On confirm, call the B4 migration endpoint. Handle the three outcomes distinctly:
  - Success → proceed into the Google-authenticated app state.
  - Goal-name collision → show the exact rename-prompt wording from §6.2 ("You have a goal named '{X}' in both, please rename them first (on local)"), and don't proceed until resolved.
  - Any other failure → calm, non-alarming messaging (per the save-status design philosophy elsewhere in the spec) — the guest data isn't at risk even if migration fails, so don't imply data loss.

---

## Review

Once all backend and frontend slices for this feature are complete, run this against `agent-reviewer.md`'s full checklist, with extra attention to:

- **B2/B3**: is the Google ID token actually verified server-side, not just decoded and trusted?
- **B4**: trace the collision-handling and duplicate-`User_Years` logic end-to-end against §6.2 — this is the single most likely place for a subtle bug to hide, given how much conditional logic it has.
- **F2/F4**: does the UI ever imply data loss or use alarming styling for normal outcomes (cancelled sign-in, pending migration)? Flag anything that contradicts the calm-messaging philosophy established elsewhere in the spec.
- Process: was each of B1–B4 and F1–F4 planned and approved individually, or did any slice quietly expand scope into the next one?