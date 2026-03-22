---
name: focus
description: Search within a specific document in your knowledge base.
argument-hint: "<document name or ID> <query>"
---

# ARAG Focus

Drill into a single document. Search within a specific resource to find exactly the paragraph you need — without noise from other documents in the KB.

## Usage

```
/arag:focus "API Guide" rate limiting
/arag:focus "Security Whitepaper" encryption at rest
/arag:focus a1b2c3d4-5678-90ab-cdef-1234567890ab authentication
```

You can reference documents by title (in quotes) or by resource ID.

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB

Same logic as other commands — named KB if specified, otherwise default.

### 3. Resolve the resource

If the user gave a document title, look it up to get the `rid`:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "API Guide"
  }' \
  "{endpoint}/catalog"
```

Match the title (case-insensitive). If multiple matches, list them and ask the user to pick:

```
Found multiple documents matching "API Guide":
1. **API Guide v2** (`a1b2...`) — Modified 2026-03-15
2. **API Guide (Legacy)** (`c3d4...`) — Modified 2025-11-01

Which one? (1 or 2)
```

If the user gave a `rid` directly, use it as-is.

### 4. Search within the resource

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "rate limiting"
  }' \
  "{endpoint}/resource/{rid}/find"
```

### 5. Present results

```
## Results in "API Guide v2"

**1.** (relevance: ★★★★★)
> "API rate limiting is enforced at 1000 requests per minute per API key.
> Exceeding this limit returns a 429 status code with a Retry-After header..."

**2.** (relevance: ★★★★☆)
> "Rate limits can be configured per endpoint. The default limit applies
> to all endpoints unless an override is specified in the dashboard..."

**3.** (relevance: ★★★☆☆)
> "For bulk operations, consider using the batch API endpoint which has
> separate rate limits and supports up to 100 operations per call..."

---
Found 3 matching paragraphs in this document.
```

### 6. Offer next steps

> "Want more?"
> - `/arag:ask [question about this document]` — get a synthesized answer
> - `/arag:sources [topic]` — find other relevant documents
> - `/arag:search [broader terms]` — search across all documents

### 7. Handle errors

- **401/403**: "Authentication failed. Run `/arag:setup` to check your API key."
- **404 (resource)**: "Document not found. It may have been deleted — check `/arag:manage list`."
- **404 (KB)**: "Knowledge base not found. Run `/arag:setup` to check your endpoint URL."
- **Empty results**: "No matches for '[query]' in this document. Try different terms or `/arag:search` to look across all documents."
- **Resource still processing**: "This document is still being processed. Check `/arag:status` and try again shortly."
- **Network error**: "Couldn't reach the ARAG API. Check your internet connection and endpoint URL."

## Example

```
You: /arag:focus "Security Whitepaper" encryption methods

Claude:
## Results in "Security Whitepaper"

**1.** (relevance: ★★★★★)
> "All data at rest is encrypted using AES-256. Encryption keys are rotated
> every 90 days and managed through AWS KMS..."

**2.** (relevance: ★★★★☆)
> "Data in transit is protected using TLS 1.3. All API endpoints require
> HTTPS — plaintext HTTP connections are rejected..."

---
Found 2 matching paragraphs in this document.

Want more?
- `/arag:ask How are encryption keys managed?`
- `/arag:sources encryption` — find other documents about encryption
```
