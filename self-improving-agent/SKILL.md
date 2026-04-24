---
name: self-improving-agent
description: "Captures learnings, errors, and corrections to enable continuous improvement. Use when: (1) A command or operation fails unexpectedly, (2) User corrects Claude ('No, that's wrong...', 'Actually...'), (3) User requests a capability that doesn't exist, (4) An external API or tool fails, (5) Claude realizes its knowledge is outdated or incorrect, (6) A better approach is discovered for a recurring task. Also review learnings before major tasks."
---

# Self-Improving Agent

Log **persistent** errors, corrections, and insights to LEARNINGS.md files. The filter is: **"Will we hit this again?"** — if yes, log it; if no, skip it.

## Two Files, Always

Every entry goes to BOTH files (different scope — never duplicate content between them):

| File | What belongs there |
|------|--------------------|
| `<project_root>/LEARNINGS.md` | Project-specific pitfalls, conventions, quirks |
| `~/.claude/LEARNINGS.md` | Machine-wide pitfalls: environment, platform, tooling |

If `LEARNINGS.md` doesn't exist in the project root, create it with this header:
```
# LEARNINGS

Project-specific pitfalls and workarounds.
```

## Entry Format

```
## [YYYY-MM-DD] <Short title>
**Tags:** tag1, tag2
**Error:** What happened (1 line)
**Root cause:** Why it will keep happening (1 line)
**Fix/Workaround:** What resolves it each time (1-2 lines)
**Revisit if:** Condition that may invalidate this entry (optional)
```

**Tag vocabulary:** `windows`, `python`, `encoding`, `path`, `rtk`, `git`, `api`, `mcp`, `tool`, `skill`, `environment`, `network`, `shell`

**Where each entry lives:**
- Environment quirks, platform behavior, CLI tool issues, encoding problems → `~/.claude/LEARNINGS.md`
- Project conventions, project-specific commands, local schema/API behavior → `<project_root>/LEARNINGS.md`
- When in doubt, write to both with different scope sentences

## Detection Triggers

Log **immediately** (before the conversation continues) when you observe:

**Corrections:**
- "No, that's not right..." / "Actually..." / "That's outdated..." / "You're wrong about..."
- User provides information Claude didn't have or had wrong

**Errors:**
- Command returns non-zero exit code or unexpected output
- Exception, stack trace, or timeout
- Tool or API returns an error response

**Knowledge gaps:**
- API behaves differently than expected
- Documentation Claude referenced is outdated
- Platform behavior differs from what Claude assumed

**Better approaches:**
- Simpler/faster method found after trying a harder one
- Cleaner pattern emerged during the task
- Workaround discovered for a recurring platform issue

**What NOT to log:**
- One-time typos or logic bugs fixed in code
- Things already in `CLAUDE.md`
- Ephemeral state from this session only

## Session-Start Review

At the start of any major task: scan the relevant `LEARNINGS.md` entries. If any apply to the current task, mention them before proceeding.

## End-of-Session Sweep

Before closing a session where persistent errors were resolved:
1. Check if each resolved error is already logged
2. Add entries for anything not yet captured
3. Remove entries whose `Revisit if:` condition has been met
4. Report: *"Added N / Removed N stale / Skipped N (already logged)"*

## Promotion to CLAUDE.md

When a learning is so fundamental it should guide ALL future sessions (not just be looked up), distill it into a one-liner rule and add it to `~/.claude/CLAUDE.md` under the relevant section. Mark the LEARNINGS entry with `**Promoted:** CLAUDE.md`.

Examples of promotion-worthy learnings:
- Platform encoding behavior that affects every Python script
- A tool invocation pattern that's always needed (e.g., RTK prefix rule)
- A shell behavior on this machine that differs from defaults

## Priority Guidelines

| Priority | When to use |
|----------|-------------|
| critical | Blocks core functionality, data loss risk, security issue |
| high | Affects common workflows, recurring issue |
| medium | Workaround exists, moderate impact |
| low | Minor edge case, nice-to-know |

Add `**Priority:** high` (or other level) between Tags and Error lines for anything above medium.
