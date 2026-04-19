# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A ReportLab-only PDF generator used as a Claude Code skill (`/pdf-doc`). It accepts JSON content files, plain-text/Markdown, or stdin and produces polished A4 technical PDFs.

**Active script:** `scripts/pdf_engine_v4.py` (v4.0.0)

## Common commands

```bash
# Install dependency
pip install reportlab

# Generate PDF from JSON
python3 scripts/pdf_engine_v4.py references/example_content.json -o output.pdf

# Generate PDF from Markdown/plain-text via stdin
echo "# My Guide\n\nSome content." | python3 scripts/pdf_engine_v4.py --stdin -o output.pdf

# Dump normalized content model (useful for debugging parsing)
python3 scripts/pdf_engine_v4.py input.json --dump-json debug.json -o output.pdf

# Key CLI flags
--toc / --no-toc         # force enable/disable table of contents
--numbered / --not-numbered
--title "Override"
--lang pt-BR
--emoji-budget 0         # disable all decorative icons
--target-pages N         # warn if rendered page count exceeds N
```

## Architecture

### Data flow

```
Input (JSON | Markdown | stdin | paste)
  → read_input_text()
  → detect_input_format()
  → load_content()          # parse_text_input() or json.loads()
  → apply_cli_overrides()
  → normalize_content()     # fill defaults, slugify filename, coerce types
  → validate_content()      # raise ValueError on schema violations
  → build_pdf()             # emit ReportLab story → PDF
```

### Key classes

| Class | Purpose |
|---|---|
| `PDFDoc` | `BaseDocTemplate` subclass; draws header/footer chrome on every page; fires `afterFlowable` to register PDF bookmarks and TOC entries |
| `Callout` | Custom `Flowable` for note/tip/warning/caution/important boxes with colored left bar |
| `CodeBlock` | Custom `Flowable` for dark-themed code with line numbers; wraps long lines |
| `BoxDiagram` | Custom `Flowable` for responsive multi-row colored box diagrams |
| `EmojiBudget` | Counts decorative icon uses; stops emitting icons when budget is exhausted |
| `SectionCounter` | Tracks h1/h2/h3 numbering state for `--numbered` mode |

### Plain-text parser (`parse_text_input`)

Parses Markdown-lite input in a single forward pass:

1. YAML-style front matter (`---` … `---`)
2. Top metadata lines (`Key: value`) until first non-metadata line
3. Title/subtitle inference from first short line or `# H1`
4. Block-by-block parsing using `consume_*` helpers

Each `consume_*` function returns `(parsed_data, new_index)` so the main loop advances the cursor correctly. The `is_block_boundary()` guard prevents greedy paragraph consumption.

### Content JSON schema

Top-level fields: `title`, `subtitle`, `filename`, `doc_kind` (tutorial/howto/reference/explanation), `audience`, `updated`, `lang`, `toc`, `numbered`, `keywords`, `summary`, `prerequisites`, `result`, `footer`, `emoji_budget`, `target_pages`.

Block types: `h1`, `h2`, `h3`, `p`, `bullets`, `steps`, `code`, `table`, `note`, `tip`, `warning`, `caution`, `important`, `boxes`, `spacer`, `hr`, `footnotes`, `pagebreak`.

See `references/example_content.json` for a complete working example.

### Font handling

`setup_optional_fonts()` runs at import time and tries a list of OS font paths to register `PDFEngineIcon` for symbol glyphs. If none are found, `ICON_FONT_AVAILABLE = False` and all icons fall back to plain ASCII characters. The skill works correctly either way.

### TOC implementation

Uses ReportLab's `TableOfContents` flowable with `doc.multiBuild(story)` (two-pass build). `PDFDoc.afterFlowable()` fires `self.notify("TOCEntry", ...)` for every heading paragraph, which populates the TOC on the second pass.

## When modifying the engine

- **Adding a new block type:** add to `SUPPORTED_BLOCKS`, add a branch in `render_block()`, add validation in `validate_content()`, update `SKILL.md` schema docs.
- **Adding a new callout variant:** add to `CALL_OUTS` dict; `render_block()` and `Callout` use it automatically.
- **Changing layout constants** (`MARGIN_H`, `CONTENT_W`, etc.): these are module-level and used by all flowables — changes affect every rendered document.
- **`sanitize_inline()`** handles the inline formatting pipeline (bold/italic/code escaping). Touch it carefully — it uses placeholder stashing to avoid double-escaping.
