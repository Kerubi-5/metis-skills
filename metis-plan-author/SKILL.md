---
name: metis-plan-author
description: >
  Author learning plans for Metis, a gamified renderer that verifies every
  rep server-side and mints XP only for machine-checkable work. Use when
  asked to create, review, or fix a Metis plan / learning-plan JSON
  (schema v1), design quizzes or exercises for Metis, or convert a syllabus,
  roadmap, or course outline into an importable Metis plan.
license: MIT
---

# Authoring Metis plans

Metis renders **AI-authored JSON** as an interactive, gamified learning
experience — you (the agent) write the plan, Metis renders and verifies it.
Your output is a single JSON document the user pastes into Metis's import
box. It is validated strictly at import: get the schema right or the import
is rejected with errors.

Full field-by-field reference: [references/plan-schema.md](references/plan-schema.md).
Complete working example: [examples/git-internals-plan.json](examples/git-internals-plan.json).

## Shape (schema v1)

```
Plan { schemaVersion: 1, title, skills[] }
└─ Skill { id, title, lessons[] }
   └─ Lesson { id, title, objectives?, prerequisites?, difficulty?, blocks[] }
      └─ Block { id, type, xp?, verify?, ...type fields }
```

Block types: `prose`, `code`, `callout`, `diagram` (mermaid), `quiz`,
`exercise`, `agent-bridge`. Blocks render in array order; there is no
gating.

## The XP contract (what makes plans motivating)

A block's `verify` method — not its author — decides what its `xp` is worth:

| verify method | check | XP track |
|---|---|---|
| quiz `answer-key` | exact answer set | **hard** (spendable) |
| exercise `expected-output` | normalized string match | **hard** |
| exercise `regex` | pattern match | **hard** |
| exercise `artifact-url` | evidence link, revocable | **hard** |
| exercise `self-attest` | honor system | participation only |
| no `verify` (prose/code/callout/diagram/agent-bridge) | — | none |

Design accordingly: put real XP on machine-checkable reps; use self-attest
sparingly for reading/watching tasks; `agent-bridge` must carry **no xp and
no verify** (a doorway is not a rep).

## Hard validation rules (import fails otherwise)

- `schemaVersion` must be exactly `1`
- block `id`s unique across the **whole plan** (they key the XP ledger)
- every quiz `verify.answer` entry must be one of its `options[].id`
- quiz needs ≥2 options; multi-answer quizzes render as checkboxes
  automatically (say "select all that apply" in the question)
- exercise regex `flags` limited to `imsu`; pattern ≤500 chars
- non-empty strings everywhere; every lesson needs ≥1 block

## Authoring quality bar

- Write for experienced engineers: dense prose, real code, no filler.
- 3–6 blocks per lesson; end most lessons with a verified rep (quiz or
  exercise) worth 20–30 XP so progress is felt.
- Quizzes should test understanding, not recall of the prose's exact words;
  make distractors plausible.
- For `expected-output` exercises, choose commands whose output is stable
  across machines (`git cat-file -t HEAD` → `commit`), or use `regex` when
  output varies (hashes, timestamps).
- Fill `objectives` and `difficulty` on lessons — Metis feeds them to the
  learner's AI tutor for grounded help.
- Self-check against the rules above before handing the JSON over; deliver
  it in a single fenced `json` block with no commentary inside.
