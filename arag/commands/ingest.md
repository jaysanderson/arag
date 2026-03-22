---
name: ingest
description: Bulk upload documents to your knowledge base from URLs or local files.
argument-hint: "<url1> <url2> ... or <file-glob>"
---

# ARAG Ingest

Bulk upload to your knowledge base. Feed in a list of URLs, file paths, or a glob pattern and let Claude handle the uploads one by one with a full progress report.

## Usage

```
/arag:ingest https://docs.example.com/page1 https://docs.example.com/page2 https://docs.example.com/page3
/arag:ingest /path/to/docs/*.pdf
/arag:ingest /path/to/reports/q1.pdf /path/to/reports/q2.pdf
```

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB

Same logic as other commands — named KB if specified, otherwise default.

### 3. Parse input

Determine what the user wants to upload:

| Input type | Detection |
|-----------|-----------|
| URLs | Starts with `http://` or `https://` |
| File paths | Starts with `/` or `./` or `~` |
| Glob pattern | Contains `*`, `?`, or `[` |

For glob patterns, expand to the list of matching files using bash:

```bash
ls /path/to/docs/*.pdf
```

### 4. Confirm with the user

Before uploading, show what will be uploaded:

```
## Ready to Ingest

About to upload **[N] resources** to **[KB Name]**:

| # | Resource | Type |
|---|----------|------|
| 1 | https://docs.example.com/page1 | URL |
| 2 | https://docs.example.com/page2 | URL |
| 3 | /path/to/report.pdf | File |

Proceed? (yes/no)
```

### 5. Upload each resource

**For URLs:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "page1",
    "origin": {
      "url": "https://docs.example.com/page1"
    }
  }' \
  "{endpoint}/resources"
```

**For files:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "X-FILENAME: report.pdf" \
  -F "file=@/path/to/report.pdf" \
  "{endpoint}/upload"
```

Track the result of each upload (success or failure with error details).

### 6. Present the ingestion report

```
## Ingestion Report for [KB Name]

| # | Resource | Status | Details |
|---|----------|--------|---------|
| 1 | page1 | Uploaded | rid: `a1b2...` |
| 2 | page2 | Uploaded | rid: `c3d4...` |
| 3 | report.pdf | Failed | 413: File too large |

---
**Uploaded: 2 of 3 successful.** Resources are now processing.

Check status with `/arag:status`. Failed uploads can be retried individually with `/arag:manage upload`.
```

### 7. Handle errors

Per-item errors do NOT stop the batch. Track each result independently.

Common per-item errors:
- **413**: File too large — note the file and suggest splitting or compressing.
- **409**: Resource already exists — note it and continue.
- **Network timeout**: Note the failure and continue with remaining items.
- **Unsupported format**: Note the file type and continue.

Batch-level errors:
- **401/403**: "Authentication failed. Run `/arag:setup` to check your API key." (Stop the batch — all items will fail.)
- **404**: "Knowledge base not found." (Stop the batch.)

## Example

```
You: /arag:ingest https://docs.example.com/api-v2 https://docs.example.com/security https://docs.example.com/compliance

Claude:
## Ready to Ingest

About to upload **3 resources** to **Product Docs**:

| # | Resource | Type |
|---|----------|------|
| 1 | https://docs.example.com/api-v2 | URL |
| 2 | https://docs.example.com/security | URL |
| 3 | https://docs.example.com/compliance | URL |

Proceed? (yes/no)

You: yes

Claude:
## Ingestion Report for Product Docs

| # | Resource | Status | Details |
|---|----------|--------|---------|
| 1 | api-v2 | Uploaded | rid: `a1b2c3d4...` |
| 2 | security | Uploaded | rid: `e5f6g7h8...` |
| 3 | compliance | Uploaded | rid: `i9j0k1l2...` |

**Uploaded: 3 of 3 successful.** Resources are now processing.

Check status with `/arag:status`. Query your new content with `/arag:ask`.
```
