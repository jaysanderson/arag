---
name: entities
description: Browse extracted entities and relationships in your knowledge base — people, organizations, products, and more.
argument-hint: "<entity name or type>"
---

# ARAG Entities

Explore the people, organizations, products, regulations, and other entities that Progress Agentic RAG has automatically extracted from your documents. See what your KB knows about — and how things connect.

## Usage

```
/arag:entities
/arag:entities people
/arag:entities organizations
/arag:entities "John Smith"
/arag:entities [Product Docs] GDPR
```

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Resolve KB

Same logic as other commands — named KB if specified, otherwise default.

### 3. Determine intent

Parse what the user wants:

| Input | Action |
|-------|--------|
| No argument | Show entity overview — top entities by type |
| Entity type (e.g., "people", "organizations") | Show all entities of that type |
| Entity name (e.g., "John Smith") | Show detail view for that entity |
| General topic | Search for entities related to the topic |

### 4. Query for entities

**For overview or type filter — search broadly:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": ""
  }' \
  "{endpoint}/find"
```

Extract entities from the `relations` field in the response. Group by type, count mentions.

**For specific entity — targeted search:**

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "John Smith"
  }' \
  "{endpoint}/find"
```

Collect all paragraphs mentioning the entity, group by source document.

### 5. Present results

**Entity overview:**

```
## Entities in [KB Name]

### People
| Entity | Mentions | Documents |
|--------|----------|-----------|
| John Smith | 15 | 4 |
| Jane Doe | 8 | 2 |

### Organizations
| Entity | Mentions | Documents |
|--------|----------|-----------|
| Acme Corp | 23 | 7 |

### Regulations
| Entity | Mentions | Documents |
|--------|----------|-----------|
| GDPR | 18 | 5 |
| SOC 2 | 14 | 3 |

Showing top entities. Use `/arag:entities <type>` to see more.
```

**Entity detail:**

```
## Entity: John Smith (PERSON)

Mentioned in 4 documents (15 times):

| # | Document | Mentions | Relevance |
|---|----------|----------|-----------|
| 1 | **Team Structure** | 6 | ★★★★★ |
| 2 | **API Design Review** | 4 | ★★★★☆ |
| 3 | **Q1 Planning** | 3 | ★★★☆☆ |

Key contexts:
> "John Smith leads the API platform team..." — Team Structure
> "Reviewed by John Smith on March 15..." — API Design Review
```

### 6. Offer next steps

> "Want to dig deeper?"
> - `/arag:ask What is John Smith responsible for?` — get a cited answer about this entity
> - `/arag:focus "Team Structure" John Smith` — search within a specific document
> - `/arag:research [topic] across all mentions` — deep research on the entity

### 7. Handle errors

Same error handling as other commands. Additionally:
- **No entities found**: "No entities extracted yet. Your KB may still be processing — check `/arag:status`."
- **Entity not found**: "No mentions of '[name]' found in [KB Name]. Try a different spelling or use `/arag:search` instead."

## Example

```
You: /arag:entities people

Claude:
## People in Product Docs

| Entity | Mentions | Documents |
|--------|----------|-----------|
| John Smith | 15 | 4 |
| Jane Doe | 8 | 2 |
| Alex Chen | 5 | 3 |
| Sarah Williams | 3 | 1 |

Found 4 people mentioned across 6 documents.

Want to dig deeper?
- `/arag:entities "John Smith"` — see all mentions and contexts
- `/arag:ask What is John Smith's role?` — get a cited answer
```
