---
title: "Session-to-Mintlify Pipeline"
description: "Agent workflow for automatically compiling Claude conversations into Mintlify docs and deploying via GitHub"
icon: "arrows-rotate"
---

## Why

Manually harvesting Claude conversations into published docs is the bottleneck. Conversation content is already structured thinking — the gap between "I explained this to Claude" and "it's on the docs site" should not require hours of editing.

The pipeline collapses that gap into three stages: **extract → transform → sync**.

## Architecture

### Stage 1: Session Capture

Claude has no official conversation export API yet. Three viable inputs:

- **Manual paste**: Copy conversation to `.txt` or `.json`. Fine for one-off imports
- **API-native**: If you're using the Claude API directly, the `messages[]` array is already structured — feed it to the extractor as-is
- **Browser extension**: Tools like "Save ChatGPT"-class extensions scrape claude.ai and emit JSON

### Stage 2: Extractor Agent

A Claude API call with a system prompt that parses conversation into structured documents. The core prompt concept:

```
You are a technical documentation extractor. Given a conversation, identify:
- Each distinct topic (one Mintlify page each)
- Title, description, body sections
- Code blocks, tables, step lists
- Category (edge/cloud/processing/storage/blog/)

Output strict JSON:
{
  "pages": [
    {
      "path": "cloud/kafka-broker",
      "title": "...",
      "description": "...",
      "sections": [...]
    }
  ]
}
```

The JSON output then runs through an MDX Writer function that converts sections into `.mdx` files with appropriate Mintlify components (`<Steps>`, `<Card>`, `<Warning>`, etc.).

### Stage 3: GitHub Sync

The GitHub Contents API lets you create or update files with a single PUT request:

```python
import base64, requests

def push_to_github(path, content, token, repo):
    url = f"https://api.github.com/repos/{repo}/contents/docs/{path}"
    existing = requests.get(
        url,
        headers={"Authorization": f"token {token}"}
    ).json()
    sha = existing.get("sha")  # required when updating existing files

    requests.put(url, json={
        "message": f"docs: sync {path} from Claude session",
        "content": base64.b64encode(content.encode()).decode(),
        "sha": sha  # omit for new files
    }, headers={"Authorization": f"token {token}"})
```

The PUT triggers the GitHub App webhook to Mintlify, which runs a build and deploys. No manual intervention required between "conversation ended" and "page is live."

## Orchestrator Options

Three directions with different complexity trade-offs:

| Approach | When to Use | Complexity |
|---|---|---|
| **Python script** | First-pass validation, manual trigger | Low |
| **n8n** (self-hosted) | Visual flow, no-code triggers (cron, webhooks) | Medium |
| **LangGraph** | Complex agent chains, retry logic, conditional routing ("is this blog or docs?") | High |

**Recommended path**: start with a Python script to validate the full flow end-to-end, then move to n8n only if you need scheduled runs or webhook triggers. Don't reach for LangGraph until the simpler options hit a wall — premature agent complexity is a common trap.

## Open Design Questions

- **Extraction granularity**: how to split a conversation that covers multiple topics? One page per subtopic vs. one page per conversation?
- **Human review**: draft PR mode (commit to a branch, open a PR) vs. direct push to main? The former is safer but adds friction
- **Version management**: when the same topic comes up in multiple conversations, merge or overwrite?
- **Component selection**: what heuristic decides `<Steps>` vs. plain numbered list, or `<Warning>` vs. plain blockquote?

## Key Risks

- Extractor classification accuracy directly determines navigation usability — a miscategorized page is worse than no page
- Conversational signal-to-noise is lower than curated drafts, so human review cannot be skipped for public-facing content
- GitHub API rate limits matter for batch syncs (5000 requests/hour for authenticated users, but the bottleneck is actually Mintlify build queue)
