---
name: research
description: Deep research across your knowledge base — decompose a topic, synthesize findings, full citation trail.
argument-hint: "<topic or research question>"
---

# ARAG Research

Deep research mode. Give Claude a broad topic and get a structured research brief with full citations, cross-referenced sources, and identified gaps.

## Usage

```
/arag:research Our approach to data security and compliance
/arag:research [Product Docs] How does our pricing model compare across tiers?
/arag:research What do our docs say about API rate limiting?
```

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB(s)

- If a specific KB is named, use that one.
- If the user says "search everything" or "all KBs", query ALL configured knowledge bases.
- Otherwise, use the default KB.

If multiple KBs are configured and the topic might benefit from cross-KB analysis, suggest: "You have [N] KBs configured. Want a quick side-by-side comparison first? Try `/arag:compare [topic]`. Otherwise, I'll do deep research on the default KB."

### 3. Decompose the topic

Break the user's broad topic into 3–5 targeted sub-questions. For example:

**Topic:** "Our approach to data security and compliance"
**Sub-questions:**
1. What encryption methods do we use for data at rest and in transit?
2. What compliance certifications do we hold (SOC 2, GDPR, HIPAA)?
3. What is our access control and authentication model?
4. What are our data retention and deletion policies?
5. How do we handle security incidents and breach notification?

Tell the user what sub-questions you're investigating.

### 4. Run API calls for each sub-question

For each sub-question, POST to `{endpoint}/ask`:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -H "x-synchronous: true" \
  -d '{
    "query": "sub-question here",
    "citations": "llm_footnotes",
    "context": []
  }' \
  "{endpoint}/ask"
```

If querying multiple KBs, run the same sub-question against each endpoint.

**Conversation context across sub-questions:** When later sub-questions relate to earlier ones, pass the accumulated context from previous answers into the `context` array. This helps the ARAG API provide more focused answers that build on prior findings:

```json
{
  "query": "sub-question 3 here",
  "citations": "llm_footnotes",
  "context": [
    {"author": "USER", "text": "sub-question 1"},
    {"author": "NUCLIA", "text": "answer to sub-question 1"},
    {"author": "USER", "text": "sub-question 2"},
    {"author": "NUCLIA", "text": "answer to sub-question 2"}
  ]
}
```

### 5. Cross-reference and deduplicate

- Collect all sources across sub-question responses.
- Deduplicate: if the same document/paragraph appears in multiple answers, merge into a single citation.
- Note when different sources agree (strengthens confidence).
- Flag conflicts: if Source A says X and Source B says Y, call it out explicitly.

### 6. Produce the research brief

Structure the output as:

```markdown
# Research Brief: [Topic]

## Executive Summary

[2-3 paragraph synthesis of key findings across all sub-questions. High-level
takeaways. Note confidence level based on citation coverage.]

## Findings

### [Theme 1: e.g. Encryption & Data Protection]

[Synthesized findings from relevant sub-questions. Inline footnotes preserved.]

### [Theme 2: e.g. Compliance Certifications]

[Synthesized findings. Inline footnotes.]

### [Theme 3: e.g. Access Control]

[Synthesized findings. Inline footnotes.]

## Source Conflicts

[If any sources contradict each other, document them here:]
- **[Topic]**: Source A ([title]) states X, while Source B ([title]) states Y.

## Knowledge Gaps

[Topics the KB doesn't cover or covers thinly:]
- The knowledge base has limited information on [topic].
- No sources were found for [specific sub-question].

## All Sources

[^1]: **[Title]** — [snippet] — [URL]
[^2]: **[Title]** — [snippet] — [URL]
...
```

### 7. Handle partial results

- If some sub-questions return empty results, include them in "Knowledge Gaps" — don't silently skip.
- If all sub-questions fail, report the issue and suggest the user check their KB content or try `/arag:search` to explore what's available.

## Tips

- Broader topics produce richer briefs. "Data security" is better than "AES-256 key length."
- Multi-KB research shines when you have separate KBs for different domains (e.g., engineering docs + compliance docs).
- Follow up with `/arag:ask` to dig deeper into any specific finding.
- Use `/arag:compare` for a quick side-by-side when comparing across KBs.
- Use `/arag:focus "[Document]" [topic]` to drill into a specific source.
- Use `/arag:entities` to explore people, organizations, and other entities mentioned in your research.
