---
name: search
description: Search your knowledge base for source documents — no AI generation, just ranked results.
argument-hint: "<search terms>"
---

# ARAG Search

Raw search across your knowledge base. Returns ranked source paragraphs with relevance scores — no LLM generation. Use when you want to browse sources directly.

## Usage

```
/arag:search data encryption at rest
/arag:search [Engineering Wiki] kubernetes deployment patterns
```

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB

Same logic as `/arag:ask` — use named KB if specified, otherwise default.

### 3. Call the ARAG Find API

Use bash `curl` to POST to `{endpoint}/find`:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "the search terms here"
  }' \
  "{endpoint}/find"
```

**Optional filters:** If the user specifies constraints like "only docs from last month" or "only from contributor X", add a `filter_expression` to the body. Common filters:

```json
{
  "query": "search terms",
  "filter_expression": {
    "and": [
      {"modified_at": {"gte": "2026-01-01T00:00:00Z"}},
      {"label": "/classification/topic"}
    ]
  }
}
```

### 4. Parse the response

The response contains ranked results. Each result includes:

- `text` — the paragraph content
- `rid` — resource (document) ID
- `score` — relevance score (higher is better)
- `metadata` — title, URL, path, dates, contributors, labels

### 5. Present results

Show up to 10 results in a scannable format:

```
## Search Results for "data encryption at rest"

**1. Data Security Architecture** (score: 0.92)
> "All customer data is encrypted at rest using AES-256. Encryption keys
> are managed through a dedicated key management service..."
📄 https://docs.example.com/security/encryption

**2. Compliance Requirements** (score: 0.87)
> "SOC 2 Type II certification requires encryption at rest for all
> personally identifiable information..."
📄 https://docs.example.com/compliance/soc2

**3. Database Configuration Guide** (score: 0.74)
> "Enable transparent data encryption (TDE) in the database settings
> panel under Security > Encryption..."
📄 https://docs.example.com/admin/database

---
Showing 3 of 12 results. Refine your search or ask me to show more.
```

### 6. Handle errors

Same error handling as `/arag:ask`.

### 7. Iterative refinement

If results aren't relevant, suggest:
- Narrowing with more specific terms
- Broadening with fewer or different terms
- Using `/arag:ask` for a synthesized answer instead
- Adding filters to scope by date, contributor, or label
