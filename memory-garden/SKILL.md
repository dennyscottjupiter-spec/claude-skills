---
name: memory-garden
description: "Use when the user invokes /memory-garden, says 'clean up memory', 'prune memories', 'dedupe my memory', or when ~/.claude/memory/ has grown past ~30 files — scans the memory directory, detects orphans, broken index pointers, near-duplicates, and stale references, then prunes/merges/flags after user confirmation."
---

# Memory Garden

Keep `~/.claude/memory/` clean. Scan, detect rot, confirm before changing anything.

## Scope

The garden is `~/.claude/memory/` and its `MEMORY.md` index. Never touch files outside this directory.

## Phase 1 — Silent scan

Read every `*.md` file in the memory directory (exclude `MEMORY.md` itself). Parse frontmatter for `name`, `description`, `type`. Read `MEMORY.md` and extract its index links (`- [Title](file.md) — hook`).

If `MEMORY.md` does not exist, print `MEMORY.md missing — treating every file as an orphan.` and proceed with an empty index. The rest of the flow stays the same.

Build four buckets:

- **Orphans** — memory files with no entry in `MEMORY.md`
- **Broken pointers** — `MEMORY.md` links to a file that does not exist
- **Near-duplicates** — two files with the same `type` AND descriptions that overlap on subject (same topic, different wording)
- **Stale references** — the memory body names a concrete artifact that currently does not exist

Stale check rule (narrow, to avoid false positives):

- Only flag absolute paths (`C:/Users/...`, `/c/Users/...`, `~/...`), skill names (`/<slash-command>`), and artifacts inside `~/.claude/` (files, skills, memory entries).
- Verify with a direct filesystem check (`ls`/`Read`/`Glob`), not a codebase grep.
- Generic statements ("user prefers Python", "works on Windows") are never stale.

## Phase 2 — Report first (always)

Print the scorecard before any file operation:

```
## Memory Garden — scan report

Files:   N        Index entries: M

Orphans (N)
  - <file> — <description>

Broken pointers (N)
  - MEMORY.md -> <file> (missing)

Near-duplicates (N)
  - <file-a> ~= <file-b> — <shared subject>

Stale references (N)
  - <file>: mentions <artifact> -> not found
```

If all four counts are zero, print `Memory garden is clean — N files, M index entries.` and stop.

If the directory has fewer than 5 memory files, print `Nothing to garden — only N files.` and stop.

## Phase 3 — Confirm each non-empty bucket

For every bucket with findings, use `AskUserQuestion` (single_select) with these three options:

- "Apply all (recommended action)"
- "Review one-by-one"
- "Skip this bucket"

Per-bucket recommended action:

- **Orphans** -> add N index entries (use each memory's `description` as the hook)
- **Broken pointers** -> remove N dead lines from `MEMORY.md`
- **Near-duplicates** -> merge each pair (winner = richer body; loser file moves to trash; `MEMORY.md` updated)
- **Stale references** -> prepend `**Stale:** YYYY-MM-DD — <artifact> not found` to the memory body (never delete)

"Review one-by-one" drops into per-item `AskUserQuestion` with: Apply / Skip / Edit manually.

## Phase 4 — Execute approved actions

- **Add orphan entries** to `MEMORY.md` immediately below existing entries
- **Remove broken pointer lines** from `MEMORY.md`
- **Merge near-duplicates:** write the combined content to the winner file, move the loser to `~/.claude/memory/.trash/<YYYY-MM-DD-HHMM>/`, update the `MEMORY.md` link
- **Flag stale:** prepend the stale line to the memory body

Never perform a hard delete in this skill. All removals go to `.trash/`. Real deletion is a manual user action.

## Phase 5 — Final report

```
## Memory garden — results

Merged:  N
Trashed: N (moved to ~/.claude/memory/.trash/<stamp>/)
Flagged: N
Indexed: N  (orphans added to MEMORY.md)
```

## Forbidden behaviors

- Editing or moving any file without explicit confirmation for that bucket
- Touching files outside `~/.claude/memory/`
- Auto-rewriting a memory body beyond the stale-flag line
- Skipping the scan report — the user must see the findings before any action
- Hard-deleting a file — always go through `.trash/`
- Running on a memory dir with fewer than 5 files
- Inventing new memories or changing an entry's `type` field
