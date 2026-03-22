---
name: kb-management
description: Manage resources in a Progress Agentic RAG knowledge base — upload, delete, label, update, and monitor processing status.
---

# KB Management

Upload files, delete outdated content, manage labels, and monitor processing status — all from within Claude. Transforms the ARAG plugin from read-only to full read-write.

## How It Works

```
┌─────────────────────────────────────────────┐
│            KB MANAGEMENT                    │
├─────────────────────────────────────────────┤
│                                             │
│  ALWAYS (standalone)                        │
│  ✓ Upload files and URLs to KB              │
│  ✓ Create resources from text content       │
│  ✓ Delete outdated resources                │
│  ✓ Add/remove classification labels         │
│  ✓ Update resource metadata                 │
│  ✓ List and browse resources                │
│  ✓ Monitor processing status                │
│  ✓ KB health dashboard                      │
│                                             │
└─────────────────────────────────────────────┘
```

## Getting Started

```
/arag:manage upload https://docs.example.com/api-guide
/arag:manage list
/arag:manage label "API Guide" add security
/arag:manage delete "Outdated Policy v1"
/arag:status
```

This skill activates when Claude detects that the user wants to add, remove, organize, or inspect resources in their knowledge base.

## Configuration

Same as other skills. Reads `arag/.claude/settings.local.json`. If missing, prompt through setup.

## API Reference

### POST `{endpoint}/upload` — Upload a file

Upload a local file to the knowledge base using multipart form data.

**Request:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "X-FILENAME: document.pdf" \
  -F "file=@/path/to/document.pdf" \
  "{endpoint}/upload"
```

**Required headers:**
- `Authorization: Bearer {api_key}`
- `X-FILENAME: {filename}` — the original filename

**Response fields:**
- `rid` — the resource ID assigned to the uploaded file
- `title` — extracted or assigned title
- `status` — processing status (`PENDING`, `PROCESSED`, `ERROR`)

### POST `{endpoint}/resources` — Create a resource from URL or text

Create a resource programmatically from a URL or raw text content.

**Request (from URL):**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "API Guide",
    "slug": "api-guide",
    "origin": {
      "url": "https://docs.example.com/api-guide"
    }
  }' \
  "{endpoint}/resources"
```

**Request (from text):**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Meeting Notes Q1",
    "texts": {
      "text": {
        "body": "The full text content goes here..."
      }
    }
  }' \
  "{endpoint}/resources"
```

**Response fields:**
- `rid` — resource ID
- `title` — resource title
- `status` — processing status

### GET `{endpoint}/resources` — List all resources

Browse all resources in the knowledge base with pagination.

**Request:**

```bash
curl -s -X GET \
  -H "Authorization: Bearer {api_key}" \
  "{endpoint}/resources?page=0&size=20"
```

**Query parameters:**
- `page` (int) — page number, starting at 0
- `size` (int) — results per page (default 20, max 100)

**Response fields:**
- `resources` — array of resource objects, each containing:
  - `id` — resource ID (`rid`)
  - `title` — resource title
  - `metadata` — origin URL, dates, contributors, labels
  - `status` — processing status (`PENDING`, `PROCESSED`, `ERROR`)
  - `created` — creation timestamp
  - `modified` — last modification timestamp
- `pagination` — total count, page info

### DELETE `{endpoint}/resource/{rid}` — Delete a resource

Permanently remove a resource from the knowledge base.

**Request:**

```bash
curl -s -X DELETE \
  -H "Authorization: Bearer {api_key}" \
  "{endpoint}/resource/{rid}"
```

**Response:** 204 No Content on success.

**Safety:** Always confirm with the user before executing a delete. Show the resource title and ID and require explicit confirmation.

### PATCH `{endpoint}/resource/{rid}` — Update metadata and labels

Update a resource's title, labels, or other metadata.

**Request (add labels):**

```bash
curl -s -X PATCH \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "usermetadata": {
      "classifications": [
        {"labelset": "classification", "label": "security"}
      ]
    }
  }' \
  "{endpoint}/resource/{rid}"
```

**Request (update title):**

```bash
curl -s -X PATCH \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Title Here"
  }' \
  "{endpoint}/resource/{rid}"
