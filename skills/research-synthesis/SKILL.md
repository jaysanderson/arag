---
name: research-synthesis
description: Deep research across a Progress Agentic RAG knowledge base — decompose topics, synthesize findings, cross-reference sources, identify gaps.
---

# Research Synthesis

Deep research mode. Take a broad topic, decompose it into targeted sub-questions, run each against the knowledge base, cross-reference sources, and produce a structured research brief with full citations.

## How It Works

```
┌─────────────────────────────────────────────┐
│           RESEARCH SYNTHESIS                │
├─────────────────────────────────────────────┤
│                                             │
│  ALWAYS (standalone)                        │
│  ✓ Topic decomposition into sub-questions   │
│  ✓ Multiple API calls with citations        │
│  ✓ Cross-KB research (if multiple KBs)      │
│  ✓ Source deduplication                     │
│  ✓ Conflict detection between sources       │
│  ✓ Gap identification                       │
│  ✓ Structured research brief output         │
│                                             │
└─────────────────────────────────────────────┘
```

## Getting Started

```
/arag:research Our approach to data security and compliance
/arag:research How does our product handle multi-tenancy?
/arag:research [All KBs] What is our company's AI strategy?
```

This skill activates when the question is broad enough that a single API call won't cover it — topics that span multiple documents or domains.

## Configuration

Same as other skills. Reads `arag/.claude/settings.local.json`. If missing, prompt through setup.

## API Reference

Uses the same `POST {endpoint}/ask` endpoint as `cited-answers`, called multiple times:

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

Each sub-question gets its own API call. If researching across multiple KBs, each sub-question runs against each KB endpoint.

## Execution Flow

### 1. Read config and resolve KB(s)

Load settings. Determine target KBs:
- Specific named KB → use that one
- "All KBs" or "search everything" → use all configured KBs
- No specification → use default KB only

### 2. Decompose the topic

Break the broad topic into 3–5 focused sub-questions. Guidelines:

- Each sub-question should target a distinct aspect of the topic
- Sub-questions should be specific enough for the ARAG API to return focused results
- Cover the topic breadth — don't cluster sub-questions around one narrow angle
- Frame as questions, not keywords

**Example decomposition:**

Topic: "Our approach to data security and compliance"

1. What encryption methods do we use for data at rest and in transit?
2. What compliance certifications do we hold (SOC 2, GDPR, HIPAA)?
3. What is our access control and authentication model?
4. What are our data retention and deletion policies?
5. How do we handle security incidents and breach notification?

**Tell the user** what sub-questions you're investigating before making API calls.

### 3. Run API calls

For each sub-question:
- POST to `{endpoint}/ask` with `citations: "llm_footnotes"` and `x-synchronous: true`
- Collect the `answer`, `retrieval_results`, and `footnote_to_context`
- Track which sub-question produced which sources
- Pass accumulated context from earlier sub-questions into later ones via the `context` array, so later sub-questions benefit from earlier findings

If using multiple KBs, run each sub-question against each KB. Label results by KB name.

### 4. Cross-reference and deduplicate

After all calls complete:

- **Deduplicate sources**: Same document/paragraph cited in multiple answers? Merge into a single citation entry. Track all the sub-questions that referenced it.
- **Cross-reference agreement**: When multiple sources support the same finding, note it — higher confidence.
- **Detect conflicts**: If Source A says X and Source B says Y on the same topic, flag as a conflict.
- **Assign unified footnote numbers**: Renumber all footnotes sequentially for the final brief.

### 5. Identify gaps

Review the sub-questions against the results:

- Did any sub-question return empty results? → Knowledge gap.
- Did any sub-question return low-relevance results only? → Weak coverage.
- Are there obvious aspects of the topic not covered by any sub-question? → Suggest additional research.

### 6. Produce the research brief

## Output Format

```markdown
# Research Brief: [Topic]

*Researched across [N] sub-questions in [KB name(s)]. [M] sources cited.*

## Executive Summary

[2-3 paragraphs synthesizing the key findings across all sub-questions. This
should read as a standalone summary. Note overall confidence level based on
citation coverage — strong, moderate, or limited.]

## Findings

### [Theme 1: descriptive name]

[Synthesized narrative combining relevant findings from multiple sub-questions.
Inline footnotes preserved: [^1], [^2], etc. Each paragraph should flow
naturally while maintaining citation traceability.]

### [Theme 2: descriptive name]

[Continue with next theme. Group by topic, not by sub-question — the user
cares about themes, not about your research methodology.]

### [Theme 3: descriptive name]

[Additional themes as needed.]

## Source Conflicts

[Only include this section if conflicts exist.]

- **[Topic]**: [Source A title][^N] states [X], while [Source B title][^M]
  states [Y]. [Brief note on which may be more current or authoritative.]

## Knowledge Gaps

[Always include this section — even if only to note that coverage is comprehensive.]

- [Topic or sub-question with no or weak results]
- [Aspect of the broader topic not covered in the KB]

## All Sources

[^1]: **[Document Title]** — [short snippet] — [URL]
[^2]: **[Document Title]** — [short snippet] — [URL]
[^3]: **[Document Title]** — [short snippet] — [URL]
...
```

### Key principles for the brief

- **Organize by theme, not by sub-question.** The user asked about a topic, not about your methodology.
- **Synthesize, don't concatenate.** Weave findings together into coherent narratives.
- **Every claim gets a footnote.** If it came from the KB, cite it. If it's your synthesis, say so.
- **Conflicts are valuable.** Don't hide them — they tell the user where to dig deeper.
- **Gaps are valuable.** They tell the user what's missing from their knowledge base.

## Compare Mode

When the user wants to contrast what different KBs say rather than merge findings, suggest `/arag:compare` for a quick side-by-side comparison. Research synthesis is best for deep analysis; compare is best for surface-level differences.

If the user explicitly wants both depth and comparison, run the full research flow across all KBs and highlight inter-KB conflicts prominently in the Source Conflicts section.

## Multi-KB Research

When researching across multiple KBs:

- Run each sub-question against all KBs
- Label results by KB name in the sources: `[^1]: **[Title]** (from Product Docs) — ...`
- Note in the executive summary which KBs contributed what
- Conflicts between KBs are especially interesting — highlight them

## Error Handling

- If one sub-question fails but others succeed, include results from successful calls and note the failure.
- If all calls fail, report the error and suggest checking credentials with `/arag:setup`.
- If a specific KB is unreachable in multi-KB mode, skip it and note which KB was unavailable.

## Related Skills

- `cited-answers` — for single focused questions (not broad research)
- `knowledge-base-search` — for browsing sources without synthesis
- `source-verification` — for verifying specific claims against the KB
- `kb-management` — for managing the resources that research draws from
- `knowledge-graph` — for exploring entities and relationships across research findings
