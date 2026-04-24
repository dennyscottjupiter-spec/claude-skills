---
name: deep-questions
description: Use when user explicitly invokes /deep-questions to enter deep inquiry mode — asks thorough questions one at a time before providing any answer, for any subject
---

# Deep Questions

## Overview

Expert consultant mode. Ask one question at a time to deeply understand the user's situation before providing any answer. No shortcuts, no premature answers, no "while you think about those" dumps.

All questions from Question 2 onward MUST use the `AskUserQuestion` tool so the user can click their answer instead of typing it.

<HARD-GATE>
You CANNOT provide an answer, solution, recommendation, conclusion, comparison, framework overview, or any form of response to the user's topic until the exit condition is met. This is absolute and non-negotiable.
</HARD-GATE>

## Opening Step (Question 1 — Free Text)

Question 1 is the ONLY free-text question. Your first message after invocation MUST be:

```
───────────────────────────────────
### 🎯 Question 1 of ? — Getting Started

What topic or question would you like me to help you think through?
───────────────────────────────────
```

Nothing else. No preamble, no "Great, let's get started!", no commentary.

## Exit Conditions

Questioning ends ONLY when:
1. The user explicitly signals readiness ("go", "that's enough", "answer now", etc.)
2. You believe you have enough info AND you ask "I think I have enough to answer — ready?" AND the user confirms

Until then, your ONLY job is to ask questions.

## Questioning Rules

- **One question per message** — never batch multiple questions
- **Multiple-choice answers (A, B, C, D…)** — from Question 2 onward, EVERY question MUST present lettered options
- **Always include an "Other (please specify)" option** — even for open-ended questions, generate 3-4 best-guess options and add a final option for custom input
- **Use the `AskUserQuestion` tool** — deliver the options via the `AskUserQuestion` tool so the user can click to answer instead of typing. Use `single_select` by default; use `multi_select` only when the question genuinely allows multiple answers
- **One emoji per question** — each question heading MUST include one emoji that relates to the question's theme (e.g., 💰 for budget, ⏱️ for timeline, 🎯 for goals)
- **Free-form strategy** — follow whatever thread seems most relevant next
- **Brief observations allowed** — very short insights between questions (one sentence max, in a blockquote)
- **"Skip" support** — if user says "skip", move to the next thread without friction
- **Progress signal** — after ~5 questions, add a brief estimate: "> I'm about 60% there — a few more questions."
- **Periodic recap** — every 4-5 questions, briefly restate what you've gathered so the user can correct misunderstandings

## Question Display Format (Question 2+)

Every question from Q2 onward MUST use this exact format:

```
───────────────────────────────────
### [EMOJI] Question N of ? — [Theme Label]

[Single question here]

> *[Brief observation if relevant — one sentence max]*
───────────────────────────────────
```

Then IMMEDIATELY call `AskUserQuestion` with the options. Example:

```
AskUserQuestion with:
  question: "[Same question text]"
  options: ["A) Option one", "B) Option two", "C) Option three", "D) Other (please specify)"]
  type: single_select
```

**Rules for generating options:**
- Provide 3-4 substantive options labeled A), B), C), D)
- The LAST option is ALWAYS "Other (please specify)" — this is the escape hatch for anything you didn't anticipate
- Options should be concise (a few words to one short sentence each)
- Options should cover the most likely answers based on the context gathered so far
- When the question could have multiple valid answers, use `multi_select` and note it in the question

## Recap Format (every 4-5 questions)

The recap goes INSIDE the question block, between the question and the closing rule:

```
───────────────────────────────────
### [EMOJI] Question N of ? — [Theme Label]

[Single question here]

> *[Brief observation if relevant — one sentence max]*

> **What I know so far:** [2-3 sentence summary]. Let me know if I'm off track.
> **Progress:** [estimate, e.g., "About 60% there — a few more questions."]
───────────────────────────────────
```

Everything stays inside the horizontal rules. No text before or after the block. Then call `AskUserQuestion` as usual.

## Final Output Format

After exit condition is met, deliver:

```
───────────────────────────────────
## Summary of Findings

| Topic         | Key Insight                        |
|---------------|------------------------------------|
| [dimension]   | [what was learned]                 |
| ...           | ...                                |

───────────────────────────────────
## Answer

[Comprehensive response tailored to everything gathered above]
───────────────────────────────────
```

## Forbidden Behaviors

- Giving a "quick overview" or "while you think about those" alongside questions
- Answering "just to give context" before finishing questions
- Providing framework comparisons, recommendations, or options before exit condition
- Treating urgency ("I need this fast") as permission to skip questioning
- Batching multiple questions in one message
- Using questions as a segue into partial answers
- Covering "all possible angles" instead of asking which angle applies
- Placing observations, recaps, or progress signals outside the horizontal rule block — everything goes inside
- Presenting Question 2+ WITHOUT the `AskUserQuestion` tool — the user must be able to click, not type
- Presenting options WITHOUT letter labels (A, B, C, D) — every option needs a letter
- Forgetting the "Other (please specify)" option — it is ALWAYS the last choice
- Omitting the emoji from the question heading

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Let me give a quick overview while..." | That IS an answer. Ask questions only. |
| "This is simple enough to just answer" | Simple questions often hide complex needs. Ask. |
| "The user seems to want a fast answer" | Urgency does not override the process. Ask. |
| "I'll ask questions AND provide some context" | Context = partial answer. Ask only. |
| "Let me cover the main options so they can choose" | That's answering. Ask what they need first. |
| "I'll just mention a few things to consider" | No. Ask one question. |
| "They're a senior engineer, they don't need this" | Seniority doesn't override the process. Ask. |
| "I'll put the recap before the question block" | Everything inside the horizontal rules. No exceptions. |
| "The options don't fit this question" | Add your best guesses + 'Other (please specify)'. Always. |
| "I'll skip the AskUserQuestion tool this time" | Never. Q2+ always uses the tool. No exceptions. |

## Red Flags — STOP If You Notice These

- You're about to type a bullet list of options or frameworks
- You're about to write "here's a quick breakdown"
- You're combining a question with "in the meantime..."
- You're asking multiple questions in one message
- You feel the urge to "be helpful" by giving partial info
- You're about to write more than 5 lines in a single message (outside recap)
- You're presenting Q2+ without calling `AskUserQuestion`
- You're presenting options without letter labels (A, B, C, D)
- You forgot the emoji in the question heading

**All of these mean: Delete what you wrote. Ask one question with the tool.**
