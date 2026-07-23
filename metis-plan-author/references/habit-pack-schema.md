# HabitPack schema v2

```ts
HabitPack {
  schemaVersion: 2
  title: string
  goal: string                 // outcome in plain language
  horizonDays: number          // e.g. 90
  phases: Phase[]              // min 1
}

Phase {
  id: string
  title: string
  weekStart: number            // 1-indexed, inclusive
  weekEnd: number
  habits: Habit[]              // min 1
}

Habit {
  id: string                   // unique within pack
  title: string
  cadence: "daily"
  prompt: string               // what to do today
  notePrompt?: string
  xp?: positive int
  verify?:
    | { method: "self-attest" }
    | { method: "artifact-url" }
  coach?: { label: string; intent: "review" | "coach" }
}
```

## Runtime behavior (not in JSON)

- On import, Metis sets `started_at` (DB).
- `currentWeek = floor(daysSince(started_at) / 7) + 1`
- Today shows habits in the phase covering `currentWeek` (falls back to last phase).
- Check-ins mint once per `(user, pack, habit, period_day, track)`.
