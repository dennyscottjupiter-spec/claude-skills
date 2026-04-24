# CLAUDE.md

Guidance for Claude Code when working on this skill.

## What This Is

`self-improving-agent` — a detection-and-capture skill that logs **persistent** errors, corrections, and learnings to `LEARNINGS.md` files. The filter is: **"Will we hit this again?"** — if yes, log it; if no, skip it.

## Integration With the User's LEARNINGS System

This skill must stay aligned with the user's global policy in `~/.claude/CLAUDE.md` → **Error Handling ↔ LEARNINGS.md** section. That section is the authority; this skill is the detection/capture mechanism.

The two-file scheme is non-negotiable:

| File | Scope |
|------|-------|
| `<project_root>/LEARNINGS.md` | Project-specific pitfalls |
| `~/.claude/LEARNINGS.md` | Machine-wide pitfalls (environment, platform, tooling) |

**Never duplicate between them** — entries in both files should be distinct with different scope sentences.

## Entry Format

Fixed and matches the format in `~/.claude/LEARNINGS.md`:

```
## [YYYY-MM-DD] <Short title>
**Tags:** tag1, tag2
**Error:** What happened (1 line)
**Root cause:** Why it will keep happening (1 line)
**Fix/Workaround:** What resolves it each time (1-2 lines)
**Revisit if:** Condition that may invalidate this entry (optional)
```

If the skill's format ever drifts from the policy format, update the skill — not the policy.

## Detection Triggers

The skill fires on: corrections ("that's wrong", "actually..."), tool/command errors, knowledge gaps, or better-approach discoveries. Full list lives in `SKILL.md`.

## Promotion to CLAUDE.md

When a learning is so fundamental it should guide every future session (not just be looked up), distill it into a one-liner rule in `~/.claude/CLAUDE.md` under the relevant section. Mark the LEARNINGS entry with `**Promoted:** CLAUDE.md`.

## Key Conventions

- Frontmatter `description` must start with "Use when..."
- Never write to a third log location — only the two files above
- End-of-session sweep reports counts: *"Added N / Removed N stale / Skipped N (already logged)"*
- Skill must not invent its own folder structure (e.g., `.learnings/`) — prior rewrite removed exactly this drift
