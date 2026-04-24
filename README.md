# claude-skills

A monorepo of custom Claude Code skills built by [@dennyscottjupiter-spec](https://github.com/dennyscottjupiter-spec).

> **GitHub:** https://github.com/dennyscottjupiter-spec/claude-skills

---

## What is a Skill?

A **skill** is a Markdown file that gives Claude Code specialized knowledge and a workflow for a specific task. When you invoke a skill (e.g. `/grill-me`), Claude reads that file and becomes an expert at that task — following the exact steps, rules, and constraints you defined.

Skills are not code plugins. They are **instructions**. The power comes from structuring those instructions well.

---

## How Skills Work in Claude Code

Claude Code loads skills from **two places**, in order of scope:

| Location | Scope | When Claude reads it |
|---|---|---|
| `~/.claude/skills/<skill-name>/SKILL.md` | **Global** — available in every project | When you type `/<skill-name>` anywhere |
| `<project>/.claude/skills/<skill-name>/SKILL.md` | **Project-scoped** — only in that repo | When you type `/<skill-name>` inside that project |

Global skills live in `~/.claude/skills/`. That is where the skills in this repo get **installed**.

---

## Skills in This Repo

| Skill | Invoke with | What it does |
|---|---|---|
| **ask** | `/ask` | Wizard that asks smart questions one-by-one to build a precise, unambiguous task brief. Displays in Atari retro UI. |
| **deep-questioning** | `/deep-questions` | Deep inquiry mode — asks thorough, one-at-a-time questions with lettered options before providing any answer. |
| **grill-me** | `/grill-me` | Plan stress-tester — walks a decision tree depth-first, one question per turn, with a recommendation attached to every question. |
| **pdf-engine** | `/pdf-engine` | Generates polished A4 technical PDFs (guides, SOPs, runbooks, tutorials) from JSON or Markdown via ReportLab. |
| **pension** | `/pension` | Dutch pension advisor with Brazilian comparison. Flags: `--search`, `--calc`, `--scenario`, `--compare`, `--test`. |
| **perplex** | `/perplex` | Perplexity API web search. Generates N complementary queries and synthesizes a token-optimized briefing. Flags: `--N`, `--test`. |
| **self-improving-agent** | auto-triggered | Captures persistent errors, corrections, and knowledge gaps to `LEARNINGS.md`. Only logs things that will recur. |
| **start** | `/start` | Cold-start generator — produces a self-contained resumption prompt to continue work in a fresh conversation. |

---

## Repo Structure

```
claude-skills/
│
├── ask/SKILL.md
├── deep-questioning/SKILL.md
├── grill-me/SKILL.md
├── pdf-engine/
│   ├── SKILL.md
│   ├── CLAUDE.md               <- dev notes for working on this skill
│   ├── scripts/pdf_engine_v4.py
│   └── references/example_content.json
├── pension/SKILL.md
├── perplex/SKILL.md
├── self-improving-agent/SKILL.md
└── start/SKILL.md
```

Each skill folder contains:

| File | Required | Purpose |
|---|---|---|
| `SKILL.md` | Yes | The skill definition. Claude reads this on invocation. |
| `CLAUDE.md` | Recommended | Dev notes for Claude when working *on* the skill itself. |
| `scripts/` | If needed | Supporting scripts the skill tells Claude to run. |
| `references/` | If needed | Reference files, examples, or templates. |

---

## How to Add a New Skill

**Step 1** — Create the folder and write `SKILL.md`:

```markdown
---
name: my-new-skill
description: >
  Use when the user asks to X, Y, or Z. One focused sentence —
  Claude uses this to decide whether to auto-trigger the skill.
---

# My New Skill
...
```

The `description` field is the trigger rule. Be specific — Claude pattern-matches it against user input to decide whether to load the skill.

**Step 2** — Install globally so Claude Code can find it at runtime:

```bash
cp -r my-new-skill ~/.claude/skills/
```

**Step 3** — Test: type `/my-new-skill` in any Claude Code session.

**Step 4** — Commit:

```bash
git add my-new-skill/
git commit -m "feat: add my-new-skill"
git push
```

---

## Design Principles

Discipline-enforcing skills (like `deep-questioning` and `grill-me`) follow a layered defense pattern:

1. **Hard gate** — non-negotiable core rule (e.g. "cannot answer until exit condition is met")
2. **Rationalization table** — maps known ways Claude bends the rules to explicit closures
3. **Red flags list** — self-check triggers Claude can detect in its own draft before outputting
4. **Forbidden behaviors** — explicit loophole closures derived from real failure modes, not hypotheticals

Simpler one-shot skills (`perplex`, `start`) skip most of this. Match the discipline level to the failure surface.

---

## What Does NOT Belong Here

| Thing | Where it belongs |
|---|---|
| Project-scoped skills tied to one repo | `<that-repo>/.claude/skills/` |
| One-off scripts with no `SKILL.md` | The relevant project |
| Third-party skill packs | Separate repo, symlinked if needed |

Rule: **if it does not have a `SKILL.md`, it is not a skill.**
