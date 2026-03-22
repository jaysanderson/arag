---
name: setup
description: Connect Claude to your Progress Agentic RAG knowledge base. Provide your endpoint URL and API key.
---

# ARAG Setup

Connect Claude to your Progress Agentic RAG knowledge base in under a minute.

## What You Need

Two things:

1. **Knowledge Base endpoint URL** — from your ARAG dashboard
2. **API key** — a NUA key from your ARAG account management

## Setup Flow

### Step 1: Check for existing configuration

Read `arag/.claude/settings.local.json`. If it exists and contains `knowledge_bases`, tell the user:

> "You already have [N] knowledge base(s) configured: [list names]. Want to add another KB, replace one, or start fresh?"

If the user wants to add another, continue below and append to the array. If replace, ask which one to replace. If start fresh, overwrite the file.

### Step 2: Ask for the endpoint URL

Ask the user:

> "What's your Knowledge Base endpoint URL?"
>
> Copy this from your ARAG dashboard. It looks like:
> `https://europe-1.rag.progress.cloud/api/v1/kb/your-kb-id`

Validate that the URL:
- Starts with `https://`
- Contains `/api/v1/kb/`
- Ends with what looks like a UUID

If it doesn't match, ask the user to double-check.

### Step 3: Ask for the API key

Ask the user:

> "What's your API key?"
>
> Go to your ARAG account management, open NUA Keys, create or copy a key. It starts with `nua-`.

Validate that the key starts with `nua-`. If not, let the user know the expected format.

### Step 4: Ask for a friendly name (optional)

> "Give this KB a friendly name? (e.g. 'Product Docs', 'Engineering Wiki')"
>
> Press Enter to skip — defaults to 'My Knowledge Base'.

### Step 5: Test the connection

Run a test API call using bash `curl`:

```bash
curl -s -o /dev/null -w "%{http_code}" \
  -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{"query": "test"}' \
  "{endpoint}/find"
```

- **200**: Connection successful.
- **401/403**: Bad API key. Ask the user to double-check.
- **404**: Bad endpoint URL. Ask the user to double-check.
- **Other**: Report the status code and ask the user to verify their details.

### Step 6: Save configuration

Write to `arag/.claude/settings.local.json`:

```json
{
  "knowledge_bases": [
    {
      "name": "Product Docs",
      "endpoint": "https://europe-1.rag.progress.cloud/api/v1/kb/df8b4c24-2807-4888-ad6c-ae97357a638b",
      "api_key": "nua-xxxxxxxxxxxxxxxx",
      "default": true
    }
  ],
  "preferred_citation_mode": "llm_footnotes"
}
```

If this is an additional KB, set `"default": false` unless the user asks to make it the default. Only one KB should have `"default": true`.

### Step 7: Confirm

> "Connected to **[name]**! Try `/arag:ask` with a question."

If multiple KBs are configured:

> "You now have [N] knowledge bases. **[default name]** is your default. Use `/arag:ask [KB Name] your question` to target a specific one."

## Notes

- The settings file is local and not committed to version control.
- API keys are stored in plaintext — the user is responsible for securing their machine.
- If any other ARAG command is run before setup, detect the missing config and run this setup flow first.
