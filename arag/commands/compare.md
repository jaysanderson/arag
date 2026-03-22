---
name: compare
description: Compare answers from multiple knowledge bases on the same question.
argument-hint: "<question>"
---

# ARAG Compare

Ask the same question across multiple knowledge bases and see the answers side by side. Spot agreements, contradictions, and unique coverage across your KBs.

## Usage

```
/arag:compare How do we handle data encryption?
/arag:compare [Product Docs] vs [Engineering Wiki] What is our deployment process?
```

Without specifying KBs, this command queries all configured knowledge bases.

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KBs to compare

- If the user specifies `[KB1] vs [KB2]`, compare those two.
- If the user says "all KBs" or doesn't specify, compare all configured KBs.
- **Minimum 2 KBs required.** If only one is configured:

> "You only have one knowledge base configured. Add another with `/arag:setup` to use compare, or use `/arag:ask` for single-KB queries."

### 3. Query each KB

For each KB, POST to `{endpoint}/ask` with the same question:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -H "x-synchronous: true" \
  -d '{
    "query": "the question here",
    "citations": "llm_footnotes",
    "context": []
  }' \
  "{endpoint}/ask"
```

Run against each KB endpoint sequentially. Collect answers and sources from each.

### 4. Analyze results

Compare the answers:

- **Agreements**: Both KBs confirm the same facts → note as high confidence.
- **Contradictions**: KB1 says X, KB2 says Y on the same topic → flag explicitly.
- **Unique coverage**: Facts only found in one KB → note which KB has the exclusive information.
- **Coverage strength**: Rate each KB's answer as strong, moderate, or limited based on citation count and specificity.

### 5. Present the comparison

```
## Comparison: "[question]"

*Queried [N] knowledge bases.*

### [KB Name 1]

[Answer text with footnotes preserved]

### [KB Name 2]

[Answer text with footnotes preserved]

---

### Analysis

| Aspect | [KB1] | [KB2] |
|--------|-------|-------|
| Coverage | Strong (4 citations) | Moderate (2 citations) |
| Focus | Technical implementation | Policy and compliance |

**Agreements:**
- Both KBs confirm [finding].

**Contradictions:**
- [KB1] states [X], while [KB2] states [Y]. [Note on which may be more current.]

**Unique to [KB1]:**
- [Finding only in KB1]

**Unique to [KB2]:**
- [Finding only in KB2]

## All Sources

**From [KB1]:**
[^1]: **[Title]** — [snippet] — [URL]
[^2]: **[Title]** — [snippet] — [URL]

**From [KB2]:**
[^3]: **[Title]** — [snippet] — [URL]
[^4]: **[Title]** — [snippet] — [URL]
```

### 6. Handle partial failures

- If one KB returns an error but others succeed, show the successful results and note the failure:

> "Could not reach **[KB Name]**: [error]. Showing results from [N-1] knowledge bases."

- If all KBs fail, report the error and suggest `/arag:setup`.

### 7. Offer next steps

> "Want to dig deeper?"
> - `/arag:research [topic]` — deep multi-step research across KBs
> - `/arag:focus "[Document]" [query]` — drill into a specific source
> - `/arag:ask [KB Name] [follow-up]` — ask a follow-up to a specific KB

## Example

```
You: /arag:compare How do we handle authentication?

Claude:
## Comparison: "How do we handle authentication?"

*Queried 2 knowledge bases.*

### Product Docs

Our platform uses OAuth 2.0 for API authentication[^1]. All API requests require
a bearer token obtained through the authorization code flow[^1]. We also support
API keys for server-to-server communication[^2].

### Engineering Wiki

Authentication is handled by the Auth Service, which implements OAuth 2.0 with
PKCE for public clients[^3]. Session tokens expire after 24 hours and refresh
tokens after 30 days[^3]. MFA is required for all admin accounts[^4].

---

### Analysis

| Aspect | Product Docs | Engineering Wiki |
|--------|-------------|------------------|
| Coverage | Moderate (2 citations) | Strong (2 citations) |
| Focus | User-facing API auth | Internal implementation |

**Agreements:**
- Both confirm OAuth 2.0 as the authentication framework.

**Unique to Product Docs:**
- API key support for server-to-server communication.

**Unique to Engineering Wiki:**
- PKCE for public clients, token expiration details, MFA requirement.

## All Sources

**From Product Docs:**
[^1]: **API Authentication Guide** — "OAuth 2.0 bearer tokens..." — https://...
[^2]: **API Key Management** — "Server-to-server API keys..." — https://...

**From Engineering Wiki:**
[^3]: **Auth Service Design Doc** — "OAuth 2.0 with PKCE..." — https://...
[^4]: **Security Policies** — "MFA required for admin..." — https://...
```
