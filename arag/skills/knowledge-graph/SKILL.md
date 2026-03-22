---
name: knowledge-graph
description: Browse extracted entities and knowledge graph relationships from a Progress Agentic RAG knowledge base — people, organizations, products, dates, and their connections.
---

# Knowledge Graph

Explore the entities and relationships that Progress Agentic RAG automatically extracts from your documents. Browse people, organizations, products, dates, and more — plus see how they connect to each other.

## How It Works

```
┌─────────────────────────────────────────────┐
│           KNOWLEDGE GRAPH                   │
├─────────────────────────────────────────────┤
│                                             │
│  ALWAYS (standalone)                        │
│  ✓ Browse 16+ extracted entity types        │
│  ✓ Search entities by name or type          │
│  ✓ View entity relationships                │
│  ✓ See which documents mention each entity  │
│  ✓ Entity-based knowledge base queries      │
│  ✓ Relationship mapping                     │
│                                             │
└─────────────────────────────────────────────┘
```

## Getting Started

```
/arag:entities
/arag:entities people
/arag:entities "John Smith"
/arag:entities [Product Docs] organizations
```

This skill activates when Claude detects questions about people, organizations, concepts, or relationships mentioned across the knowledge base.

## Configuration

Same as other skills. Reads `arag/.claude/settings.local.json`. If missing, prompt through setup.

## Supported Entity Types

Progress Agentic RAG automatically extracts these entity types during document processing:

| Type | Examples |
|------|----------|
| **PERSON** | John Smith, Jane Doe |
| **ORG** | Acme Corp, Progress Software |
| **PLACE** | San Francisco, Europe |
| **DATE** | March 2026, Q1 2025 |
| **PRODUCT** | NucliaDB, Agentic RAG |
| **MONEY** | $1M, 500K EUR |
| **EVENT** | Annual Summit, Sprint Review |
| **LAW** | GDPR, HIPAA, SOC 2 |
| **LANGUAGE** | Python, JavaScript |
| **PERCENT** | 99.9%, 15% |
| **QUANTITY** | 1000 requests, 5 seconds |
| **TIME** | 5pm EST, business hours |
| **WORK_OF_ART** | API Guide v2, Security Whitepaper |
| **FAC** | Building A, Data Center US-East |
| **NORP** | Engineering Team, Compliance Group |
| **GPE** | United States, European Union |

## API Reference

### POST `{endpoint}/find` — Entity-enriched search

The standard search endpoint returns entity annotations in results. Use targeted queries to find entity mentions:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "John Smith"
  }' \
  "{endpoint}/find"
```

Entity mentions appear in the `relations` field of search results, containing:
- `entity` — the entity name
- `entity_type` — the type (PERSON, ORG, etc.)
- `positions` — where in the text the entity appears

### POST `{endpoint}/ask` — Entity-aware Q&A

Ask questions about entities and their relationships:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -H "x-synchronous: true" \
  -d '{
    "query": "What is John Smith responsible for?",
    "citations": "llm_footnotes",
    "context": []
  }' \
  "{endpoint}/ask"
```

The response includes:
- `answer` — generated text about the entity with citations
- `relations` — extracted entity relations from the response
- `retrieval_results` — source paragraphs mentioning the entity

### GET `{endpoint}/search?query=...` — Separate result sets with entities

Returns full-text, fuzzy, and semantic matches separately — useful for seeing all entity mentions:

```bash
curl -s -G \
  -H "Authorization: Bearer {api_key}" \
  --data-urlencode "query=Acme Corp" \
  "{endpoint}/search"
```

## Output Format

### Entity overview

```markdown
## Entities in [KB Name]

*Extracted from [N] processed resources.*

### People
| Entity | Mentions | Documents |
|--------|----------|-----------|
| John Smith | 15 | 4 |
| Jane Doe | 8 | 2 |
| Alex Chen | 5 | 3 |

### Organizations
| Entity | Mentions | Documents |
|--------|----------|-----------|
| Acme Corp | 23 | 7 |
| Progress Software | 12 | 4 |

### Regulations & Standards
| Entity | Mentions | Documents |
|--------|----------|-----------|
| GDPR | 18 | 5 |
| SOC 2 | 14 | 3 |
| HIPAA | 9 | 2 |

---
Showing top entities by mention count. Use `/arag:entities <type>` to see all entities of a specific type.
```

### Entity detail

```markdown
## Entity: John Smith (PERSON)

**Mentioned in 4 documents (15 times):**

| # | Document | Mentions | Relevance |
|---|----------|----------|-----------|
| 1 | **Team Structure** | 6 | ★★★★★ |
| 2 | **API Design Review** | 4 | ★★★★☆ |
| 3 | **Q1 Planning** | 3 | ★★★☆☆ |
| 4 | **Security Audit** | 2 | ★★☆☆☆ |

**Key contexts:**
> "John Smith leads the API platform team and is responsible for..." — Team Structure
> "Reviewed by John Smith on March 15..." — API Design Review

Want to learn more? Try:
- `/arag:ask What is John Smith responsible for?`
- `/arag:focus "Team Structure" John Smith`
```

### Relationship view

```markdown
## Relationships

John Smith --[leads]--> API Platform Team
John Smith --[reports to]--> Jane Doe
Jane Doe --[manages]--> Engineering Division
Acme Corp --[certified for]--> SOC 2
Acme Corp --[compliant with]--> GDPR
```

## Execution Flow

1. **Read config** — load `arag/.claude/settings.local.json`. Missing? Run setup.
2. **Resolve KB** — use named KB if specified, otherwise default.
3. **Parse intent** — determine: browse all entities, filter by type, search by name, or query relationships.
4. **Query the KB** — use `/find` for entity search, `/ask` for entity-related questions.
5. **Extract entities** — parse entity annotations from response `relations` field.
6. **Aggregate** — group entities by type, count mentions, track source documents.
7. **Deduplicate** — merge variant spellings (e.g., "J. Smith" and "John Smith") where possible.
8. **Rank** — sort by mention count within each type.
9. **Present results** — show entity tables, detail views, or relationship maps.
10. **Suggest next steps** — offer `/arag:ask` for entity Q&A, `/arag:focus` for document drill-down.

## Error Handling

| Error | Message | Action |
|-------|---------|--------|
| 401/403 | "Authentication failed" | Suggest running `/arag:setup` |
| 404 | "KB not found" | Check endpoint URL, suggest `/arag:setup` |
| No entities | "No entities extracted yet" | KB may still be processing; check `/arag:status` |
| Entity not found | "No mentions of [name] found" | Suggest broader search or different spelling |
| Network error | "Can't reach API" | Check connection and endpoint |

## Related Skills

- `cited-answers` — ask questions about specific entities with full citations
- `knowledge-base-search` — find documents mentioning specific entities
- `source-verification` — verify claims about entities against KB sources
- `kb-management` — manage the resources that entities are extracted from
