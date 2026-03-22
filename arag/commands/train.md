---
name: train
description: Train custom classifiers and intent detection models on your Knowledge Box.
argument-hint: "<action> [options]"
---

# ARAG Train

Train custom machine learning models on your Knowledge Box content. Create classifiers, train intent detection, and build custom entity recognition — all from within Claude.

## Usage

```
/arag:train create-classifier "Support Ticket Type" bug feature question
/arag:train add-examples "Support Ticket Type" bug "The app crashes when I click submit"
/arag:train start "Support Ticket Type"
/arag:train status "Support Ticket Type"
/arag:train test "Support Ticket Type" "I can't log in to my account"
```

## Prerequisites

This command uses the same Knowledge Box API key configured via `/arag:setup`. No additional configuration is needed.

## Execution Flow

### 1. Load configuration

Read `arag/.claude/settings.local.json`. If missing, run `/arag:setup` first.

### 2. Parse the action

| Action | Description |
|--------|-------------|
| **create-classifier** | Define a new classifier with label categories |
| **add-examples** | Add labeled training examples to a classifier |
| **start** | Trigger model training |
| **status** | Check training progress |
| **test** | Run inference on new text |
| **list** | Show all configured classifiers |

### 3. Create a classifier

Define the classification task and its labels:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Support Ticket Type",
    "labels": ["bug", "feature", "question", "other"]
  }' \
  "{endpoint}/classifier"
```

**Output:**

```
## Classifier Created

| Field | Value |
|-------|-------|
| Name | **Support Ticket Type** |
| Labels | `bug`, `feature`, `question`, `other` |
| Status | CREATED — needs training examples |

Add examples: `/arag:train add-examples "Support Ticket Type" <label> "<text>"`
```

### 4. Add training examples

Provide labeled text examples for each category:

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "classifier": "Support Ticket Type",
    "examples": [
      {"text": "The app crashes when I click submit", "label": "bug"},
      {"text": "Can you add dark mode?", "label": "feature"},
      {"text": "How do I reset my password?", "label": "question"}
    ]
  }' \
  "{endpoint}/classifier/train"
```

**Output:**

```
Added 3 training examples to **Support Ticket Type**:
- `bug`: 1 example (total: 5)
- `feature`: 1 example (total: 3)
- `question`: 1 example (total: 7)

Recommend at least 10 examples per label before training. Run `/arag:train start` when ready.
```

### 5. Start training

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "classifier": "Support Ticket Type"
  }' \
  "{endpoint}/classifier/start"
```

**Output:**

```
## Training Started

| Field | Value |
|-------|-------|
| Classifier | **Support Ticket Type** |
| Training examples | 42 across 4 labels |
| Status | TRAINING |

Check progress: `/arag:train status "Support Ticket Type"`
```

### 6. Check status

```bash
curl -s -X GET \
  -H "Authorization: Bearer {api_key}" \
  "{endpoint}/classifier/Support%20Ticket%20Type/status"
```

**Output:**

```
## Training Status: Support Ticket Type

| Field | Value |
|-------|-------|
| Status | READY |
| Accuracy | 94.2% |
| Labels | 4 |
| Training examples | 42 |
| Last trained | 2026-03-20 |

Test it: `/arag:train test "Support Ticket Type" "your text here"`
```

### 7. Test / run inference

```bash
curl -s -X POST \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "classifier": "Support Ticket Type",
    "text": "I can'\''t log in to my account"
  }' \
  "{endpoint}/classifier/predict"
```

**Output:**

```
## Classification Result

**Input:** "I can't log in to my account"

| Label | Confidence |
|-------|------------|
| **bug** | 72.3% |
| question | 24.1% |
| feature | 2.8% |
| other | 0.8% |

**Predicted: bug** (72.3% confidence)
```

### 8. Handle errors

- **401/403**: "Authentication failed. Run `/arag:setup` to check your API key."
- **404**: "Classifier not found. Run `/arag:train list` to see available classifiers."
- **409**: "Classifier already exists. Use a different name or delete the existing one."
- **Insufficient examples**: "Need at least [N] examples per label to start training."
- **Training failed**: "Training failed. Check your examples for quality and label balance."
- **Network error**: "Couldn't reach the API. Check your internet connection."

**Note:** The training API endpoints documented here reflect the expected API surface. Verify exact endpoint paths against the current [Progress Agentic RAG documentation](https://docs.rag.progress.cloud) as they may evolve.

## Example

```
You: /arag:train test "Support Ticket Type" "The export button does nothing when I click it"

Claude:
## Classification Result

**Input:** "The export button does nothing when I click it"

| Label | Confidence |
|-------|------------|
| **bug** | 89.1% |
| question | 7.2% |
| feature | 3.1% |
| other | 0.6% |

**Predicted: bug** (89.1% confidence)
```
