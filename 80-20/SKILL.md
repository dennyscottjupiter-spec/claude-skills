---
name: "80-20"
description: "Use when the user has a list, backlog, TODO, or plan with >=5 items and says 'what is the 80/20', 'rank these', 'what matters most', 'which first', 'pareto this', or invokes /80-20 — ranks items depth-first with the 'if only one' heuristic and returns the top 20% plus a deferred list for the rest."
---

# 80/20

Given a list, find the 20% that carries 80% of the value. Depth-first scoring, one concrete next step, no fake precision.

Pairs with `/grill-me`: grill-me resolves decisions; `/80-20` chooses which decisions to make first.

## Accepted inputs

- A pasted list (bullets, numbered, comma-separated lines)
- An absolute file path to a markdown TODO
- "The conversation above" — extract the implicit list
- A backlog dump (e.g. `gh issue list`, `gh pr list`)

If the input has fewer than 5 items, respond exactly:

```
List too short (N items). Rank trivially or do them all.
```

If the input has more than 30 items, ask the user once to pre-filter or batch. Do not guess an arbitrary cut.

## The "if only one" heuristic

For each candidate, silently answer: **"If we only did this one, how much of the total value do we keep?"**

Score 1-5:

- **5** — unblocks everything else or is the goal itself
- **4** — high value and independent
- **3** — useful but has substitutes
- **2** — nice-to-have
- **1** — churn, trivia, or duplicate

**Top 20% = items scoring 4-5 that also fit within ~20% of estimated effort.** Adjust only to return **at least 2** and **at most 6** items in the top list.

## Output shape (exact)

```
## Top 20% — do these

1. **<item>** — <one-line why it carries the weight>
2. **<item>** — <one-line why it carries the weight>
...

## Deferred — the other 80%

- <item> — <one-line reason not first>
- <item> — <one-line reason not first>
...

## The call

<one sentence: the single next physical action from the top list>
```

## Optional trailing section

If a load-bearing item is missing from the user's list, add this section after `## The call`:

```
## Missing?

- <item> — <one-line why it belongs in the list>
```

Cap at 3 missing items. Skip the section entirely if nothing critical is missing.

## Rules

- **Depth-first.** Score each item fully before moving to the next; do not re-rank mid-pass.
- **Always include "The call".** The user must leave with one concrete next action, not a ranked list.
- **One line per item.** No paragraphs in the ranking.
- **Deferred items always get a reason.** The user needs to sanity-check what you ranked against.
- **No invented work in the main list.** New items only surface in `## Missing?`.
- **Ties are explicit.** If two items genuinely score 5 and cannot be separated, group them: `1a. X / 1b. Y — parallel, do both first`. Do not fabricate an ordering.

## Forbidden behaviors

- Returning a ranking without "The call"
- Skipping the deferred section
- Inventing items outside the `## Missing?` block
- Running on fewer than 5 items
- Re-ranking mid-pass ("actually, on reflection...")
- Asking the user to score items themselves — the skill does the synthesis
- Writing paragraphs or multi-line explanations inside the ranking
