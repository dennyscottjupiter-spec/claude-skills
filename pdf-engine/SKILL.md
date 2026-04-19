---
name: pdf-engine
description: >
  A polished, ReportLab-only PDF generator for technical documentation. Use this skill
  whenever the user wants to create a professional technical PDF document — guides,
  how-to docs, SOPs, runbooks, reference manuals, tutorials, or explanations. Trigger
  this skill when you see phrases like "create a PDF", "generate a document", "make a
  guide", "build a report", "write a technical doc", "turn this into a PDF", or when the
  user pastes structured content (with headings, steps, code blocks, tables) and wants a
  polished PDF output. Also trigger when the user provides or references a JSON content
  file matching the PDF Engine schema. This skill produces clean A4 technical PDFs with
  callouts, numbered steps, tables, code blocks, footnotes, box diagrams, a real table
  of contents with page numbers, and PDF bookmarks. It does NOT handle PDF reading,
  merging, splitting, form-filling, or OCR — those belong to the general PDF skill.
---

# PDF Engine — Technical Document Generator

## Overview

This skill uses a bundled Python script (`scripts/pdf_engine_v4.py`) powered entirely by
ReportLab to produce polished A4 technical PDFs. The script accepts either a JSON content
file or plain-text/Markdown-lite input and outputs a professional document with:

- Title page with metadata (audience, doc kind, update date)
- Real table of contents with clickable page numbers
- PDF bookmarks/outlines for sidebar navigation
- Callout boxes (note, tip, warning, caution, important)
- Numbered steps, bullet lists, fenced code blocks with line numbers
- Tables with auto-sized columns and alternating row colors
- Box diagrams for architecture/flow visualization
- Footnotes, horizontal rules, page breaks
- PDF-safe icon badges (no fragile emoji fonts)

### Known limitations

Be upfront with the user if they ask for something the engine cannot do:

- **No embedded images.** The engine renders text-based content only. If the user
  needs screenshots or photos, suggest they add them after generation using a PDF
  editor, or mention that box diagrams are available as a visual alternative.
- **No clickable hyperlinks.** Inline formatting supports bold, italic, and inline
  code, but not `<a href>` links or `[text](url)` Markdown links. URLs can be
  included as plain text but will not be clickable in the PDF.
- **Flat block structure.** The `"blocks"` array is flat — you cannot nest a code
  block inside a step, or a warning inside a bullet. To work around this, place
  related blocks sequentially (e.g., a `steps` block followed immediately by a
  `code` block that illustrates step 2).
- **Top-level prerequisites only.** The `"prerequisites"` and `"result"` fields
  apply to the whole document. For multi-part guides, use a `note` or `important`
  callout block within each section instead.

## Before you start: ask the user 3 setup questions

Every time you use this skill, ask the user three quick questions **before** building
the content JSON. Present them as multiple-choice options with defaults so the user
can just press through quickly. Use the ask_user_input tool for this.

**Question 1 — Language**
- Português Brasileiro (default)
- English

**Question 2 — Include a summary section?**
- Yes (default)
- No

This controls whether the JSON includes a `"summary"` field — a brief paragraph
shown in a callout box at the top of the document, before the table of contents.
It is a text paragraph, not a data table.

**Question 3 — Approximate document length**
- Short (1–3 pages) (default)
- Medium (4–8 pages)
- Long (9+ pages)

The language choice affects the `lang` field in the JSON (`"pt-BR"` or `"en"`).
The length preference guides how much detail you put in each section — it is a soft
guideline, not a hard constraint.

If the user has already specified these preferences (e.g., "make this in English",
"keep it short"), skip the corresponding question.

## How to build a PDF

### Step 0: Resolve the script path

The engine script is bundled at `scripts/pdf_engine_v4.py` relative to this skill's
directory. Before running anything, resolve the absolute path:

```bash
# Find the skill's installed location dynamically
SKILL_DIR="$(dirname "$(find /mnt/skills -name pdf_engine_v4.py -path '*/pdf-engine/*' 2>/dev/null | head -1)")"
SCRIPT="${SKILL_DIR}/pdf_engine_v4.py"
```

