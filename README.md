# ARAG

The ultimate Agentic RAG plugin for Cowork, Anthropic's agentic desktop application — also works in Claude Code. Query, manage, analyze, and train your knowledge bases with fully-cited AI answers where every claim traces back to a source document. Powered by Progress Agentic RAG.

## What is ARAG?

ARAG connects Claude to your company's knowledge bases. Instead of Claude answering from its general training data, it searches **your documents** and gives answers backed by **your sources** — with footnotes linking every claim to the exact document it came from.

Think of it as giving Claude access to your internal docs, wikis, policies, and research — and having it cite its sources every time.

## Prerequisites

Before installing, you need a **Progress Agentic RAG** Knowledge Box. You'll need two things from your Knowledge Box dashboard:

1. **Knowledge Box endpoint URL** — looks like `https://europe-1.rag.progress.cloud/api/v1/kb/your-kb-id`
2. **Knowledge Box API key** — generated from your Knowledge Box settings (a service access token scoped to your specific KB)

If you don't have a Knowledge Box yet, visit [docs.rag.progress.cloud](https://docs.rag.progress.cloud) to get started.

## Installation

### Cowork (Anthropic's Desktop App)

Cowork is Anthropic's desktop application for working with Claude. If you're using Cowork, follow these steps:

**Option A — Install from the plugin browser:**

1. Open Cowork
2. Click the **Customize** button (bottom-left of the sidebar)
3. Select **Browse plugins**
4. Search for **ARAG**
5. Click **Install**

**Option B — Install from a ZIP file (for custom or private plugins):**

1. Download or export the ARAG plugin as a `.zip` file
2. In Cowork, go to **Organization Settings** → **Plugins**
3. Click **Add plugins**
4. Upload the ZIP file (must be under 50 MB)

**Option C — Connect via GitHub (for automatic updates):**

1. In Cowork, go to **Organization Settings** → **Plugins**
2. Click **Add plugins** → **Connect GitHub repository**
3. Point it to the ARAG plugin repository

After installation, you'll see ARAG's commands available when you type `/` in the chat input.

### Claude Code (CLI)

If you're using Claude Code in your terminal:

```
claude plugins install progress/arag
```

## Getting started

### Step 1: Connect your knowledge base

After installing, type this in your Claude conversation:

```
/arag:setup
```

Claude will walk you through it interactively — just paste your endpoint URL and API key when prompted. The whole process takes under a minute.

### Step 2: Ask your first question

```
/arag:ask What is our data retention policy?
```

You'll get an AI-generated answer with inline footnote citations like `[^1]`, `[^2]`, plus a **Sources** section at the bottom mapping every footnote to the specific document it came from.

### Step 3: Explore further

```
/arag:search API rate limiting
/arag:sources deployment guides
/arag:status
```

- **`/arag:search`** — Find specific documents in your KB (no AI generation, just ranked results)
- **`/arag:sources`** — Browse what's in your KB
- **`/arag:status`** — See a health dashboard of your KB (total resources, processing status, errors)

That's it — you're up and running.

## Using ARAG in Cowork

In Cowork, there are two ways ARAG works:

### Slash commands (you trigger them)

Type `/` in the chat input to see available commands. All ARAG commands start with `/arag:`. Select one or type the full name:

```
/arag:ask How do webhooks work?
```

### Skills (Claude triggers them automatically)

Skills work behind the scenes. When you ask Claude a question that relates to your knowledge base, ARAG's skills activate automatically — you don't need to do anything. For example, if you ask Claude to fact-check a claim, the `source-verification` skill kicks in and cross-references your KB sources.

You'll know a skill is active when Claude starts citing your KB documents in its response.

## Commands

### Query

| Command | What it does |
|---------|-------------|
| `/arag:ask` | Ask a question, get a fully-cited AI answer from your KB |
| `/arag:search` | Search your KB for source documents — ranked results, no AI generation |
| `/arag:sources` | Browse what's in your KB — document titles, URLs, relevance scores |
| `/arag:focus` | Search within a specific document — drill into a single resource |
| `/arag:research` | Deep research — decompose a topic, synthesize across sources, full citation trail |

### Manage

| Command | What it does |
|---------|-------------|
| `/arag:manage` | Upload, delete, label, and update resources in your KB |
| `/arag:status` | KB health dashboard — resource counts, processing status, errors |
| `/arag:ingest` | Bulk upload documents from URLs or local files |

### Analyze

| Command | What it does |
|---------|-------------|
| `/arag:compare` | Compare answers from multiple KBs on the same question |
| `/arag:entities` | Browse extracted entities — people, organizations, products, and relationships |

### Configure

| Command | What it does |
|---------|-------------|
| `/arag:setup` | Connect to your knowledge base — provide endpoint URL and API key |
| `/arag:train` | Train custom classifiers and intent detection on your KB |

### Command examples

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

Skills activate automatically based on what you're doing — no slash command needed.

| Skill | When it activates |
|-------|-------------------|
| `cited-answers` | When you ask a knowledge base question — generates answers with footnote citations |
| `knowledge-base-search` | When you're browsing or looking for sources — hybrid semantic + keyword search |
| `research-synthesis` | When you ask a broad or complex question — decomposes into sub-questions and cross-references |
| `source-verification` | When you're fact-checking — verifies claims as supported, contradicted, or not found |
| `kb-management` | When you're managing KB content — handles uploads, deletions, labels, and monitoring |
| `knowledge-graph` | When you're exploring entities — extracts people, organizations, products, and relationships |

## Read-write operations

ARAG isn't just for reading — you can manage your knowledge base directly from Claude:

- **Upload** files, URLs, or text content
- **Delete** outdated or errored resources
- **Label** and organize resources with classification tags
- **Bulk ingest** entire directories or URL lists
- **Monitor** processing status and KB health
- **Train** custom classifiers on your KB content

## Multiple knowledge bases

You can connect as many KBs as you need. Run `/arag:setup` again to add another. One KB is always marked as the default — used when you don't specify a name.

Target a specific KB by putting its name in brackets:

```
/arag:ask [Engineering Wiki] How does the build pipeline work?
/arag:search [Product Docs] authentication flow
/arag:research [All KBs] What is our AI strategy?
```

## How citations work

Every answer includes inline Markdown footnotes (`[^1]`, `[^2]`) that link back to specific paragraphs in your source documents. At the end of every answer, a **Sources** section shows the document title, URL, and a relevant snippet for each footnote.

Claims that can't be backed by a source are flagged, so you always know what's KB-backed and what isn't.

## Configuration details

Configuration is saved to `arag/.claude/settings.local.json` after running `/arag:setup`. This file is local and not committed to version control. API keys are stored in plaintext — secure your machine accordingly.

Every command and skill works standalone — no MCP connectors or external tools are needed.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "No knowledge base configured" | Run `/arag:setup` to connect your KB |
| 401 or 403 error | Your API key is invalid or expired — generate a new one from your Knowledge Box settings |
| 404 error | Your endpoint URL is wrong — make sure it contains `/api/v1/kb/` and ends with your KB ID |
| No results returned | Your KB may be empty or still processing — run `/arag:status` to check |
| Plugin not showing up in Cowork | Make sure you've installed it (Customize → Browse plugins) and restart Cowork if needed |

## Learn more

- [Progress Agentic RAG documentation](https://docs.rag.progress.cloud)
- [Cowork plugin guide](https://support.claude.com/en/articles/13837440-use-plugins-in-cowork)
