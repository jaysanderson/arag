---
name: ask
description: Ask your knowledge base a question and get a fully-cited AI answer.
argument-hint: "<question>"
---

# ARAG Ask

The hero command. Ask your knowledge base anything — get a cited answer with every claim traced to its source.

## Usage

```
/arag:ask How do MCP drivers work in agentic workflows?
/arag:ask [Product Docs] What is our data retention policy?
```

The optional `[KB Name]` in brackets targets a specific knowledge base. Without it, Claude uses the default.

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing or empty, run `/arag:setup` first.

### 2. Resolve which KB to use

- If the user specified `[KB Name]`, find the matching entry in `knowledge_bases` by name (case-insensitive).
- If no name given, use the entry where `"default": true`.
- If no default is set, use the first entry.
- If the named KB isn't found, list available KBs and ask the user to pick.

### 3. Call the ARAG Ask API

Use bash `curl` to POST to `{endpoint}/ask`:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -H "x-synchronous: true" \
  -d '{
    "query": "the user question here",
    "citations": "llm_footnotes",
    "context": []
  }' \
  "{endpoint}/ask"
```

**Headers:**
- `Authorization: Bearer {api_key}`
- `Content-Type: application/json`
- `x-synchronous: true` — required for synchronous response

**Body fields:**
- `query` — the user's question (from `$ARGUMENTS`)
- `citations` — always `"llm_footnotes"`
- `context` — array of prior Q&A pairs for multi-turn. Empty for first question.

### 4. Parse the response

The response JSON contains:

- `answer` — the generated text with inline Markdown footnotes like `[^1]`, `[^2]`
- `retrieval_results` — source paragraphs with metadata (URL, title, path, dates, contributors, tags)
- `footnote_to_context` — maps block IDs (`block-AA`, `block-CB`) to paragraph IDs. This is the citation traceability chain.
- `citations` — paragraph ID to answer text span mapping
- `retrieval_best_matches` — best-matching paragraph IDs
- `status` — `"success"` or `"error"`

### 5. Present the answer

**Answer section:**
Display the `answer` text with its inline footnotes preserved exactly as returned.

**Sources section:**
After the answer, add a `## Sources` section. For each footnote number in the answer:

1. Look up the block ID in `footnote_to_context`
2. Find the corresponding paragraph in `retrieval_results`
3. Present:

```
## Sources

[^1]: **[Document Title]** — [short snippet of the source paragraph]
  [URL if available]

[^2]: **[Document Title]** — [short snippet of the source paragraph]
  [URL if available]
```

**Unsupported claims:**
If any substantive claim in the answer does NOT have a footnote, flag it:

> ⚠️ **Note:** Some claims in this answer are not directly supported by citations from the knowledge base. Verify independently.

### 6. Handle errors

- **401/403**: "Authentication failed. Run `/arag:setup` to check your API key."
- **404**: "Knowledge base not found. Run `/arag:setup` to check your endpoint URL."
- **Empty results**: "The knowledge base didn't return any relevant results for this question. Try rephrasing or use `/arag:search` to browse available content."
- **Network error**: "Couldn't reach the ARAG API. Check your internet connection and endpoint URL."

### 7. Multi-turn conversations

If the user asks a follow-up question in the same conversation, pass prior exchanges in the `context` array:

```json
{
  "query": "follow-up question",
  "citations": "llm_footnotes",
  "context": [
    {"author": "USER", "text": "previous question"},
    {"author": "NUCLIA", "text": "previous answer"}
  ]
}
```

This gives the ARAG model conversational context for better answers.

## Example

```
You: /arag:ask How does document ingestion work?

Claude:
Document ingestion in ARAG follows a multi-stage pipeline[^1]. Files are first
processed through format-specific extractors that handle PDF, DOCX, HTML, and
other formats[^2]. The extracted text is then chunked into paragraphs and each
chunk is embedded using a neural model for semantic search[^1].

## Sources

[^1]: **Ingestion Pipeline Overview** — "The ingestion pipeline processes
  documents through extraction, chunking, and embedding stages..."
  https://docs.example.com/ingestion

[^2]: **Supported File Formats** — "ARAG supports over 40 file formats
  including PDF, DOCX, HTML, Markdown, and plain text..."
  https://docs.example.com/formats
```