```

**Response fields:**
- Updated resource metadata confirming the changes

### POST `{endpoint}/catalog` — Search resource metadata

Search across resource metadata and filter by processing status. Useful for health dashboards and resource discovery.

**Request:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": ""
  }' \
  "{endpoint}/catalog"
```

**With status filter:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "",
    "filters": {
      "status": "ERROR"
    }
  }' \
  "{endpoint}/catalog"
```

**Response fields:**
- `resources` — array of resource metadata objects
  - `id` — resource ID
  - `title` — resource title
  - `status` — `PROCESSED`, `PENDING`, or `ERROR`
  - `labels` — classification labels
  - `origin` — URL, path, dates, contributors

## Output Format

### Upload confirmation

```markdown
## Upload Successful

| Field | Value |
|-------|-------|
| Title | **[extracted title]** |
| Resource ID | `{rid}` |
| Status | PENDING — processing will complete shortly |

Check progress with `/arag:status`. Query this resource with `/arag:ask`.
```

### Resource list

```markdown
## Resources in [KB Name]

| # | Title | Status | Labels | Modified |
|---|-------|--------|--------|----------|
| 1 | **API Guide** | PROCESSED | security, api | 2026-03-15 |
| 2 | **Onboarding Deck** | PROCESSED | hr, onboarding | 2026-03-10 |
| 3 | **Q1 Report** | PENDING | — | 2026-03-20 |
| 4 | **Old Policy** | ERROR | compliance | 2026-01-05 |

Showing 4 of 4 resources. Page 1 of 1.
```

### Delete confirmation

```markdown
## Confirm Delete

You're about to permanently delete:
- **[Title]** (`{rid}`)

This cannot be undone. Proceed? (yes/no)
```

After confirmation:

```markdown
Deleted **[Title]** from [KB Name].
```

### Label update

```markdown
Updated labels for **[Title]**:
- Added: `security`, `compliance`
- Current labels: `security`, `compliance`, `api`
```

### KB health dashboard

```markdown
## KB Health: [KB Name]

### Summary
| Metric | Value |
|--------|-------|
| Total Resources | 47 |
| Processed | 44 |
| Processing | 2 |
| Errors | 1 |
| Last Updated | 2026-03-20 |

### Recent Activity
| # | Title | Status | Modified |
|---|-------|--------|----------|
| 1 | **New API Docs** | PENDING | 2026-03-20 |
| 2 | **Security Policy v3** | PROCESSED | 2026-03-19 |
| 3 | **Broken Import** | ERROR | 2026-03-18 |

Use `/arag:manage` to fix errors or upload new content.
```

## Execution Flow

1. **Read config** — load `arag/.claude/settings.local.json`. Missing? Run setup.
2. **Resolve KB** — use named KB if specified, otherwise default.
3. **Parse action** — determine intent: upload, delete, update, list, label, or status.
4. **Validate input** — check file paths exist, URLs are valid, resource names resolve.
5. **Resolve resource** — if action targets a resource by name, look it up via catalog or resource list to get the `rid`.
6. **Confirm destructive actions** — for delete operations, show what will be removed and require explicit confirmation.
7. **Call API** — execute the appropriate curl command.
8. **Check response** — handle success and error codes.
9. **Present results** — show confirmation, updated state, or error details.
10. **Suggest next steps** — `/arag:status` after upload, `/arag:ask` after successful processing, `/arag:manage list` after changes.

## Error Handling

| Error | Message | Action |
|-------|---------|--------|
| 401/403 | "Authentication failed" | Suggest running `/arag:setup` |
| 404 | "Resource not found" | Check resource ID/name, suggest `/arag:manage list` |
| 409 | "Resource already exists" | Suggest updating instead of creating, or use a different slug |
| 413 | "File too large" | Note the file size limit; suggest splitting or compressing |
| 422 | "Invalid request" | Check request format, show what was sent |
| Network error | "Can't reach API" | Check connection and endpoint |
| Processing error | "Resource failed to process" | Check `/arag:status` for details; suggest re-uploading |

## Related Skills

- `cited-answers` — query the KB after uploading new content
- `knowledge-base-search` — search to verify uploaded content is indexed
- `knowledge-graph` — browse entities extracted from uploaded resources
