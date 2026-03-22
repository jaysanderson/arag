# Connectors

## This plugin works standalone

The ARAG plugin connects directly to Progress Agentic RAG knowledge bases via REST API. No MCP connectors are needed.

## Configuration

You only need two things:

1. **Knowledge Base endpoint URL** — from your ARAG dashboard (e.g., `https://europe-1.rag.progress.cloud/api/v1/kb/your-kb-id`)
2. **API key** — a NUA key from your ARAG account management (starts with `nua-`)

Run `/arag:setup` to configure.

## Multiple knowledge bases

You can connect as many knowledge bases as you need. Run `/arag:setup` again to add more. Target a specific KB in any command:

```
/arag:ask [Product Docs] How does ingestion work?
/arag:search [Engineering Wiki] deployment patterns
```

## Learn more

Visit [docs.rag.progress.cloud](https://docs.rag.progress.cloud) for Progress Agentic RAG documentation.
