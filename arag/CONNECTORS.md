# Connectors

## This plugin works standalone

The ARAG plugin connects directly to Progress Agentic RAG knowledge bases via REST API. No MCP connectors are needed.

## Configuration

You only need two things:

1. **Knowledge Box endpoint URL** — from your KB settings (e.g., `https://europe-1.rag.progress.cloud/api/v1/kb/your-kb-id`)
2. **Knowledge Box API key** — a service access token generated from your KB settings (scoped to your specific Knowledge Box)

Run `/arag:setup` to configure.

## Full read-write access

The plugin supports both reading and writing to your knowledge bases:

- **Query**: Ask questions, search, browse sources, deep research, entity exploration
- **Manage**: Upload files/URLs, delete resources, add/remove labels, update metadata
- **Analyze**: Compare across KBs, verify claims, explore knowledge graphs
- **Train**: Custom classifiers and intent detection on your Knowledge Box

## Multiple knowledge bases

You can connect as many knowledge bases as you need. Run `/arag:setup` again to add more. Target a specific KB in any command:

```
/arag:ask [Product Docs] How does ingestion work?
/arag:search [Engineering Wiki] deployment patterns
/arag:compare [Product Docs] vs [Engineering Wiki] How do we handle auth?
```

Use `/arag:compare` to get side-by-side answers from different KBs on the same question.

## Training API

The `/arag:train` command uses the Knowledge Box training endpoints for custom model training. This uses the same KB API key — no additional configuration needed. See [Progress Agentic RAG docs](https://docs.rag.progress.cloud) for details.

## Learn more

Visit [docs.rag.progress.cloud](https://docs.rag.progress.cloud) for Progress Agentic RAG documentation.
