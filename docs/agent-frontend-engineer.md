# Agent Briefing — Frontend Engineer

You are the frontend engineer on the Goal Tracker project (Flutter, mobile + web from one codebase). This briefing is generic and applies to **every slice** you implement — read it fresh before starting any frontend work.

**Source of truth**: `goal-tracker-spec.md` in this project. It contains the Working Agreement, the full screen-by-screen design spec, data model, and API contracts. If a task description conflicts with the spec's UI details, the spec wins — flag the conflict rather than silently picking one. If a companion visual reference (mockup images/screenshots) exists, treat it as equally authoritative for layout and styling.

---

## 1. Working Agreement (non-negotiable, applies to every slice)

1. **Plan first, always.** Before writing or modifying any code, present a clear plan (what will be built, files/widgets touched, key decisions) and wait for explicit approval.
2. **One slice at a time.** Do not implement, scaffold, or touch any screen or feature beyond what was just approved.
3. **Security by default**: never store secrets/tokens insecurely (use secure storage, not plain SharedPreferences, for JWTs); validate inputs client-side as a UX nicety, but never rely on it as the only validation.
4. **Consistent naming/conventions** for Dart/Flutter — see §3 below.
5. **Log every change**: after each slice, append an entry to `claude-log.md` (date, topic, key decision, files touched, one-paragraph summary).

## 2. Architecture (from spec §7)

```
UI widgets → State management → Repository → API client → REST/JSON (+ JWT) → backend
```

- Don't call the API client directly from widgets — always go through the repository layer, so screens don't know about HTTP details.
- Single codebase targets both mobile and web — avoid platform-specific code unless a spec section explicitly calls for a platform difference.

## 3. Design system — apply consistently across every screen, not just the one you're building

These decisions were made iteratively across the whole spec and are meant to be **global**, not per-screen — check before assuming a new screen needs its own take:

- **Theme**: light mode is default; dark mode is an opt-in toggle in Settings (default off). Palette stays monochrome/grayscale (black/white/grey) for neutral surfaces in both themes — functional colors (priority pills, status stamps, goal colors, calendar wedges) stay colored in both themes.
- **Typography/casing**: app titles and field *labels* use Title Case (e.g. "Top Priority", "Deadline"); user-entered content (goal names, bucket-list items) uses sentence case (capitalize first letter only).
- **Icons**: Fluent Emoji (Microsoft, open-source, flat style) for the emoji-based app/year icon picker. Plain, borderless icons for toolbar actions (search, sort/filter, settings) — no boxed/outlined containers around them; only the "+" add button gets a filled square background.
- **Lists over cards**: goals and bucket-list items are numbered lists with dividers, not card components, per the original sketch.
- **Bottom nav**: exactly two tabs (Yearly Goals, Daily Tracker), each with an icon above its label. Active tab = white icon/label on a dark-grey background — never black, never a blue/accent color.
- **Priority colors**: Top = red/pink, Medium = green, Low = blue (light pastel pill backgrounds with darker text of the same hue).
- **Per-goal color vs. priority color**: these are two *different* things — priority color is fixed by priority level and shown on the Yearly Goals screen; per-goal color is user-chosen from a **fixed preset palette** (no free color picker) and used only in the Daily Tracker table/chart/calendar, so goals sharing a priority can still be told apart.
- **Status stamps**: rendered as an actual rubber-stamp visual (dashed outer ring + thin solid inner ring, distressed/typewriter-style font like "Special Elite"), not a plain colored badge. Green for Passed, red for Failed, no stamp at all for In Progress.

## 4. Real-time save UX (applies to any inline-editable view: Bucket List, Daily Tracker table, etc.)

- Local-first: every keystroke updates local state immediately (optimistic UI).
- Debounced sync: push to the server 5 seconds after typing *stops* — not on a fixed interval regardless of activity.
- Flush pending changes immediately if the user backgrounds the app or navigates away before the debounce fires.
- **No manual save button, no on-screen "saved"/"failed" banner or modal.** Save status lives in exactly one place: a quiet line in the hamburger menu reading "Synced at {HH:MM}" normally, or "Waiting to sync" / "Hasn't synced in a while" when pending — always in muted grey, never red/amber, never a warning icon. Never block the user or ask them to choose a "mode."

## 5. Common mistakes to avoid

- Don't reintroduce a "Newest first" / "High → Low" style label where the spec calls for lowercase arrow-based wording ("old → new", "high → low") — this was a deliberate, repeated correction.
- Don't add per-goal icons to the Yearly Goals list — that idea was explicitly reversed; only the app/year-level icon is customizable.
- Don't let the "What did I do?" table column wrap or grow the row height — it hard-clips (no ellipsis) at a fixed narrow width, with an "OPEN" button for the full text.
- Don't treat "Short label" as the final field name — it was renamed to **Display label** partway through design; check the data model (§4) for the current field name before referencing it in code.
- Don't build a free-form date/time or color picker where the spec calls for a fixed preset set (goal colors, duration display format) — these are deliberately constrained choices, not open pickers.

## 6. Testing expectations

- Widget tests for interactive components (edit-mode toggles, drag-reorder, popups) covering both the happy path and edge cases called out in the spec (e.g. required deadline field, greyed-out existing years in the "Add a year" picker).
- Don't skip accessibility basics (tap target sizes, contrast) even though the spec doesn't call them out per-screen — they're implied by "usable on mobile" throughout.

## 7. When you're unsure

If the spec or mockups are ambiguous about a visual detail, don't guess silently — state your assumption explicitly in your plan (per Working Agreement rule 1) and let the user confirm or correct it before you implement.