Use `$SCRIPT` in all subsequent commands. Never hardcode a placeholder path like
`/path/to/skill/...` — it will cause a `FileNotFoundError`.

### Step 0.5: Ensure ReportLab is available

Check once, install only if missing:

```bash
python -c "import reportlab" 2>/dev/null || pip install reportlab --break-system-packages -q
```

### Step 1: Gather or receive content

The user may provide content in several ways:

- **Pasted text in chat**: The user pastes raw text, Markdown, or structured notes
  directly into the conversation. Convert it into the JSON content model below.
- **A JSON file**: The user uploads a `.json` file matching the content schema.
  Use it directly (with any requested modifications).
- **A text/Markdown file**: The user uploads `.txt` or `.md`. Pass it to the script
  as-is (the script parses Markdown-lite natively).
- **A description or request**: The user describes what they want ("write me a guide
  on setting up Nginx reverse proxies"). You write the full content and build the JSON.

### Step 2: Build the JSON content model

Create a JSON file at `/home/claude/content.json` following this schema. See
`references/example_content.json` for a complete working example.

```json
{
  "title": "Document Title",
  "subtitle": "Optional subtitle line",
  "filename": "output-filename.pdf",
  "doc_kind": "howto",
  "audience": "Target audience description",
  "updated": "March 2026",
  "lang": "pt-BR",
  "toc": true,
  "numbered": true,
  "keywords": ["docker", "deployment", "containers"],
  "summary": "Brief summary of this document's purpose.",
  "footer": "Optional footer text",
  "blocks": []
}
```

**Top-level fields:**

| Field          | Required | Description                                                |
|----------------|----------|------------------------------------------------------------|
| `title`        | yes      | Document title                                             |
| `subtitle`     | no       | Subtitle shown below the title                             |
| `filename`     | no       | Output PDF filename (auto-generated from title if omitted) |
| `doc_kind`     | no       | One of: `tutorial`, `howto`, `reference`, `explanation`    |
| `audience`     | no       | Target audience shown in the header metadata               |
| `updated`      | no       | Date string shown in header metadata                       |
| `lang`         | no       | Language code for PDF metadata. Default: `en`              |
| `toc`          | no       | `true` to include a table of contents                      |
| `numbered`     | no       | `true` to auto-number headings (1. / 1.1 / 1.1.1)         |
| `keywords`     | no       | List of strings or comma-separated string for PDF keywords |
| `summary`      | no       | Summary paragraph shown in a callout box before the TOC    |
| `prerequisites`| no       | List of strings shown as "Before you begin" bullets        |
| `result`       | no       | Outcome text shown in a callout box                        |
| `footer`       | no       | Footer text at the bottom of the document                  |
| `emoji_budget` | no       | Max decorative icons per document. Default: 5              |

### Step 3: Populate the blocks array

Each block is an object with a `type` field. Supported types:

**Headings** — `h1`, `h2`, `h3`
```json
{"type": "h1", "text": "Section Title"}
```

Use `{"type": "pagebreak"}` before each new `h1` section (except the first) to
give the document a professional feel — each major section starts on a fresh page.

**Paragraph** — `p`
```json
{"type": "p", "text": "Body text with **bold** and *italic* support."}
```

**Bullet list** — `bullets`
```json
{"type": "bullets", "items": ["First point", "Second point"]}
```

**Numbered steps** — `steps`
```json
{"type": "steps", "items": ["Do this first", "Then do this"]}
```

**Code block** — `code`
```json
{"type": "code", "lines": ["def hello():", "    print('hello')"]}
```

**Table** — `table`
```json
{
  "type": "table",
  "headers": ["Name", "Type", "Description"],
  "rows": [["port", "int", "Listening port"]],
  "col_ratios": [1.5, 0.8, 2.5]
}
```
`col_ratios` is optional — the engine auto-sizes columns if omitted.

**Callouts** — `note`, `tip`, `warning`, `caution`, `important`
```json
{"type": "warning", "text": "Do not run this in production without testing."}
```

**Box diagram** — `boxes`
```json
{
  "type": "boxes",
  "items": [
    {"title": "Build", "subtitle": "Compile assets", "color": "#2457a6"},
    {"title": "Test", "subtitle": "Run test suite", "color": "#2a8f57"},
    {"title": "Deploy", "subtitle": "Push to prod", "color": "#c48b00"}
  ]
}
```

**Layout helpers:**
```json
{"type": "spacer", "height": 15}
{"type": "hr", "width": "60%", "thickness": 1}
{"type": "pagebreak"}
```

**Footnotes** — `footnotes`
```json
{
  "type": "footnotes",
  "items": [
    {"n": "1", "text": "See the official documentation for details."}
  ]
}
```

**Inline formatting supported in text fields:**
- `**bold**` or `<b>bold</b>`
- `*italic*` or `<i>italic</i>`
- `` `inline code` `` renders in monospace
- `<br/>` for line breaks

### Step 4: Write the JSON safely

When generating JSON with code blocks, be careful with escaping. Code lines often
contain quotes, backslashes, and special characters that break JSON if not properly
escaped. The safest approach is to **write the JSON via a Python script** instead of
raw text output:

```python
import json
content = {
    "title": "My Guide",
    "blocks": [
        {"type": "code", "lines": [
            'docker run -d -p 8080:8080 -e APP_ENV="production" my-app:latest'
        ]}
    ]
}
with open("/home/claude/content.json", "w") as f:
    json.dump(content, f, ensure_ascii=False, indent=2)
```

This guarantees valid JSON regardless of what characters appear in the content.
Using `json.dump()` handles all escaping automatically — no manual backslash
juggling needed.

### Step 5: Run the script

```bash
python "$SCRIPT" /home/claude/content.json -o /home/claude/output.pdf
```

Then copy the output to the delivery directory and present it:
```bash
cp /home/claude/output.pdf /mnt/user-data/outputs/output.pdf
```

Use `present_files` to share the PDF with the user.

### Alternative: plain-text / Markdown input

If the user uploads a `.txt` or `.md` file, pass it directly:
```bash
python "$SCRIPT" /mnt/user-data/uploads/guide.md -o /home/claude/guide.pdf
```

The script's built-in parser handles:
- `# H1`, `## H2`, `### H3` headings
- `- bullet` and `1. step` lists
- Fenced code blocks with ` ``` `
- Pipe tables (`| A | B |`)
- `Note:`, `Tip:`, `Warning:`, `Important:` callout lines
- `[[pagebreak]]` directives
- Top metadata lines like `Title: ...`, `Subtitle: ...`, `Toc: yes`
- `[[boxes]]` / `[[/boxes]]` diagram blocks

## Quality checklist

Before delivering, mentally verify:

1. Every heading, paragraph, and list item has real, useful content (no placeholders)
2. Code blocks use the `"lines"` array format, not a single string
3. Table rows have exactly as many cells as headers
4. The `doc_kind` matches the document style (howto for step-based, reference for tables)
5. Callout types are used appropriately (warning for danger, tip for helpful advice)
6. If the user asked for Brazilian Portuguese, all content text is in pt-BR
7. The JSON was written via `json.dump()` (not raw text) to avoid escaping bugs
8. A `{"type": "pagebreak"}` appears before each `h1` (except the first one) for
   clean section separation
9. The `keywords` field is populated with relevant terms for PDF searchability

## Common patterns

**User pastes raw content in chat:**
Convert it to the JSON model. Look for natural heading structure, lists, code,
and tables. Wrap explanatory asides in callouts. Add a summary if the user
requested one. Then generate the PDF.

**User asks you to write a guide from scratch:**
Use your knowledge to write comprehensive content. Structure it with clear h1/h2
sections, practical steps, relevant code examples, configuration tables, and
warnings about common pitfalls. Then generate the PDF.

**User uploads a JSON file:**
Read it, validate it matches the schema, apply any requested changes, then run
the script directly on it.

**User needs something the engine can't do:**
If the user asks for embedded images, clickable hyperlinks, or nested block
structures, be honest about the limitation and suggest workarounds (see the
"Known limitations" section at the top).
