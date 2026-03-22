---
name: knowledge-base-search
description: Search a Progress Agentic RAG knowledge base for source documents — ranked results with relevance scores, no AI generation.
---

# Knowledge Base Search

Search your knowledge base directly. Get ranked source paragraphs with relevance scores — no LLM generation. Use when you want to browse sources, explore what's available, or find specific documents.

## How It Works

```
┌─────────────────────────────────────────────┐
│          KNOWLEDGE BASE SEARCH              │
├─────────────────────────────────────────────┤
│                                             │
│  ALWAYS (standalone)                        │
│  ✓ Hybrid semantic + keyword search         │
│  ✓ Ranked results with relevance scores     │
│  ✓ Document metadata (title, URL, dates)    │
│  ✓ Filter by date, contributor, label       │
│  ✓ Iterative refinement                     │
│                                             │
└─────────────────────────────────────────────┘
```

## Getting Started

```
/arag:search API rate limiting
/arag:sources deployment architecture
```

Or just mention searching your knowledge base — this skill activates automatically.

## Configuration

Same as `cited-answers`. Reads `arag/.claude/settings.local.json`. If missing, prompt through setup.

## API Reference

### POST `{endpoint}/find` — Hybrid search (primary)

The main search endpoint. Combines semantic and keyword search for best results.

**Request:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "API rate limiting"
  }' \
  "{endpoint}/find"
```

**Response fields:**
- `resources` — matched resources (documents) with metadata
- `sentences` — matched sentences/paragraphs with:
  - `text` — the paragraph content
  - `rid` — resource (document) ID
  - `score` — relevance score (0.0 to 1.0, higher is better)
  - `field` — which field the match came from
  - `position` — start/end positions in the original document
- `paragraphs` — matched paragraphs (similar structure)
- Each result includes origin metadata: title, URL, path, creation date, modification date, contributors, labels/tags

### With filters

Add `filter_expression` to scope results:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "API rate limiting",
    "filter_expression": {
      "and": [
        {"modified_at": {"gte": "2026-01-01T00:00:00Z"}}
      ]
    }
  }' \
  "{endpoint}/find"
```

Common filter patterns:
- **By date**: `{"modified_at": {"gte": "2026-01-01T00:00:00Z"}}`
- **By label**: `{"label": "/classification/category"}`
- **By contributor**: Use metadata filters on contributor fields
- **Combined**: Wrap multiple conditions in `"and"` or `"or"`

### GET `{endpoint}/search?query=...` — Separate result sets

Alternative search endpoint. Returns separate result sets for full-text, fuzzy, and semantic search. Useful for debugging which search type finds what.

```bash
curl -s -G \
  -H "Authorization: Bearer {api_key}" \
  --data-urlencode "query=API rate limiting" \
  "{endpoint}/search"
```

## When to Use Find vs Ask

| Scenario | Use |
|----------|-----|
| "What documents do we have about X?" | `/find` |
| "Show me sources on X" | `/find` |
| "What does our KB say about X?" | `/ask` |
| "Explain X based on our docs" | `/ask` |
| "Find everything about X" | `/find` |
| "How does X work?" | `/ask` |
| Browsing and exploring | `/find` |
| Need a synthesized answer | `/ask` |

Rule of thumb: if the user wants to READ sources, use `/find`. If they want an ANSWER, use `/ask`.

## Output Format

### Search results

```markdown
## Search Results for "[query]"

**1. [Document Title]** (relevance: ★★★★★)
> [First ~150 chars of matching paragraph...]
📄 [URL]
🏷️ [Labels/tags if available]
📅 Modified: [date]

**2. [Document Title]** (relevance: ★★★★☆)
> [First ~150 chars of matching paragraph...]
📄 [URL]

---
Showing [N] of [total] results.
```

### Source catalogue (for /arag:sources)

Group by unique document, show paragraph count:

```markdown
## Sources on "[topic]" in [KB Name]

| # | Document | Relevance | Matches |
|---|----------|-----------|---------|
| 1 | **[Title]** | ★★★★★ | 5 paragraphs |
| 2 | **[Title]** | ★★★★☆ | 3 paragraphs |
| 3 | **[Title]** | ★★★☆☆ | 1 paragraph |

Found [N] documents with [M] matching paragraphs.
```

### Relevance score to stars

| Score Range | Stars |
|-------------|-------|
| 0.9 – 1.0 | ★★★★★ |
| 0.7 – 0.89 | ★★★★☆ |
| 0.5 – 0.69 | ★★★☆☆ |
| 0.3 – 0.49 | ★★☆☆☆ |
| 0.0 – 0.29 | ★☆☆☆☆ |

## Execution Flow

1. **Read config** — load settings. Missing? Run setup.
2. **Resolve KB** — named or default.
3. **Parse user intent** — extract search terms, any filters (date, label, contributor).
4. **Build request** — construct curl with query and optional filter_expression.
5. **Call API** — execute, capture response.
6. **Check status** — handle errors.
7. **Parse results** — extract paragraphs, deduplicate by document.
8. **Rank and format** — sort by relevance, convert scores to stars.
9. **Present results** — show in the appropriate format (search results or catalogue).
10. **Suggest next steps** — offer refinement, `/arag:ask` for synthesis, or deeper search.

## Iterative Refinement

If results aren't relevant, help the user refine:

- **Too broad**: "Try narrowing to a specific aspect, like 'API rate limiting per endpoint' instead of 'rate limiting'"
- **Too narrow**: "Try broader terms, like 'API limits' instead of 'GraphQL mutation rate limit 429 error'"
- **Wrong domain**: "Try searching a different KB with `/arag:search [KB Name] your terms`"
- **Need synthesis**: "Want me to synthesize these into an answer? Try `/arag:ask`"

## Related Skills

- `cited-answers` — when the user wants a generated answer, not raw sources
- `research-synthesis` — when exploring a broad topic across multiple sub-questions
- `source-verification` — when checking if a specific claim exists in the KB
