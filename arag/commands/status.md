---
name: status
description: Knowledge base health dashboard — resource counts, processing status, and recent activity.
argument-hint: "[KB Name]"
---

# ARAG Status

See the health of your knowledge base at a glance. Total resource count, processing status breakdown, recent activity, and any errors that need attention.

## Usage

```
/arag:status
/arag:status [Engineering Wiki]
```

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB

Same logic as other commands — named KB if specified, otherwise default.

### 3. Get resource catalog

Fetch the full resource list to compute statistics:

```bash
curl -s -X GET \
  -H "Authorization: Bearer {api_key}" \
  "{endpoint}/resources?page=0&size=100"
```

If the KB has more than 100 resources, paginate to get all:

```bash
curl -s -X GET \
  -H "Authorization: Bearer {api_key}" \
  "{endpoint}/resources?page=1&size=100"
```

### 4. Check for errors

Query the catalog for resources with error status:

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

### 5. Present the dashboard

```
## KB Health: [KB Name]

### Summary

| Metric | Value |
|--------|-------|
| Total Resources | 47 |
| Processed | 44 |
| Processing | 2 |
| Errors | 1 |
| Endpoint | `https://europe-1.rag.progress.cloud/api/v1/kb/...` |

### Errors (if any)

| # | Title | Resource ID | Modified |
|---|-------|-------------|----------|
| 1 | **Broken Import** | `abc123...` | 2026-03-18 |

### Recent Activity

| # | Title | Status | Modified |
|---|-------|--------|----------|
| 1 | **New API Docs** | PENDING | 2026-03-20 |
| 2 | **Security Policy v3** | PROCESSED | 2026-03-19 |
| 3 | **User Guide v2** | PROCESSED | 2026-03-18 |
| 4 | **Broken Import** | ERROR | 2026-03-18 |
| 5 | **Release Notes** | PROCESSED | 2026-03-17 |
```

Show the 5 most recently modified resources in the Recent Activity table.

### 6. Offer next steps

After the dashboard:

> "Actions you can take:"
> - `/arag:manage delete "Broken Import"` — remove errored resources
> - `/arag:manage upload <file-or-url>` — add new content
> - `/arag:ask` — query the knowledge base

### 7. Handle errors

Same error handling as other commands.

### 8. Multi-KB summary

If the user has multiple KBs configured and doesn't specify one, show a summary across all KBs:

```
## All Knowledge Bases

| # | KB Name | Resources | Processed | Errors | Default |
|---|---------|-----------|-----------|--------|---------|
| 1 | **Product Docs** | 47 | 44 | 1 | Yes |
| 2 | **Engineering Wiki** | 23 | 23 | 0 | — |
| 3 | **Compliance Docs** | 12 | 11 | 1 | — |

Run `/arag:status [KB Name]` for details on a specific KB.
```

## Example

```
You: /arag:status

Claude:
## KB Health: Product Docs

### Summary

| Metric | Value |
|--------|-------|
| Total Resources | 47 |
| Processed | 44 |
| Processing | 2 |
| Errors | 1 |

### Errors

| # | Title | Resource ID | Modified |
|---|-------|-------------|----------|
| 1 | **Broken Import** | `a1b2c3d4...` | 2026-03-18 |

### Recent Activity

| # | Title | Status | Modified |
|---|-------|--------|----------|
| 1 | **New API Docs** | PENDING | 2026-03-20 |
| 2 | **Security Policy v3** | PROCESSED | 2026-03-19 |
| 3 | **Broken Import** | ERROR | 2026-03-18 |

Actions:
- `/arag:manage delete "Broken Import"` — remove the errored resource
- `/arag:manage upload <file>` — add new content
- `/arag:ask` — query the knowledge base
```
