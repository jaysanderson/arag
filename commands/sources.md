---
name: sources
description: Browse what's in your knowledge base — see document titles, URLs, and relevance scores.
argument-hint: "<topic to browse>"
---

# ARAG Sources

Browse the contents of your knowledge base. See what documents exist on a topic — titles, URLs, dates, relevance scores. No AI generation, just a catalogue of what's available.

## Usage

```
/arag:sources API documentation
/arag:sources [Engineering Wiki] deployment guides
/arag:sources everything about onboarding
```

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB

Same logic as other commands — named KB or default.

### 3. Call the ARAG Find API

Use bash `curl` to POST to `{endpoint}/find`:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "the topic here"
  }' \
  "{endpoint}/find"
```

### 4. Extract unique documents

The API returns paragraphs, not documents. Multiple paragraphs may come from the same document. Group results by document (using `rid` — resource ID) and present unique documents:

### 5. Present the catalogue

```
## Sources on "[topic]" in [KB Name]

| # | Document | Relevance | Paragraphs |
|---|----------|-----------|------------|
| 1 | **[Title]** | ★★★★★ | 5 matches |
|   | 📄 [URL] | | |
| 2 | **[Title]** | ★★★★☆ | 3 matches |
|   | 📄 [URL] | | |
| 3 | **[Title]** | ★★★☆☆ | 1 match |
|   | 📄 [URL] | | |

---
Found [N] documents with [M] matching paragraphs.
```

Convert numeric relevance scores to stars (5-star scale) for quick scanning.

### 6. Offer next steps

After showing results:

> "Want me to dig into any of these? Try:"
> - `/arag:ask [specific question about a document]`
> - `/arag:search [narrower terms]`
> - `/arag:research [broader topic]`

### 7. Handle errors

Same error handling as other commands.
