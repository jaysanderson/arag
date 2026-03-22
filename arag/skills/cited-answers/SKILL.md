---
name: cited-answers
description: Get fully-cited AI answers from a Progress Agentic RAG knowledge base. Every claim traced to its source document.
---

# Cited Answers

Ask a knowledge base any question and get an AI-generated answer where every claim is traced back to its source document through inline footnotes.

## How It Works

```
┌─────────────────────────────────────────────┐
│              CITED ANSWERS                  │
├─────────────────────────────────────────────┤
│                                             │
│  ALWAYS (standalone)                        │
│  ✓ Generative RAG answers via ARAG API      │
│  ✓ Inline footnote citations                │
│  ✓ Source document traceability             │
│  ✓ Multi-turn conversation context          │
│  ✓ Unsupported claim flagging               │
│                                             │
└─────────────────────────────────────────────┘
```

## Getting Started

```
/arag:ask What is our data retention policy?
/arag:ask [Product Docs] How does document ingestion work?
```

Or just ask naturally — this skill activates when Claude detects a question that should be answered from the knowledge base.

## Configuration

This skill reads credentials from `arag/.claude/settings.local.json`:

```json
{
  "knowledge_bases": [
    {
      "name": "Product Docs",
      "endpoint": "https://europe-1.rag.progress.cloud/api/v1/kb/df8b4c24-...",
      "api_key": "kb-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "default": true
    }
  ],
  "preferred_citation_mode": "llm_footnotes"
}
```

If this file is missing or empty, prompt the user through setup:
1. Ask for their KB endpoint URL (looks like `https://europe-1.rag.progress.cloud/api/v1/kb/{kb-id}`)
2. Ask for their Knowledge Box API key (a service access token from their KB settings)
3. Optionally ask for a friendly name
4. Save to `arag/.claude/settings.local.json`
5. Test the connection

If multiple KBs are configured, use the one marked `"default": true` unless the user specifies a name.

## API Reference

### POST `{endpoint}/ask` — Generative RAG with citations

**Request:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -H "x-synchronous: true" \
  -d '{
    "query": "How does document ingestion work?",
    "citations": "llm_footnotes",
    "context": []
  }' \
  "{endpoint}/ask"
```

**Required headers:**
- `Authorization: Bearer {api_key}`
- `Content-Type: application/json`
- `x-synchronous: true`

**Required body fields:**
- `query` (string) — the user's question
- `citations` (string) — always use `"llm_footnotes"`
- `context` (array) — prior Q&A pairs for multi-turn, empty for first question

**Optional body fields:**
- `extra_context_images` (array of strings) — base64-encoded images or image URLs. Include when the user provides a screenshot, diagram, or image to ask about alongside their question. Enables visual Q&A against the KB.
- `filter_expression` (object) — scope the answer to specific documents by date, label, or contributor. Same syntax as the `/find` endpoint filters:
  - By date: `{"modified_at": {"gte": "2026-01-01T00:00:00Z"}}`
  - By label: `{"label": "/classification/security"}`
  - Combined: `{"and": [{"modified_at": {"gte": "..."}}, {"label": "..."}]}`

**Response fields:**
- `answer` — generated text with inline Markdown footnotes `[^1]`, `[^2]`, etc.
- `retrieval_results` — source paragraphs with origin metadata:
  - title, URL, path, dates, contributors, tags
- `footnote_to_context` — maps block IDs (e.g., `block-AA`, `block-CB`) to paragraph IDs. **This is the citation traceability chain.** Every inline footnote references a block, and this map tells you which source paragraph that block came from.
- `citations` — paragraph ID to answer text span mapping
- `retrieval_best_matches` — IDs of the best-matching paragraphs
- `metadata` — token counts, timing info
- `relations` — extracted entity relations
- `status` — `"success"` or `"error"`

### Citation mode: `llm_footnotes` (BETA — always use this)

This mode injects inline Markdown footnotes directly into the answer text. The traceability chain is:

```
Answer text with [^1] → footnote_to_context maps block-XX → paragraph ID → retrieval_results has full source
```

### Multi-turn context

For follow-up questions, pass previous exchanges in the `context` array:

```json
{
  "query": "What about the chunking step specifically?",
  "citations": "llm_footnotes",
  "context": [
    {"author": "USER", "text": "How does document ingestion work?"},
    {"author": "NUCLIA", "text": "Document ingestion follows a multi-stage pipeline..."}
  ]
}
```

## Output Format

Present every answer in this structure:

```markdown
[Answer text with inline footnotes preserved exactly as returned from the API]

## Sources

[^1]: **[Document Title]** — [first ~100 chars of source paragraph]
  [URL if available]

[^2]: **[Document Title]** — [first ~100 chars of source paragraph]
  [URL if available]
```

### Building the Sources section

For each footnote `[^N]` in the answer:

1. The answer text contains references like `[^1]`. The API response's `footnote_to_context` maps block IDs to paragraph IDs.
2. Look through `retrieval_results` to find the matching paragraph.
3. Extract the document title, a short snippet of the paragraph text, and the URL.
4. List them in footnote order.

### Flagging unsupported claims

If the answer contains substantive factual claims that do NOT have an associated footnote:

> ⚠️ **Note:** Some claims in this answer are not directly cited from the knowledge base. Consider verifying independently or using `/arag:search` to find supporting sources.

## Execution Flow

1. **Read config** — load `arag/.claude/settings.local.json`. Missing? Run setup.
2. **Resolve KB** — use named KB if specified, otherwise default.
3. **Build request** — construct the curl command with query, citations mode, and context.
4. **Call API** — execute the curl command, capture response.
5. **Check status** — if error, report and suggest fixes.
6. **Parse response** — extract answer, retrieval_results, footnote_to_context.
7. **Present answer** — show answer text with footnotes.
8. **Build sources** — map footnotes to source documents, present Sources section.
9. **Flag gaps** — note any uncited claims.
10. **Store context** — save Q&A pair for potential follow-up questions.

## Error Handling

| Error | Message | Action |
|-------|---------|--------|
| 401/403 | "Authentication failed" | Suggest running `/arag:setup` |
| 404 | "KB not found" | Check endpoint URL, suggest `/arag:setup` |
| Empty results | "No relevant results" | Suggest rephrasing, try `/arag:search` |
| Network error | "Can't reach API" | Check connection and endpoint |
| Malformed response | "Unexpected response" | Show raw response, suggest retrying |

## Related Skills

- `knowledge-base-search` — when the user wants raw results, not generated answers
- `research-synthesis` — when the question is broad and needs decomposition
- `source-verification` — when the user wants to verify a specific claim
- `kb-management` — when the user wants to upload, delete, or organize KB content
- `knowledge-graph` — when the user wants to explore entities and relationships
