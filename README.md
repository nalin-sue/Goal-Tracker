# Goal-Tracker
Yearly goals tracker mobile application with daily activity log.

# How to run backend
1. Install Java JDK21
1. Run `./mvnw quarkus:dev` in terminal. (`mvnw.cmd quarkus:dev` on Windows)
2. Go to `http://localhost:8080/q/swagger-ui` in browser to test APIs.

# How to run frontend
1. Install Flutter SDK 3.44+ and make sure `flutter`/`dart` are on PATH.
2. From `frontend/`, run `flutter pub get`, then `flutter run -d chrome` (web) or `flutter run` (mobile, with an emulator running or a device connected).

# Planning
## Requirement Planning
1. Sketch or write requirements (functional, non-functional), UX workflow, screen mockups, a data model, and API list
2. Review the draft. Answer edge cases and find contradictions between sections. Update document and add special case sections.
3. Decide project tech stack
   - Frontend framework & state management approach = Flutter & Riverpod
   - Backend framework & API style = Quarkus, imperative JAX-RS
   - Database = PostgreSQL
   - Testing frameworks
     - Backend = JUnit5 + RestAssured
     - Frontend = flutter_test, mocktail
   - Repo structure = Monorepo, /backend and /frontend folders at root
   - Authentication = JWT, Google OAuth + guest mode

## Project Initilization Steps with Claude Code
1. From my plan document, write the spec requirement doc (.md) — full data model, API contracts, business rules, "read this every time" cautions, and the Working Agreement (see below) as its own section at the top.
```
Working Agreement with Claude Code (Read this before every slice.)
This project is being implemented slice by slice with Claude Code.
The following rules apply to every single slice, without exception:

1. Plan first, always. Before writing or modifying any code, present a clear plan (what will be built, files touched, key decisions) and wait for explicit approval before implementing.
2. One slice at a time. Do not implement, scaffold, or touch any other slice or feature beyond what was just approved.
3. Security by default. Apply security best practices relevant to the feature being built (input validation, auth checks, no secrets in code, parameterized queries, etc.) without being asked each time.
4. Consistent conventions. Follow best-practice naming and coding conventions for the relevant language/framework (Dart/Flutter, Java/Quarkus).
5. Log every change. After each slice is completed, add an entry to the progress log claude-log.md (date, topic, key decision, files touched, summary paragraph).
```

2. Write the agent briefing files (backend engineer, frontend engineer, reviewer) — generic enough to reuse across all features
3. Write Slice 0 (scaffolding) prompts.
4. Write Login feature prompts (backend slices → frontend slices → review prompt) in detail.
5. Create the progress log template claude-log.md (date / topic / decision / files touched table).
6. Future features get their prompt files generated one at a time, after the previous feature ships — not all upfront.

# Backend

Quarkus (Java 21, Maven) app in `/backend`, backed by PostgreSQL. This slice only proves the app boots and can reach Postgres — no entities, auth, or business endpoints yet.

**Layout** (`backend/src/main/java/com/goaltracker/`):
- `resource/` — JAX-RS endpoints (controllers). No business logic, no SQL. Currently just `HealthResource` (`GET /health`).
- `service/` — business logic, called by resources. Empty placeholder (`.gitkeep`) for now.
- `repository/` — data access, called by services. No SQL in resources or services. Empty placeholder (`.gitkeep`) for now.

`docker-compose.yml` and `.env.example` live in `backend/` (not the repo root), since they're backend-only concerns.

**Run locally:**

1. From `backend/`, copy the env template and fill in real values:
   ```
   cd backend
   cp .env.example .env
   ```
2. Start Postgres:
   ```
   docker-compose up -d
   ```
3. Export the same variables into your shell (docker-compose auto-loads `.env`, but the Maven-run app does not), then start the app in dev mode:
   ```
   export $(grep -v '^#' .env | xargs)
   ./mvnw quarkus:dev
   ```
   On Windows PowerShell, instead of `export`:
   ```
   Get-Content .env | Where-Object { $_ -match '=' } | ForEach-Object { $k,$v = $_ -split '=',2; Set-Item "Env:$k" $v }
   .\mvnw.cmd quarkus:dev
   ```
4. Confirm it's up:
   ```
   curl http://localhost:8080/health
   ```

# Frontend

Flutter app (mobile + web) in `/frontend`, using Riverpod for state management. This slice only proves the app boots — default counter-app placeholder screen, no Goal-Tracker features yet.

**Layout** (`frontend/lib/`):
- `ui/` — widgets and screens.
- `state/` — Riverpod providers and app state.
- `repository/` — mediates between state and the API client; the only thing widgets/state should talk to for data.
- `api_client/` — HTTP/REST client code; the only layer that knows about the backend's wire format.

**Run locally:**

1. Install the Flutter SDK (3.44+) and make sure `flutter`/`dart` are on PATH — confirm with:
   ```
   flutter doctor
   ```
2. From `frontend/`, fetch dependencies:
   ```
   cd frontend
   flutter pub get
   ```
3. Run on web:
   ```
   flutter run -d chrome
   ```
4. Run on mobile — start an Android emulator (Android Studio → Device Manager) or connect a physical device with USB debugging enabled, then:
   ```
   flutter devices
   flutter run
   ```
