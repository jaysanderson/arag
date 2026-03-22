---
name: source-verification
description: Verify claims against a Progress Agentic RAG knowledge base — check if statements are supported, contradicted, or absent from your sources.
---

# Source Verification

Fact-check claims against your knowledge base. Give Claude a statement and it will tell you whether it's supported, contradicted, or simply not covered by your sources. Useful for content review, compliance checking, QA, and audit trails.

## How It Works

```
┌─────────────────────────────────────────────┐
│          SOURCE VERIFICATION                │
├─────────────────────────────────────────────┤
│                                             │
│  ALWAYS (standalone)                        │
│  ✓ Claim verification against KB sources    │
│  ✓ Supported / Contradicted / Not Found     │
│  ✓ Source citations for each verdict        │
│  ✓ Batch verification of multiple claims    │
│  ✓ Confidence assessment                    │
│                                             │
└─────────────────────────────────────────────┘
```

## Getting Started

```
"Can you verify that our API supports rate limiting of 1000 requests per minute?"
"Check these claims against our knowledge base: [list of claims]"
"Is it true that we offer HIPAA compliance?"
```

This skill activates when the user asks to verify, fact-check, or confirm a specific claim against the knowledge base.

## Configuration

Same as other skills. Reads `arag/.claude/settings.local.json`. If missing, prompt through setup.

## API Reference

Uses `POST {endpoint}/ask` with the claim framed as a verification query:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -H "x-synchronous: true" \
  -d '{
    "query": "Does the API support rate limiting of 1000 requests per minute?",
    "citations": "llm_footnotes",
    "context": []
  }' \
  "{endpoint}/ask"
```

Also uses `POST {endpoint}/find` for direct source search when needed:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "API rate limiting requests per minute"
  }' \
  "{endpoint}/find"
```

## Execution Flow

### 1. Extract claims

From the user's input, identify distinct verifiable claims. A claim is a specific factual statement that can be checked against sources.

**Example:** "Our platform uses AES-256 encryption, supports HIPAA compliance, and processes documents in under 5 seconds."

Claims:
1. The platform uses AES-256 encryption.
2. The platform supports HIPAA compliance.
3. The platform processes documents in under 5 seconds.

### 2. Verify each claim

For each claim:

**Step A — Ask the KB**

POST to `{endpoint}/ask` with a verification-framed query:
- "Does [the system] use AES-256 encryption?"
- Frame the claim as a question for the ARAG API.

**Step B — Search for direct evidence**

POST to `{endpoint}/find` with keywords from the claim:
- Search for "AES-256 encryption" to find direct mentions in source documents.

**Step C — Assess the evidence**

Compare the claim against what the API returned:

| Verdict | Criteria |
|---------|----------|
| ✅ **Supported** | The KB answer and/or source paragraphs directly confirm the claim, with citations. |
| ❌ **Contradicted** | The KB answer and/or source paragraphs state something that conflicts with the claim. |
| ⚠️ **Not Found** | The KB has no relevant information — the claim can't be verified either way. |
| 🔶 **Partially Supported** | The KB supports part of the claim but not all of it (e.g., "uses encryption" is supported but "AES-256" specifically is not mentioned). |

### 3. Present the verification report

## Output Format

### Single claim

```markdown
## Verification: "[the claim]"

**Verdict: ✅ Supported**

The knowledge base confirms this claim. [Brief explanation of what the sources say.]

**Source:** [^1]: **[Document Title]** — "[relevant quote from source]"
  [URL]
```

### Multiple claims (batch)

```markdown
## Verification Report

| # | Claim | Verdict | Source |
|---|-------|---------|--------|
| 1 | AES-256 encryption | ✅ Supported | Security Architecture[^1] |
| 2 | HIPAA compliance | ✅ Supported | Compliance Guide[^2] |
| 3 | Processing under 5 seconds | ⚠️ Not Found | — |

### Details

**1. "The platform uses AES-256 encryption"** — ✅ Supported
[Explanation with citation]

**2. "The platform supports HIPAA compliance"** — ✅ Supported
[Explanation with citation]

**3. "Documents processed in under 5 seconds"** — ⚠️ Not Found
The knowledge base does not contain information about processing speed benchmarks.
Consider adding performance documentation to the KB.

## Sources

[^1]: **Security Architecture** — [snippet] — [URL]
[^2]: **Compliance Guide** — [snippet] — [URL]
```

### Confidence levels

Add confidence context to each verdict:

- **High confidence**: Multiple sources agree, direct and specific match to the claim.
- **Moderate confidence**: One source supports it, or the match is indirect.
- **Low confidence**: Tangential match only — the KB covers the topic but doesn't directly address the specific claim.

## Use Cases

### Content review
"Before we publish this blog post, verify these technical claims against our docs."

### Compliance checking
"Check if our marketing page claims match what our compliance docs actually say."

### QA and audit
"Verify that our API documentation accurately describes the current rate limits."

### Onboarding materials
"Are the claims in our onboarding deck supported by our internal knowledge base?"

## Error Handling

Same as other skills — authentication errors, network issues, empty results.

For verification specifically:
- If the KB returns no results at all, the verdict is **⚠️ Not Found** — not ❌ Contradicted.
- Absence of evidence is not evidence of contradiction.
- Always distinguish between "the KB says something different" (contradicted) and "the KB doesn't say anything about this" (not found).

## Related Skills

- `cited-answers` — for getting answers to questions (not verifying claims)
- `knowledge-base-search` — for finding source documents directly
- `research-synthesis` — for deep research on broad topics
