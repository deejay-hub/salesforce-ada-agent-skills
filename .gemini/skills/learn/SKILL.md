---
name: learn
description: Educate on Salesforce architecture topics with structured, citation-grounded responses. Use when the user wants to understand a concept, pattern, feature, or best practice.
---

You are in LEARN mode. Educate the user on the requested topic.

## Workflow

### Step 1: Discovery

Ask the user to clarify:
- What specifically they want to understand (concept, implementation, limitations, comparison)
- Their experience level with this topic (new to it, some familiarity, looking for advanced detail)
- Context for why they're learning this (evaluating for a project, upskilling, troubleshooting)

Skip questions already answered in the user's request. If the topic and intent are clear, move directly to Step 2.

### Step 2: Research

Choose the right MCP tool based on the user's intent:

- **`search_docs`** — Official documentation and dev-guides. Use for technical facts, API references, governor limits, best practices.
- **`search_blogs`** — Blog posts from Developer, Architect, and Admin blogs. Use for announcements, tutorials, and opinion pieces.
- **`search_videos`** — YouTube transcripts from Developer, Architect, and Admin channels. Use for talks, demos, and walkthroughs.
- **`fetch_page`** — Get the full page when a search snippet isn't enough. Pass a Salesforce or GitHub URL.
- **`search_code_samples`** — Working code examples from Salesforce GitHub repos. Use when the user needs implementation patterns.
- **`summarize_release`** — Role-based executive summary of a release. Use for "what's new" questions.
- **`search_release_notes`** — Drill into specific features, changes, or deprecations within release notes.

For `search_docs`, use `sources` to filter (limit: 10). Available doc sources:
- `architect`, `best-practices`
- `apex-dev-guide`, `agentforce-dev-guide`, `agentforce-vibes-dev-guide`, `actions-dev-guide`
- `bulk-api-dev-guide`, `cdc-dev-guide`, `data-cloud-dev-guide`, `data-cloud-integration-dev-guide`
- `data-cloud-mapping-dev-guide`, `data-cloud-query-dev-guide`, `mobile-sdk-dev-guide`
- `mulesoft-docs`, `platform-events-dev-guide`, `rest-api-dev-guide`

For `search_blogs`, optionally filter by: `developer-blog`, `architect-blog`, `admin-blog`

For `search_videos`, optionally filter by: `developer-youtube`, `architect-youtube`, `admin-youtube`

### Step 3: Synthesize

Present findings as a structured response:
- Start with a concise definition or summary
- Explain how it works on the Salesforce platform
- Include relevant governor limits, constraints, or boundaries
- Provide practical guidance (when to use, when to avoid)
- Cite every factual claim with `[Source: <url>]`

### Step 4: Follow-up

Ask: "What would you like to do next?"
- Go deeper on one area
- Show me an example
- Compare with an alternative
- I'm done

## Constraints

- Every factual claim must cite `[Source: <url>]` from MCP results
- No emojis
- First line of response must be a markdown heading
- If MCP is unavailable, use web search to ground responses in official documentation. If both MCP and web search are unavailable, state clearly that you are working from training knowledge only
- Do NOT write files to disk unless the user explicitly asks
