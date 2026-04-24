---
name: handoff
description: "Use when the session is ending, the user says 'handoff', 'wrap up', 'save progress', 'write a handoff', or invokes /handoff — writes a <=120-word paragraph capturing where work stopped, the next physical action, and the gotcha, then auto-saves to ~/.claude/handoff/YYYY-MM-DD.md for /start to consume."
---

# Handoff

Generate a <=120-word paragraph that lets a fresh session resume work in one read. Auto-save it. That is the entire skill.

Sister to `/start` (which generates a full resumption prompt). `/handoff` is the lighter, daily version — zero ceremony, single paragraph, saved to disk.

## Output shape (fixed, <=120 words total)

One paragraph with three beats, in this order, separated by normal sentence punctuation (no bullets, no headings):

1. **Where I stopped.** The last concrete thing that moved forward. Include the absolute file path if code was involved.
2. **Next physical action.** The literal next thing to do at keystroke level (e.g. "run `pytest tests/test_auth.py::test_login`", "edit `scan_result.py:42` to add a `freshness` column"). Not a goal — a command or a line-level edit.
3. **The gotcha.** The non-obvious thing the next session will trip over if nobody tells them (a failed approach, a flag that must be set, a platform quirk). If there is no gotcha, say so explicitly in one sentence — do not invent one.

No preamble, no numbered list, no headings in the paragraph itself.

## Save step

Write the paragraph to `~/.claude/handoff/YYYY-MM-DD.md` using today's date from the session's `currentDate`.

- If the directory does not exist, create it.
- If the file already exists (multiple handoffs same day), append with a divider:

```
---
## HH:MM
<paragraph>
```

After saving, print the paragraph to chat plus one final line:

```
Saved -> ~/.claude/handoff/YYYY-MM-DD.md (N words)
```

## When the session had no meaningful work

If the conversation produced no concrete progress (only planning, only Q&A, nothing saved or changed), do not fabricate a handoff. Write exactly:

```
Nothing to hand off — this session only <explored / discussed / planned>. Next session can start fresh.
```

Save that line anyway so the date is recorded, and print the same confirmation footer.

## Rules

- **<=120 words.** Count. If over, cut. Aim for 90-120.
- **Three beats only.** Stop, next, gotcha. No "what we explored", no "summary of approaches".
- **Keystroke-level next action.** "Continue the work" is banned. "Run X", "edit file Y line Z", "read doc at path" — that level.
- **Absolute paths.** The next session may not know the working directory.
- **Facts only.** No "we made great progress", no "hopefully this helps".
- **Extract from conversation state.** Do not ask the user what to write.

## Forbidden behaviors

- Asking the user what to put in the handoff
- Exceeding 120 words
- Multiple paragraphs or bullet points inside the paragraph
- Inventing a gotcha when none exists
- Using relative paths
- Printing to chat without saving, or saving without printing
- Adding extra sections beyond the paragraph + save confirmation line
