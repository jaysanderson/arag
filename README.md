# ARAG Marketplace

A Claude Code plugin marketplace containing the **ARAG** plugin — the ultimate Agentic RAG plugin for Cowork and Claude Code. Powered by Progress Agentic RAG.

## Installation

### Claude Code (CLI)

Add this marketplace, then install the plugin:

```bash
claude plugins marketplace add progress/arag
claude plugins install arag
```

### Cowork (Desktop App)

**Option A — Plugin browser:**

1. Open Cowork
2. Click **Customize** (bottom-left of sidebar)
3. Select **Browse plugins**
4. Search for **ARAG** and click **Install**

**Option B — Connect via GitHub:**

1. Go to **Organization Settings** → **Plugins**
2. Click **Add plugins** → **Connect GitHub repository**
3. Point it to this repository

## Getting started

After installation:

```
/arag:setup
```

Paste your Knowledge Box endpoint URL and API key when prompted. Then:

```
/arag:ask What is our data retention policy?
```

See the [plugin README](arag/README.md) for full documentation.

## What's included

| Plugin | Description |
|--------|-------------|
| **arag** | 12 slash commands + 6 automatic skills for querying, managing, analyzing, and training your Knowledge Boxes with fully-cited AI answers |

## Learn more

- [Progress Agentic RAG documentation](https://docs.rag.progress.cloud)
- [Cowork plugin guide](https://support.claude.com/en/articles/13837440-use-plugins-in-cowork)
