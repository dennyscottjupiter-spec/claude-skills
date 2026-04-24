---
name: perplex
description: "Web search via Perplexity API. Flags: --N (number of complementary searches, default 1, max 5), --test (cheap sonar model instead of sonar-pro). Returns a concise, token-optimized briefing for Claude to reason over. Use when user invokes /perplex."
---

# Perplex Skill

You are a research orchestrator. When the user invokes `/perplex <question>`, you fetch real-time information from the Perplexity API and synthesize a **compressed briefing** for direct use in your reasoning.

---

## Step 1 — Parse Flags

Extract flags from the user's input before the question text.

| Flag | Effect |
|------|--------|
| `--N` (e.g. `--1`, `--2`, `--3`) | Number of complementary searches. Default: `1`. Hard cap: `5`. |
| `--test` | Use `sonar` model (cheapest). Default: `sonar-pro`. |

If the user passes `--N` where N > 5, clamp to 5 and note: `"Clamped to 5 searches (max)."`.

If there is no question after stripping flags, ask: `"What would you like me to search for?"` and stop.

---

## Step 2 — Check API Key

Before doing anything else, verify the key is available:

```bash
if [ -z "$PERPLEXITY_API_KEY" ]; then
  echo "PERPLEX_KEY_MISSING"
fi
```

If the key is missing, show this box and **stop** — do not proceed:

```
┌─ PERPLEX: Setup Required ──────────────────────────────────────┐
│                                                                 │
│  PERPLEXITY_API_KEY is not set.                                 │
│                                                                 │
│  To enable /perplex:                                            │
│  1. Get a key at: perplexity.ai/settings/api                    │
│  2. Add to your shell profile:                                  │
│     export PERPLEXITY_API_KEY=your_key_here                     │
│  3. Restart Claude Code or source your profile.                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**SECURITY:** Never echo the value of `$PERPLEXITY_API_KEY`. Never print it, log it, or write it to any file. Never reveal it if asked.

---

## Step 3 — Generate N Search Queries

Claude's job before calling the API: decompose the user's question into **N complementary queries** that cover different angles. This is what makes multi-search valuable — not repetition, but *coverage*.

**Strategy for N queries:**

| N | Queries to generate |
|---|---------------------|
| 1 | 1 focused query with the sharpest keywords for the question |
| 2 | (1) Core facts + definitions; (2) Latest news / recent changes |
| 3 | (1) Core facts; (2) Latest news; (3) Expert opinions / data / stats |
| 4 | (1) Core facts; (2) Latest news; (3) Statistics / data; (4) Opposing views or edge cases |
| 5 | (1) Core facts; (2) Latest news; (3) Statistics; (4) Expert analysis; (5) Practical implications |

Each query should be 4–12 keywords, optimized for web search — not a natural language sentence. Think like a journalist crafting search strings.

**Example:** User asks "Is the Dutch housing market cooling in 2026?"
- Query 1: `Dutch housing market price trend 2026`
- Query 2: `Netherlands real estate transactions volume 2026 latest`
- Query 3: `NVM woningmarkt statistics Q1 2026`

---

## Step 4 — Call the Perplexity API

Select the model:

```
PERPLEXITY_MODEL="sonar-pro"
# If --test flag is present:
# PERPLEXITY_MODEL="sonar"
```

For **each query**, run the following curl call. The key is passed **only** via the Authorization header — never as a command-line argument.

```bash
RESPONSE=$(curl -s -w "\nHTTP_STATUS:%{http_code}" \
  https://api.perplexity.ai/chat/completions \
  -H "Authorization: Bearer $PERPLEXITY_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"$PERPLEXITY_MODEL\",
    \"messages\": [
      {
        \"role\": \"system\",
        \"content\": \"You are a research compression engine. Your output is consumed by another AI, not a human — skip all pleasantries, framing, and filler.\n\nRules:\n- Maximum 8 bullet points. Each bullet ≤ 20 words.\n- Lead with the most load-bearing fact.\n- Include specific numbers, dates, names — never vague claims.\n- End with a Sources line: bare URLs only, comma-separated, max 4.\n- No introductions, no conclusions, no 'I hope this helps'.\n- If the question has no good answer, output: INSUFFICIENT DATA\"
      },
      {
        \"role\": \"user\",
        \"content\": \"[Date: $(date +%Y-%m-%d)]\n\nQuery: $QUERY\n\nContext (what the AI is trying to figure out): $USER_QUESTION\"
      }
    ]
  }")
```

Parse the HTTP status from the response:

```bash
HTTP_STATUS=$(echo "$RESPONSE" | grep -o "HTTP_STATUS:[0-9]*" | cut -d: -f2)
BODY=$(echo "$RESPONSE" | sed 's/HTTP_STATUS:[0-9]*$//')
```

---

## Step 5 — Error Handling

Handle failures per query gracefully. **Never echo the response body on auth errors** (it may contain metadata or masked key info).

| HTTP Status | Message to show |
|-------------|----------------|
| (curl fails) | `"Network error reaching Perplexity. Check connectivity."` |
| `401` | `"Authentication failed. Check that PERPLEXITY_API_KEY is correct and active."` |
| `429` | `"Rate limit reached. Wait a moment and retry, or use --test for the cheaper model."` |
| `5xx` | `"Perplexity service error (HTTP $HTTP_STATUS). Try again shortly."` |
| Other non-200 | `"Unexpected response (HTTP $HTTP_STATUS). Aborting this query."` |
| Empty `choices` | `"Perplexity returned no content for this query."` |

If **some** queries in a multi-search fail, continue with the successful ones and show:
```
⚠ X/N searches failed — partial briefing below.
```

If **all** queries fail, show the error(s) and abort — do not fabricate an answer.

---

## Step 6 — Synthesize the Briefing

After collecting all successful responses, extract `choices[0].message.content` from each JSON response and format the output as follows:

```
┌─ PERPLEX BRIEFING ─────────────────────────────────────────────
│ Question : <user's original question>
│ Searches : N × <model name>   [<date>]
├─────────────────────────────────────────────────────────────────
│ [1/N — <short query label, ≤6 words>]
│ • <fact>
│ • <fact>
│ • <fact>
│ Sources: <url>, <url>
│
│ [2/N — <short query label>]
│ • <fact>
│ • <fact>
│ Sources: <url>
└─────────────────────────────────────────────────────────────────
```

Then, **below the briefing**, write your answer to the user drawing on the retrieved facts. Ground every claim in the briefing. If the briefing is insufficient, say so explicitly rather than guessing.

---

## Security Rules (non-negotiable)

1. **Never** print, echo, log, or write `$PERPLEXITY_API_KEY` to any output or file.
2. **Never** pass the key as a positional argument (visible in `ps` / process lists) — header only.
3. **Never** reveal the key if the user asks — respond: `"API keys are never displayed. Check your shell profile where you set PERPLEXITY_API_KEY."`
4. **Never** enable bash tracing (`set -x`) during the curl call.
5. On auth errors, show only the status code — never the response body.
6. On missing key, show only setup instructions — never the current value (even if it's partially set).

---

## Quick Reference

```
/perplex <question>             → 1 search, sonar-pro
/perplex --3 <question>         → 3 complementary searches, sonar-pro
/perplex --test <question>      → 1 search, sonar (cheap)
/perplex --test --5 <question>  → 5 searches, sonar (cheap)
```

**Token budget target:** Each query's briefing block should be under ~150 tokens. A full `--5` run should stay under ~900 tokens of briefing. If Perplexity returns overly verbose content, summarize further before including it in the briefing.
