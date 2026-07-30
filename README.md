# Goal-Tracker
Yearly goals tracker mobile application with daily activity log.

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
