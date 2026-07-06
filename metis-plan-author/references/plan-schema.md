# Metis plan schema v1 — field reference

The schema is validated with zod at import. Unknown **block types** are
accepted (open union — a newer Metis may render them; older ones show a
graceful placeholder). Malformed _known_ types are rejected; unknown extra
fields on known types are silently stripped — don't rely on them surviving.
Evolve additively; never repurpose a field.

Id convention (from the examples, not enforced): `sk_` skills, `ls_`
lessons, `b_` blocks — short, snake_case, descriptive.

## Plan

```jsonc
{
  "schemaVersion": 1,          // literal 1 — anything else is rejected
  "title": "string, non-empty",
  "skills": [Skill, …]         // ≥1
}
```

## Skill

```jsonc
{
  "id": "sk_unique",           // non-empty
  "title": "string",
  "lessons": [Lesson, …]       // ≥1
}
```

## Lesson

```jsonc
{
  "id": "ls_unique",
  "title": "string",
  "objectives": ["string", …],     // optional, ≤10 — fed to the AI tutor
  "prerequisites": ["string", …],  // optional, ≤10
  "difficulty": "intro" | "intermediate" | "advanced",  // optional
  "blocks": [Block, …]             // ≥1, rendered in order, no gating
}
```

## Block envelope

Every block: `{ "id": string, "type": string }` plus type-specific fields.
`id` must be unique across the entire plan (it keys completions and the XP
ledger). `xp` is a positive integer; it only mints when the block has a
`verify` and the learner passes it.

### prose

```jsonc
{ "id": "b1", "type": "prose", "markdown": "GitHub-flavored markdown" }
```

### code

```jsonc
{ "id": "b2", "type": "code", "language": "typescript", "source": "…" }
```

Syntax-highlighted (shiki). Use real, runnable snippets.

### callout

```jsonc
{ "id": "b3", "type": "callout", "variant": "tip" | "note" | "warning",
  "markdown": "…" }
```

### diagram

```jsonc
{
  "id": "b4",
  "type": "diagram",
  "mermaid": "graph TD; A-->B;",
  "caption": "optional",
}
```

Rendered client-side with mermaid (strict security level). Broken source
degrades to showing the code — but write valid mermaid anyway.

### quiz

```jsonc
{
  "id": "b5", "type": "quiz", "xp": 20,
  "question": "string",
  "options": [ { "id": "a", "text": "…" }, … ],   // ≥2
  "verify": { "method": "answer-key", "answer": ["a"] }  // every entry must be an option id
}
```

`answer` with >1 entries renders as checkboxes (multi-select, exact set
match, order-insensitive). Single entry renders as radios.

### exercise

```jsonc
{
  "id": "b6", "type": "exercise", "xp": 25,
  "instructions": "markdown",
  "placeholder": "optional input hint",
  "verify": <one of the four below>
}
```

| verify                                                | fields                             | notes                                   |
| ----------------------------------------------------- | ---------------------------------- | --------------------------------------- |
| `{ "method": "expected-output", "expected": "…" }`    | exact match after trim + CRLF→LF   | pick machine-stable outputs             |
| `{ "method": "regex", "pattern": "…", "flags": "i" }` | pattern ≤500 chars, flags ⊆ `imsu` | for varying output (hashes, paths)      |
| `{ "method": "artifact-url" }`                        | learner submits an https URL       | evidence tier: mints hard XP, revocable |
| `{ "method": "self-attest" }`                         | one-click honor system             | participation XP only, never spendable  |

### agent-bridge

```jsonc
{
  "id": "b7", "type": "agent-bridge",
  "label": "Stuck on X? Talk it through with your agent.",
  "intent": "tutor" | "explain" | "review" | "debug",   // default "tutor"
  "context": {                                          // optional
    "include": ["lesson"] | ["block:b1", "block:b5"],   // ≤20 refs
    "note": "author steer for the discussion, ≤2000 chars"
  }
}
```

**Must not** carry `xp` or `verify` — the schema rejects it. Metis packages
the referenced blocks (answer keys stripped) and hands off to the learner's
own agent.

## XP routing summary

`hard` (spendable): answer-key, expected-output, regex (deterministic) and
artifact-url (evidence, revocable). `participation` (progress + streak only):
self-attest. Blocks without `verify` mint nothing regardless of `xp`.
