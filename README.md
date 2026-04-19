# claude-skills

A monorepo of custom Claude Code skills built by [@dennyscottjupiter-spec](https://github.com/dennyscottjupiter-spec).

> **GitHub:** https://github.com/dennyscottjupiter-spec/claude-skills

---

## What is a Skill?

A **skill** is a Markdown file that gives Claude Code specialized knowledge and a workflow for a specific task. When you invoke a skill (e.g. `/pdf-engine`), Claude reads that file and becomes an expert at that task — following the exact steps, rules, and constraints you defined.

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

## This Repo's Structure

```
claude-skills/               ← This repo (development & source of truth)
│
└── pdf-engine/              ← One folder per skill
    ├── SKILL.md             ← The skill definition Claude Code reads
    ├── CLAUDE.md            ← Dev notes: architecture, how to modify the engine
    ├── scripts/
    │   └── pdf_engine_v4.py ← Supporting Python script called by the skill
    └── references/
        └── example_content.json
```

Each skill folder contains:

| File | Required | Purpose |
|---|---|---|
| `SKILL.md` | ✅ Yes | The skill definition. Claude reads this when the skill is invoked. |
| `CLAUDE.md` | Recommended | Dev notes for Claude when working *on* the skill itself (architecture, rules, how to modify). |
| `scripts/` | If needed | Supporting scripts the skill tells Claude to run. |
| `references/` | If needed | Reference files, examples, templates the skill can instruct Claude to read. |

---

## Currently Available Skills

| Skill | Invoke with | What it does |
|---|---|---|
| **pdf-engine** | `/pdf-engine` | Generates polished A4 technical PDFs from JSON or Markdown. Powered by `pdf_engine_v4.py` (ReportLab). |

---

## Skills Installed Globally (not in this repo)

These skills live directly in `~/.claude/skills/` and were not developed as standalone repos. They are managed in-place.

| Skill | Invoke with | What it does |
|---|---|---|
| **ask** | `/ask` | Wizard that asks smart questions one-by-one to build a precise task brief. |
| **deep-questioning** | `/deep-questions` | Enters deep inquiry mode — asks thorough questions before answering anything. |
| **pension** | `/pension` | Dutch pension system advisor with Brazilian comparison. Calls Perplexity AI with `--search`. |
| **perplex** | `/perplex` | Web search via Perplexity API. Flags: `--N`, `--test`. |
| **self-improving-agent** | auto-triggered | Captures errors and corrections to improve future sessions. |
| **start** | `/start` | Generates a self-contained handoff prompt to resume work in a fresh conversation. |

---

## How to Add a New Skill to This Repo

### Step 1 — Create the folder

```
claude-skills/
└── my-new-skill/
    ├── SKILL.md      ← write this first
    └── CLAUDE.md     ← optional but recommended
```

### Step 2 — Write `SKILL.md`

Every `SKILL.md` must start with a YAML frontmatter block:

```markdown
---
name: my-new-skill
description: >
  One sentence describing what this skill does and when Claude should use it.
  Be specific — this is what Claude uses to decide whether to auto-trigger the skill.
---

# My New Skill

## When to use this skill
...

## Step-by-step instructions
...
```

The `description` field is critical. Claude uses it to auto-detect when the skill is relevant. Write it as a trigger rule: "Use this skill when the user asks to X, Y, or Z."

### Step 3 — Install it globally

Symlink or copy the `SKILL.md` into `~/.claude/skills/`:

```bash
# Option A: symlink (changes here reflect immediately — recommended)
mkdir -p ~/.claude/skills/my-new-skill
ln -s ~/Claudinho/SKILLS/my-new-skill/SKILL.md ~/.claude/skills/my-new-skill/SKILL.md

# Option B: copy (static snapshot)
cp -r ~/Claudinho/SKILLS/my-new-skill ~/.claude/skills/
```

### Step 4 — Test it

```
/my-new-skill hello world
```

Claude should pick up the skill and follow its instructions.

### Step 5 — Commit and push

```bash
cd ~/Claudinho/SKILLS
git add my-new-skill/
git commit -m "Add my-new-skill"
git push
```

---

## What Does NOT Belong Here

| Thing | Where it belongs instead |
|---|---|
| Third-party skill packs (e.g. `marketingskills` by Corey Haines) | `~/Claudinho/Local-Repos/` |
| Project-scoped skills tied to one repo | `<that-repo>/.claude/skills/` |
| Skill reference docs / planning notes (not the skill itself) | `~/Claudinho/Local-Repos/<skill>-docs/` |
| One-off scripts with no SKILL.md | `~/Claudinho/Local-Repos/` or the relevant project |

The rule of thumb: **if it doesn't have a `SKILL.md`, it is not a skill.**

---

## Relationship Between This Repo and `~/.claude/`

```
~/Claudinho/SKILLS/          ← Source of truth (this repo, versioned on GitHub)
        │
        │  skill is developed and tested here
        │
        ▼
~/.claude/skills/            ← Where Claude Code looks at runtime
        │
        │  install via symlink or copy
        │
        ▼
  Claude reads SKILL.md when you type /<skill-name>
```

Keeping source in `~/Claudinho/SKILLS/` means your skills are version-controlled and backed up on GitHub, while `~/.claude/skills/` is the runtime location Claude Code reads from.

---

## Useful Commands

```bash
# List all globally installed skills
ls ~/.claude/skills/

# List skills in this repo
ls ~/Claudinho/SKILLS/

# See what a skill does
cat ~/.claude/skills/pdf-engine/SKILL.md

# Run the pdf-engine directly (for testing)
python3 ~/Claudinho/SKILLS/pdf-engine/scripts/pdf_engine_v4.py input.json -o output.pdf
```
