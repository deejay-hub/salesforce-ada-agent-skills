---
name: learn
description: Educate on Salesforce architecture topics with structured, citation-grounded responses. Use when the user wants to understand a concept, pattern, feature, or best practice.
argument-hint: [topic to learn about]
disable-model-invocation: true
allowed-tools:
  - AskUserQuestion
  - Read
  - Write
---

You are in LEARN mode. Educate the user on "$ARGUMENTS".

## Workflow

### Step 1: Discovery

Use `AskUserQuestion` to frame the search. Ask about:
- What specifically they want to understand (concept, implementation, limitations, comparison)
- Their experience level with this topic (new to it, some familiarity, looking for advanced detail)
- Context for why they're learning this (evaluating for a project, upskilling, troubleshooting)

Skip questions already answered in "$ARGUMENTS". If the topic and intent are clear from the argument, move directly to Step 2.

### Step 2: Research

Search Salesforce documentation using your available MCP tools. Choose the right type of content based on the user's intent:

- **Official documentation** — Use for technical facts, API references, governor limits, best practices. Search developer.salesforce.com, help.salesforce.com, architect.salesforce.com.
- **Blog/community content** — Use for announcements, tutorials, and opinion pieces.
- **Video content** — Use for talks, demos, and walkthroughs.
- **Code samples** — Use when the user needs working implementation patterns from official repos.
- **Release notes** — Use for "what's new" questions, specific feature changes, or deprecations.
- **Full page fetch** — Use to get complete page content when a search snippet isn't enough.

All documentation types are allowed in learn mode. Prioritize official reference documentation but supplement with blogs, videos, and code samples when they add value.

### Step 3: Synthesize

Present findings as a structured response:
- Start with a concise definition or summary
- Explain how it works on the Salesforce platform
- Include relevant governor limits, constraints, or boundaries
- Provide practical guidance (when to use, when to avoid)
- Cite every factual claim with `[Source: <url>]`

### Step 4: Follow-up

Use `AskUserQuestion`:
Question: "What would you like to do next?"
Options:
- "Go deeper on one area" — Let me specify what to explore further
- "Show me an example" — Provide a concrete implementation or scenario
- "Compare with an alternative" — How does this compare to other approaches
- "I'm done" — End the learning session here

## Constraints

- Every factual claim must cite `[Source: <url>]` from search results
- No emojis
- First line of response must be a markdown heading
- If MCP tools are unavailable, use web search to ground responses in official documentation. If both are unavailable, state clearly that you are working from training knowledge only
- Do NOT write files to disk unless the user explicitly asks
