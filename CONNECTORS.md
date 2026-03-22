# Connectors

## This plugin works standalone

The ARAG plugin connects directly to Progress Agentic RAG knowledge bases via REST API. No MCP connectors are needed.

## Configuration

You only need two things:

1. **Knowledge Base endpoint URL** — from your ARAG dashboard (e.g., `https://europe-1.rag.progress.cloud/api/v1/kb/your-kb-id`)
2. **API key** — a NUA key from your ARAG account management (starts with `nua-`)

Run `/arag:setup` to configure.

## Full read-write access

The plugin supports both reading and writing to your knowledge bases:

- **Query**: Ask questions, search, browse sources, deep research, entity exploration
- **Manage**: Upload files/URLs, delete resources, add/remove labels, update metadata
- **Analyze**: Compare across KBs, verify claims, explore knowledge graphs
- **Train**: Custom classifiers and intent detection via the NUA API

## Multiple knowledge bases

You can connect as many knowledge bases as you need. Run `/arag:setup` again to add more. Target a specific KB in any command:

```
/arag:ask [Product Docs] How does ingestion work?
/arag:search [Engineering Wiki] deployment patterns
/arag:compare [Product Docs] vs [Engineering Wiki] How do we handle auth?
```

Use `/arag:compare` to get side-by-side answers from different KBs on the same question.

## NUA API integration

The `/arag:train` command uses the Progress NUA (Nuclia Understanding API) for custom model training. This uses the same `nua-` API key — no additional configuration needed. See [NUA API docs](https://docs.rag.progress.cloud/docs/nua-api/) for details.

## Learn more

Visit [docs.rag.progress.cloud](https://docs.rag.progress.cloud) for Progress Agentic RAG documentation.
