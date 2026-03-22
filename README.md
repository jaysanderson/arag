# ARAG

The ultimate Agentic RAG plugin for Cowork, Anthropic's agentic desktop application — also works in Claude Code. Query, manage, analyze, and train your knowledge bases with fully-cited AI answers where every claim traces back to a source document. Powered by Progress Agentic RAG.

## Installation

```
claude plugins install progress/arag
```

## Quick start

1. Install the plugin
2. Run `/arag:setup` — provide your KB endpoint URL and API key
3. `/arag:status` — see your KB health at a glance
4. `/arag:ask "How does document ingestion work?"`

That's it. You'll get an AI-generated answer with inline footnote citations, plus a Sources section mapping every footnote to its source document.

## Commands

### Query

| Command | Description |
|---------|-------------|
| `/arag:ask` | Ask a question, get a fully-cited AI answer from your KB |
| `/arag:search` | Search your KB for source documents — ranked results, no AI generation |
| `/arag:sources` | Browse what's in your KB — document titles, URLs, relevance scores |
| `/arag:focus` | Search within a specific document — drill into a single resource |
| `/arag:research` | Deep research — decompose a topic, synthesize across sources, full citation trail |

### Manage

| Command | Description |
|---------|-------------|
| `/arag:manage` | Upload, delete, label, and update resources in your KB |
| `/arag:status` | KB health dashboard — resource counts, processing status, errors |
| `/arag:ingest` | Bulk upload documents from URLs or local files |

### Analyze

| Command | Description |
|---------|-------------|
| `/arag:compare` | Compare answers from multiple KBs on the same question |
| `/arag:entities` | Browse extracted entities — people, organizations, products, and relationships |

### Configure

| Command | Description |
|---------|-------------|
| `/arag:setup` | Connect to your knowledge base — provide endpoint URL and API key |
| `/arag:train` | Train custom classifiers and intent detection via NUA API |

### Examples

```
/arag:ask What is our data retention policy?
/arag:ask [Product Docs] How do webhooks work?
/arag:search API rate limiting
/arag:research Our approach to security and compliance
/arag:sources deployment guides
/arag:focus "Security Whitepaper" encryption at rest
/arag:manage upload https://docs.example.com/new-guide
/arag:status
/arag:compare How do we handle authentication?
/arag:entities people
/arag:ingest /path/to/docs/*.pdf
```

## Skills

| Skill | Description |
|-------|-------------|
| `cited-answers` | Generates AI answers with inline footnote citations — auto-activates for KB Q&A |
| `knowledge-base-search` | Hybrid semantic + keyword search — auto-activates for source browsing |
| `research-synthesis` | Decomposes broad topics into sub-questions, cross-references sources — auto-activates for research tasks |
| `source-verification` | Verifies claims against KB sources (supported / contradicted / not found) — auto-activates for fact-checking |
| `kb-management` | Resource CRUD, label management, processing monitoring — auto-activates for KB management tasks |
| `knowledge-graph` | Entity extraction, relationship mapping, entity-based queries — auto-activates for entity exploration |

Every command and skill works standalone — no MCP connectors needed.

## Read-Write Operations

The ARAG plugin supports full read-write operations on your knowledge base:

- **Upload** files, URLs, or text content directly from Claude
- **Delete** outdated or errored resources
- **Label** and organize resources with classification tags
- **Bulk ingest** entire directories or URL lists
- **Monitor** processing status and KB health
- **Train** custom classifiers on your KB content

## Configuration

You need two things from your Progress Agentic RAG account:

1. **Knowledge Base endpoint URL** — from your ARAG dashboard (e.g., `https://europe-1.rag.progress.cloud/api/v1/kb/your-kb-id`)
2. **API key** — a NUA key from account management (starts with `nua-`)

Run `/arag:setup` and Claude walks you through it. Configuration is saved to `arag/.claude/settings.local.json`.

### Multiple knowledge bases

Connect as many KBs as you need. Run `/arag:setup` again to add another. Target any KB by name:

```
/arag:ask [Engineering Wiki] How does the build pipeline work?
/arag:research [All KBs] What is our AI strategy?
```

One KB is always marked as the default — used when you don't specify a name.

## How citations work

The plugin uses Progress Agentic RAG's `llm_footnotes` citation mode. Every answer includes inline Markdown footnotes (`[^1]`, `[^2]`) that trace back through the API's `footnote_to_context` mapping to specific source paragraphs. The Sources section at the end of every answer shows document title, URL, and a snippet for each footnote.

Claims without citations are flagged so you know what's KB-backed and what isn't.

## Learn more

Visit [docs.rag.progress.cloud](https://docs.rag.progress.cloud) for Progress Agentic RAG documentation.
