---
name: brief
description: "Use when the user wants a long document, PR, transcript, article, or thread compressed, says 'brief me on this', 'tl;dr this', 'summarize concisely', or invokes /brief — returns a fixed 4-section briefing (TL;DR / 3 Decisions / Risks / Next Action) with zero narrative."
---

# Brief

Compress any long input into a fixed 4-section briefing. No narrative, no "let me explain" — just the sections.

## Accepted inputs

- An absolute file path
- A URL (use `WebFetch`)
- Pasted text
- "The conversation above" (summarize the current conversation)
- A PR number (use `gh pr view <N>` + `gh pr diff <N>`)
- An issue number (use `gh issue view <N>`)

If the input is ambiguous, pick the most recent content the user mentioned and proceed. Do not ask.

## Output shape (exact, no deviations)

```
## TL;DR
<line 1 — what this is>
<line 2 — why it matters>

## 3 Decisions
1. <decision> — <one-line rationale>
2. <decision> — <one-line rationale>
3. <decision> — <one-line rationale>

## Risks
- <risk> — <likelihood / blast radius in <=6 words>
- <risk> — <likelihood / blast radius in <=6 words>

## Next Action
<one sentence: the single next physical step>
```

## Rules

- **Fixed structure.** Four sections, exactly as shown. No additions, no omissions, no reordering.
- **TL;DR = 2 lines.** Not 2 sentences — 2 lines. Enforce.
- **Exactly 3 Decisions.** If the document has fewer, pad with "Implicit: <decision the doc assumes>". If it has more, merge the lowest-impact ones.
- **Risks = 2-4 bullets.** If the content is genuinely risk-free (reference doc, changelog), write `- None surfaced` as the sole bullet.
- **Next Action is physical.** "run X", "reply to Y", "open file Z" — not "consider", not "think about", not "discuss".
- **No citations, no quotes.** Compress, do not paraphrase.

## When the input is too thin

If the input is under ~50 words or contains no decisions at all, do not invent structure. Respond with exactly:

```
Too thin to brief. Content is: <one-line summary>.
```

## Forbidden behaviors

- Adding sections (Background, Context, Glossary, Methodology, etc.)
- Writing narrative paragraphs before or after the briefing
- Exceeding 2 lines in TL;DR
- Fewer or more than 3 Decisions
- Vague Next Action ("consider X", "think about Y", "discuss Z")
- Asking clarifying questions before briefing — the whole point is zero ceremony
- Explaining what you did after the briefing ("Hope this helps", "Let me know if...")
