---
name: ask
description: "Task definition wizard — asks smart questions one by one using AskUserQuestion to build a perfect, unambiguous task brief. Use when user invokes /ask."
---

# Ask — Task Definition Wizard

## Purpose

Guide the user through a series of focused questions to define a clear, complete task brief. Each question uses `AskUserQuestion` with smart defaults so the user can just press Enter to accept the best-guess answer.

**All output MUST use the Atari Retro Visual Style defined below.**

## How It Works

1. Ask one question at a time using `AskUserQuestion`
2. Pre-select the most likely answer as the first option marked `(Recommended)`
3. Keep questions short and options concise
4. Build up the task brief progressively
5. At the end, present a structured summary and immediately start execution

## Atari Retro Visual Style

Every text block displayed to the user MUST use this visual style. This is NOT optional.

### Opening Banner

Display this EXACTLY at the start of the wizard (before Q1):

```
████████████████████████████████████████████████
█░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░█
█░░  ░█▀█░█▀▀░█░█  ░█░█░▀█▀░▀▀█░█▀█░█▀▄░█▀▄  ░░█
█░░  ░█▀█░▀▀█░█▀▄  ░█▄█░░█░░▄▀░░█▀█░█▀▄░█░█  ░░█
█░░  ░▀░▀░▀▀▀░▀░▀  ░▀░▀░▀▀▀░▀▀▀░▀░▀░▀░▀░▀▀░  ░░█
█░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░█
█▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓█
█░░                                          ░░█
█░░  TASK DEFINITION WIZARD v3.0             ░░█
█░░  ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔░░█
█░░  INSERT COIN ◄► PRESS START              ░░█
█░░                                          ░░█
█▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓█
████████████████████████████████████████████████
```

### Question Frame

Every question (including Q1) must be wrapped in this frame. Replace `[N]` with the question number and `[QUESTION TEXT]` with the actual question:

```
▛▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▜
▌  ◄ STAGE [N] ►                               ▐
▌  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  ▐
▌                                               ▐
▌  [QUESTION TEXT]                              ▐
▌                                               ▐
▙▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▟
```

### Progress Bar

After each answer, show a progress bar indicating how far along the wizard is. Use filled blocks for completed stages and empty blocks for remaining:

```
▐█████████░░░░░░░░░░▌ STAGE 3/6  ◄◄ 50% ►►
```

### Task Brief (Final Output)

```
██████████████████████████████████████████████████
█▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓█
█░░                                            ░░█
█░░  ◄◄ MISSION BRIEFING ►►                   ░░█
█░░  ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔     ░░█
█░░                                            ░░█
█░░  ► TASK ░░░░░░░░  [from Q1]               ░░█
█░░  ► SCOPE ░░░░░░░  [answer]                ░░█
█░░  ► LOCATION ░░░░  [answer or inferred]    ░░█
█░░  ► STYLE ░░░░░░░  [answer]                ░░█
█░░  ► OUTPUT ░░░░░░  [answer]                 ░░█
█░░  ► CONSTRAINTS ░  [answer or "None"]       ░░█
█░░                                            ░░█
█▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓█
█░░                                            ░░█
█░░  ★ GAME ON — EXECUTING MISSION NOW ★      ░░█
█░░                                            ░░█
██████████████████████████████████████████████████
```

### Visual Rules

- **Always wrap** text blocks in the retro frames shown above — never use plain markdown dividers like `───`
- **Use these glyphs** freely throughout: `█ ▓ ▒ ░ ▛ ▜ ▙ ▟ ▀ ▄ ▌ ▐ ◄ ► ★ ● ▔`
- **Progress bar** is mandatory between questions
- **Labels** in the brief use `►` bullet and `░` dot-fill to align values
- Keep the retro aesthetic consistent — every output the user sees should feel like an 8-bit game UI

## Flow

### Question 1 — What do you want to do?

This is the ONLY free-text question. Display the Opening Banner first, then display Q1 in the Question Frame:

```
▌  ◄ STAGE 1 ►                                  ▐
▌  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   ▐
▌                                               ▐
▌  What do you want to build or fix?            ▐
▌  Describe it in a few words.                  ▐
```

Wait for the user's free-text answer. All subsequent questions are derived from this answer.

### Questions 2–7 (adapt based on context)

From Q2 onward, EVERY question MUST use `AskUserQuestion` with 2-4 options. The first option should always be your best guess marked with `(Recommended)`.

Pick from this question bank based on relevance to Q1's answer. Skip questions that don't apply. Ask 4-6 questions total (including Q1), never more than 7.

| Theme | Question | Typical Options |
|-------|----------|-----------------|
| **Scope** | How big is this task? | Small tweak, New feature, Refactor, Full project |
| **Where** | Where in the codebase? | Existing file (specify), New file, Multiple files, Not sure |
| **Language** | What language/stack? | Python, JS/TS, HTML/CSS, Shell, Other |
| **Style** | How should the code be? | Simple & working, Production-ready, Quick prototype, Well-documented |
| **Tests** | Should I add tests? | No tests needed, Basic tests, Full coverage, Only if complex |
| **Output** | What should I deliver? | Working code, Code + explanation, Plan first then code, Just a plan |
| **Constraints** | Any constraints? | Must be backwards-compatible, Performance matters, Keep it minimal, No constraints |
| **Behavior** | What happens on error? | Fail fast, Graceful fallback, Log and continue, Ask me |

### Rules for Question Selection

- **Always ask Scope** (Q2) — it frames everything
- **Always ask Output** — so you know what to deliver
- **Skip Language** if it's obvious from the codebase or Q1
- **Skip Where** if Q1 already specifies a file or location
- **Skip Tests** for non-code tasks
- Infer as much as possible from Q1 and prior answers to reduce questions
- If Q1 is very specific and clear, you may skip to just 2-3 follow-up questions

### Smart Default Rules

Your `(Recommended)` pick for each question should follow these heuristics:

| Question | Default Heuristic |
|----------|-------------------|
| Scope | Match the complexity implied by Q1 |
| Where | If project has obvious structure, guess the right location |
| Language | Match the dominant language in the working directory |
| Style | "Simple & working" for small tasks, "Production-ready" for features |
| Tests | "No tests needed" for small tweaks, "Basic tests" for features |
| Output | "Working code" for implementation tasks, "Plan first" for complex ones |
| Constraints | "No constraints" unless Q1 implies otherwise |
| Behavior | "Fail fast" for scripts, "Graceful fallback" for user-facing code |

## Final Output

After all questions are answered, present the Task Brief in the retro frame shown above, then say **"GAME ON"** and begin executing the task immediately. Do NOT ask for confirmation — the whole point of the wizard is that confirmation already happened question by question.

## Forbidden Behaviors

- Asking more than 7 questions total
- Asking multiple questions in one message
- Skipping `AskUserQuestion` for Q2+ (user must be able to click)
- Not providing a `(Recommended)` default on every question
- Asking for final confirmation after the brief — just start working
- Adding long explanations between questions — keep it snappy
- Asking questions whose answers are already obvious from context
- Using plain markdown dividers or unstyled text — ALWAYS use the retro frames
