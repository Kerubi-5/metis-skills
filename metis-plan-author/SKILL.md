---
name: metis-plan-author
description: >
  Author HabitPack programs for Metis, an accountability buddy that runs
  daily check-ins, mints XP for attested or evidenced reps, and keeps
  streaks. Use when asked to create a Metis habit program from a goal
  (e.g. pull-ups in 3 months, code every day, learn guitar), review or
  fix HabitPack JSON (schema v2), or convert a practice roadmap into an
  importable Metis pack.
license: MIT
---

# Authoring Metis HabitPacks

Metis is an **accountability buddy**, not a course renderer. You (the agent)
turn a goal into a progressive HabitPack JSON; Metis runs daily check-ins,
XP, streaks, and the shop. Output a single JSON document the user pastes
into Metis's import box (or sends via `import_habit_pack`). Import validates
strictly — schema mistakes are rejected.

Full reference: [references/habit-pack-schema.md](references/habit-pack-schema.md).

Examples:

- Flagship: [examples/pullup-90-day.json](examples/pullup-90-day.json) — “pull-ups in 3 months”
- [examples/coding-daily-90.json](examples/coding-daily-90.json) — daily coding habit
- [examples/guitar-practice.json](examples/guitar-practice.json) — hobby practice

## Shape (schema v2)

```
HabitPack { schemaVersion: 2, title, goal, horizonDays, phases[] }
└─ Phase { id, title, weekStart, weekEnd, habits[] }
   └─ Habit { id, title, cadence: "daily", prompt, notePrompt?, xp?, verify?, coach? }
```

Phases advance by calendar week since pack import (`startedAt`). Today shows
habits in the active phase only.

## Goal → program workflow

1. Clarify the goal and horizon (default 90 days if the user says "3 months").
2. Split into 3–5 progressive phases with non-overlapping week ranges covering the horizon.
3. Put 1–3 **daily** habits per phase — concrete prompts, not vague intentions.
4. Prefer `self-attest` for most reps (participation XP); use `artifact-url` when a photo/video/link proves the rep (hard XP).
5. Deliver one fenced `json` block, no commentary inside.

**Example ask:** "I want to be able to do pull-ups in the next 3 months."
→ Phases: scapular hangs → negatives → assisted → unassisted attempts.

## XP contract

| verify method  | check          | XP track             |
| -------------- | -------------- | -------------------- |
| `artifact-url` | https evidence | **hard** (spendable) |
| `self-attest`  | honor system   | participation only   |
| no `verify`    | guidance only  | none                 |

`coach` is a BYOA doorway (label + `coach`|`review`) — never carries XP.

## Hard validation rules

- `schemaVersion` must be exactly `2`
- habit `id`s unique across the whole pack
- `weekEnd >= weekStart`; weeks 1-indexed
- `cadence` is `"daily"` only (MVP)
- non-empty strings; every phase needs ≥1 habit
- `horizonDays` positive integer (≤ 3650)

## Quality bar

- Progressive overload: later phases should be harder / closer to the goal.
- Prompts are what to do _today_, not essays.
- End most phases with at least one verified habit so progress is felt.
- Include `notePrompt` so session journals feed BYOA reflection.
- Self-check schema rules before handing JSON over.
