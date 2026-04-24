---
name: start
description: "Conversation resumption & cold-start skill. Generates a self-contained master prompt that captures ALL context needed to continue work in a fresh conversation. Also works as a generic project handoff generator. Use when user invokes /start."
---

# Start — Conversation Resumption & Cold-Start Generator

## Purpose

Generate a **self-contained master prompt** that allows a brand-new Claude Code conversation to pick up exactly where the current conversation left off — with zero context loss. The output is a single block of text the user can paste into a fresh conversation to resume work immediately.

## When To Use

- User wants to continue complex work in a new conversation
- Context window is getting large and user wants a fresh start
- User wants to hand off work to a different session
- User wants to create a reusable "project brief" for recurring tasks
- User says things like "summarize this for a new conversation", "I need to start fresh", "create a handoff"

## How It Works

### Phase 1: Gather Context (Silent — No Output to User)

Before generating anything, silently gather ALL of the following. Do NOT ask the user for this — extract it from the current conversation and filesystem:

1. **Task Definition**
   - What is the user trying to accomplish? (the goal, not the steps)
   - Why? (motivation, if known)
   - What triggered this work? (bug report, feature request, refactor, etc.)

2. **Current State**
   - What has been done so far in this conversation?
   - What decisions were made and WHY?
   - What was tried and FAILED (so the new conversation doesn't repeat mistakes)?
   - What is the current blocker or stopping point?

3. **File Inventory**
   - Which files are relevant? (full absolute paths)
   - Which files were created or modified in this conversation?
   - Which files should be READ FIRST in the new conversation?

4. **Technical Context**
   - Language/stack involved
   - Key dependencies or tools needed
   - Environment quirks (OS, paths, versions)
   - Any non-obvious configuration (API keys, env vars, special flags)

5. **User Preferences** (from conversation behavior, NOT from memory files)
   - Communication style observed in this conversation
   - Level of autonomy the user expects
   - Any corrections or redirections the user gave

6. **Unfinished Work**
   - Remaining tasks or steps
   - Open questions that still need answers
   - Decisions the user hasn't made yet

### Phase 2: Generate the Master Prompt

Output the master prompt inside a single fenced code block so the user can copy it easily. The prompt MUST follow this exact structure:

````
```
# RESUMPTION PROMPT — [Short Task Title]
# Generated: [YYYY-MM-DD] | Source conversation: [brief identifier]

## GOAL
[1-3 sentences: what we're trying to accomplish and why]

## CONTEXT
[Background the new conversation needs to understand the task.
Include any domain knowledge, constraints, or non-obvious details.]

## WHAT'S BEEN DONE
[Bulleted list of completed steps, decisions made, and their rationale]
- [Step/decision] — because [reason]
- [Step/decision] — because [reason]

## WHAT FAILED (DO NOT REPEAT)
[Bulleted list of approaches that were tried and didn't work]
- [Approach] — failed because [reason]

## CURRENT STATE
[Where we stopped. What's the immediate next action?]
- Status: [in progress / blocked / ready for next phase]
- Blocker: [if any]
- Next action: [specific next step]

## KEY FILES
[Absolute paths to every relevant file, with one-line descriptions]
- `[path]` — [what it is / why it matters]
- `[path]` — [what it is / why it matters]
READ FIRST: `[path]` — [this is the most important file to understand]

## TECHNICAL DETAILS
- Stack: [languages, frameworks]
- Tools: [CLI tools, APIs, services involved]
- Environment: [OS, shell, any quirks]
- Config: [env vars needed, special setup]

## REMAINING WORK
[Ordered list of what still needs to be done]
1. [Task] — [brief detail]
2. [Task] — [brief detail]
3. [Task] — [brief detail]

## OPEN QUESTIONS
[Decisions not yet made, things to clarify]
- [Question]
- [Question]

## HOW TO WORK WITH ME
[User preferences observed in this conversation]
- [Preference — e.g., "Ask before web searches", "Use /ask methodology for complex tasks"]
- [Preference]

## INSTRUCTIONS
[Specific instructions for the new conversation. Be directive.]
1. Start by reading [specific file(s)]
2. Then [specific action]
3. [Any methodology to follow — e.g., "Use /ask to gather requirements before coding"]
```
````

### Phase 3: Present to User

After the code block, add a brief note:

```
▛▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▜
▌  ◄ READY TO GO ►                              ▐
▌  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   ▐
▌                                                ▐
▌  Copy the block above and paste it as your     ▐
▌  first message in a new conversation.          ▐
▌                                                ▐
▌  Files to read: [N] | Tasks remaining: [N]     ▐
▌                                                ▐
▙▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▟
```

## Quality Rules

1. **Self-contained** — The new conversation must need ZERO additional context. Everything is in the prompt.
2. **No assumptions** — Don't assume the new conversation will have access to memory, CLAUDE.md, or any prior state. Include everything explicitly.
3. **Actionable** — The new conversation should be able to start working immediately after reading the prompt.
4. **Honest about unknowns** — If something wasn't decided, say so. Don't fill gaps with guesses.
5. **Compact but complete** — Every line must earn its place. No filler. But don't sacrifice completeness for brevity.
6. **Preserve failures** — The "WHAT FAILED" section is critical. It prevents the new conversation from wasting time on dead ends.
7. **Absolute paths only** — Never use relative paths. The new conversation may have a different working directory.
8. **Include the WHY** — For every decision listed, include why it was made. Decisions without rationale are useless to a fresh context.

## Arguments

The skill accepts an optional argument to customize the output:

| Argument | Effect |
|----------|--------|
| (none) | Generate resumption prompt for current conversation's work |
| `--brief` | Shorter version — just GOAL, CURRENT STATE, KEY FILES, REMAINING WORK |
| `--handoff` | Include extra onboarding context as if handing to a different person |
| `--reusable` | Strip conversation-specific state, keep only the reusable project brief |

## Edge Cases

- **Nothing was done yet:** If the conversation only discussed/planned but didn't execute, focus the prompt on the plan and decisions made.
- **Multiple tasks:** If the conversation covered multiple unrelated tasks, generate separate resumption prompts for each (ask which ones the user wants).
- **No failures:** If nothing failed, omit the "WHAT FAILED" section entirely rather than writing "None".
- **User provides extra context:** If the user says "/start [extra details]", incorporate those details into the appropriate sections of the prompt.

## Forbidden Behaviors

- Generating a prompt that requires reading other prompts or files to understand
- Including memory file contents (the new conversation will load its own memories)
- Being vague — "continue the work" is not an instruction; "read SKILL.md at line 45 and fix the hardcoded EUR amount" is
- Adding explanations outside the code block — the user just needs the copyable prompt
- Asking clarifying questions — extract everything from context; only ask if truly ambiguous
