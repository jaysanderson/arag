---
name: manage
description: Upload, delete, label, and update resources in your knowledge base.
argument-hint: "<action> [options]"
---

# ARAG Manage

Full resource management for your knowledge base. Upload new content, remove outdated documents, organize with labels, and update metadata — without leaving Claude.

## Usage

```
/arag:manage upload https://docs.example.com/api-guide
/arag:manage upload /path/to/document.pdf
/arag:manage list
/arag:manage delete "Outdated Policy v1"
/arag:manage label "API Guide" add security
/arag:manage label "API Guide" remove draft
/arag:manage update "API Guide" title "API Reference v2"
```

The optional `[KB Name]` syntax works here too: `/arag:manage [Engineering Wiki] list`.

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB

Same logic as other commands — named KB if specified, otherwise default.

### 3. Parse the action

Determine what the user wants to do from their input:

| Action | Trigger patterns |
|--------|-----------------|
| **upload** | "upload", "add", "import", a URL, or a file path |
| **list** | "list", "show", "browse", "what's in" |
| **delete** | "delete", "remove", "drop" |
| **label** | "label", "tag", "classify", "categorize" |
| **update** | "update", "rename", "change", "set" |

### 4. Upload a resource

**From URL:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "API Guide",
    "origin": {
      "url": "https://docs.example.com/api-guide"
    }
  }' \
  "{endpoint}/resources"
```

**From local file:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "X-FILENAME: document.pdf" \
  -F "file=@/path/to/document.pdf" \
  "{endpoint}/upload"
```

**From text content:**

If the user pastes text or asks to create a resource from content in the conversation:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "User-provided title",
    "texts": {
      "text": {
        "body": "The text content here..."
      }
    }
  }' \
  "{endpoint}/resources"
```

**Output after upload:**

```
## Upload Successful

| Field | Value |
|-------|-------|
| Title | **[title]** |
| Resource ID | `{rid}` |
| Status | PENDING |

Processing will complete shortly. Check with `/arag:status` or query with `/arag:ask`.
```

### 5. List resources

```bash
curl -s -X GET \
  -H "Authorization: Bearer {api_key}" \
  "{endpoint}/resources?page=0&size=20"
```

**Output:**

```
## Resources in [KB Name]

| # | Title | Status | Labels | Modified |
|---|-------|--------|--------|----------|
| 1 | **API Guide** | PROCESSED | security, api | 2026-03-15 |
| 2 | **Onboarding Deck** | PROCESSED | hr | 2026-03-10 |
| 3 | **Q1 Report** | PENDING | — | 2026-03-20 |

Showing 3 of 3 resources. Page 1 of 1.
```

If more than 20 resources, offer to show the next page.

### 6. Delete a resource

**Step 1 — Resolve the resource.** If the user gave a name, search for it:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{"query": "resource name here"}' \
  "{endpoint}/catalog"
```

Match by title (case-insensitive). If multiple matches, list them and ask the user to pick.

**Step 2 — Confirm.** Always ask before deleting:

```
## Confirm Delete

You're about to permanently delete:
- **[Title]** (`{rid}`)

This cannot be undone. Proceed? (yes/no)
```

**Step 3 — Delete.**

```bash
curl -s -X DELETE \
  -H "Authorization: Bearer {api_key}" \
  "{endpoint}/resource/{rid}"
```

**Output after delete:**

```
Deleted **[Title]** from [KB Name].
```

### 7. Manage labels

**Add a label:**

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

**Output:**

```
Updated labels for **[Title]**:
- Added: `security`
- Current labels: `security`, `api`, `compliance`
```

### 8. Update metadata

**Update title:**

```bash
curl -s -X PATCH \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "New Title Here"
  }' \
  "{endpoint}/resource/{rid}"
```

**Output:**

```
Updated **[Old Title]** → **[New Title]**
```

### 9. Handle errors

- **401/403**: "Authentication failed. Run `/arag:setup` to check your API key."
- **404**: "Resource not found. Run `/arag:manage list` to see available resources."
- **409**: "A resource with that identifier already exists. Try updating it instead."
- **413**: "File too large for upload. Try compressing or splitting the file."
- **Network error**: "Couldn't reach the ARAG API. Check your internet connection and endpoint URL."

## Example

```
You: /arag:manage upload https://docs.example.com/security-whitepaper

Claude:
## Upload Successful

| Field | Value |
|-------|-------|
| Title | **Security Whitepaper 2026** |
| Resource ID | `a1b2c3d4-5678-90ab-cdef-1234567890ab` |
| Status | PENDING |

Processing will complete shortly. Check with `/arag:status` or query with `/arag:ask`.
```
