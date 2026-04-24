---
name: grill-me
description: Use when user says "grill me", "stress-test this", "poke holes", "interview me", or invokes /grill-me — relentlessly interviews the user about a plan or design, walking the decision tree one branch at a time, with a recommended answer attached to every question.
---

# Grill Me

Pressure-test a plan until every consequential decision is explicitly resolved. Walk the **decision tree** one branch at a time. Supply a recommended answer with every question so the user can accept or redirect in one word.

**Decision tree:** the set of open choices a plan implies, where some choices only become relevant once earlier ones are resolved (child nodes).

## When to fire

- User says "grill me", "stress-test this", "poke holes in this", "interview me", "pressure-test it"
- User invokes `/grill-me`
- User presents a plan or design and asks for scrutiny before committing
- A spec has ambiguity that would otherwise force arbitrary choices mid-implementation

## Core protocol

### 1. Survey first, ask second

Before the first question, explore the codebase for anything already answerable from existing code — naming conventions, existing abstractions, test patterns, folder layout, dependency versions. **If the repo answers it, you answer it.** Do not spend a user turn on a lookup.

Use `Grep`, `Glob`, `Read` liberally up front. The user's turn is for decisions only they can make.

### 2. Enumerate the tree out loud

First message after invocation: list every open decision you can identify from the plan, numbered. As answers land, child nodes will appear — add them. Show the tree each turn so neither side loses the thread:

```
Open
  [ ] 1. Storage — SQLite vs Postgres
  [ ] 2. Auth model (depends on 1)
  [ ] 3. Behavior if migration aborts mid-run
Resolved
  [x] 0. Single-tenant only → yes
```

### 3. One question per turn

Never batch. One question, one answer, re-plan, next question. Batching collapses the tree walk and leaves branches half-resolved.

### 4. Every question ships with a recommendation

Use this exact shape:

```
**Q[n]:** <one-sentence question>
**Recommend:** <your best-guess answer>
**Why:** <one line of reasoning>
**Unblocks:** <child nodes that open if the recommendation is taken>
```

The user should be able to say "yes" and move on. Forcing blank-page answers is a failure mode — the whole point of this skill is to shift synthesis *toward* you, not away.

If the answer space is **discrete**, call `AskUserQuestion` with your recommendation highlighted among the options. If the answer space is **open-ended** (numbers, names, architectural shape), use prose and invite a free reply.

### 5. Depth-first traversal

Walk one branch to its leaves before starting another. Jumping between branches forces the user to rebuild mental state each turn.

Exception: if answering node A reveals that node B is load-bearing and blocks further progress on A, pivot to B — but name the pivot explicitly.

### 6. Cover the usual suspects

Before you declare done, make sure you've probed each of these for the plan at hand:

- **Goal** — what does success look like, verifiable how?
- **Scope** — what's explicitly *out*?
- **Constraints** — budget, deadline, compatibility, data shape, perf budget
- **Failure modes** — what breaks, and how loudly?
- **Rollback** — can we undo, how fast, what state survives?
- **Assumption surface** — what is the plan taking on faith that hasn't been verified?

Not every plan needs all six. Skip the ones that are genuinely N/A — but skip knowingly, not by forgetting.

### 7. Exit conditions

Stop when **any** of these hold:

- Tree is fully resolved
- User says "enough" / "stop" / "ship it" / "we're good" / equivalent
- Remaining questions are trivia the user explicitly defers
- Answers are visibly shortening → ask once: **"keep grilling or wrap?"** and honor the reply

On exit, produce the **Decisions Ledger**.

## Decisions Ledger (final artifact)

The only prose output allowed after exit. Hand this to implementation:

```
## Decisions Ledger

1. <Decision> → **<Answer>** — <one-line rationale>
2. <Decision> → **<Answer>** — <one-line rationale>
...

Deferred
- <Question> — <why it was deferred, when to revisit>

Open assumptions
- <Thing the plan depends on that was not verified during grilling>
```

## Anti-patterns

- ❌ **Detached hypotheticals** — "what if you had a million users?" matters only if million-user scale is in scope for this plan
- ❌ **Asking what the code already answers** — `grep` first, ask second
- ❌ **Multi-part questions** — "and also, what about X, Y, Z?" is three turns, not one
- ❌ **Questions without a recommendation** — if you can't form one, say so explicitly and offer two concrete alternatives with tradeoffs
- ❌ **Drifting across branches** — finish the branch you're on before opening another
- ❌ **Continuing past exhaustion** — if answers shorten, check in rather than push
- ❌ **Re-litigating resolved branches** — once `[x]`, leave it alone unless a later answer directly contradicts it
- ❌ **Bundling grilling with implementation** — this skill does not write code. If implementation is clear post-ledger, hand off and stop.

## Rationalization table

| Excuse | Reality |
|---|---|
| "I'll batch these two questions, they're related" | Batching collapses the tree walk. Split them. |
| "I don't have a recommendation — just ask" | Form one. If genuinely split, state the split explicitly and offer two alternatives with tradeoffs. |
| "The user can tell me how the codebase is structured" | No. Read it. Their turn is for decisions, not lookups. |
| "Let me cover this whole branch in one big question" | Walk the branch depth-first, one node per turn. |
| "They said 'grill me' but seem annoyed now — I'll wrap early" | Don't guess. Ask explicitly: "keep going or wrap?" |
| "I'll give a quick overview of the options while I'm at it" | That's an answer. Ask only. |
| "This question has 5 discrete options, I'll just list them in prose" | Use `AskUserQuestion`. Clicking beats typing. |

## Red flags — stop if you notice these

- You're writing "and also..." into a question
- You're about to ask about something answerable by `grep` or `Read`
- Your question has no `**Recommend:**` line attached
- You're jumping branches instead of finishing one
- You're asking about scenarios that aren't in scope for the actual plan
- You're still grilling after the user has said "enough"
- You're tempted to skip the tree display this turn "because it hasn't changed much" — show it anyway

**If any of these fire: delete the draft, re-open the tree, ask one question with a recommendation.**
