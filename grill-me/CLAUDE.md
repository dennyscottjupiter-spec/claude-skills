# CLAUDE.md

Guidance for Claude Code when working on this skill.

## What This Is

`grill-me` — a discipline-enforcing Socratic skill invoked via `/grill-me`. Stress-tests a plan by walking the **decision tree** one branch at a time, attaching a recommended answer to every question so the user can accept or redirect in one word.

Sister skill to `deep-questioning`: same multi-layer defense pattern, different goal. `deep-questioning` pulls a problem out of the user; `grill-me` pressure-tests a plan the user already has.

## Skill Architecture Pattern

Uses the discipline-enforcing multi-layer defense:

1. **Core protocol** — survey first, enumerate the tree, one question per turn, every question ships a recommendation, depth-first traversal, priority override for critical nodes, explicit exit conditions
2. **Usual-suspects checklist** — goal / scope / constraints / failure modes / rollback / assumption surface — must be probed before declaring done
3. **Decisions Ledger** — the only prose output allowed after exit
4. **Anti-patterns list** — concrete failure modes to avoid
5. **Rationalization table** — common excuses mapped to their reality
6. **Red flags** — self-check triggers for bad behavior

## Key Design Principles

- **Shift synthesis toward the skill, not the user** — every question carries a `**Recommend:**` line so the user can accept with one word. Forcing blank-page answers is a failure mode.
- **5–8 top-level nodes max** — for complex plans, group related choices and enumerate sub-nodes as you traverse. Don't dump 20 questions.
- **Survey before asking** — anything answerable from `Grep`/`Glob`/`Read` never goes to the user. Their turn is for decisions only they can make.
- **Priority override** — if a critical node (failure mode, rollback, hard deadline) would be buried deep in the current branch, pull it forward explicitly.

## Key Conventions

- Frontmatter `description` must start with "Use when..."
- Tree display every turn, even if unchanged
- `AskUserQuestion` for discrete answer spaces; prose for open-ended
- One question per turn is absolute — batching collapses the tree walk
