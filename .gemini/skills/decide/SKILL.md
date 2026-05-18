---
name: decide
description: Generate a Decision Record through structured analysis pipeline. Use when comparing architectural options, choosing between technical approaches, evaluating build-vs-buy trade-offs, or making any design decision that needs evidence-based scoring.
---

You are in DECIDE mode. Guide the user through a structured decision pipeline.

## Pipeline

Execute these steps IN ORDER. Pause for user input between each step. Do not skip steps.

### Step 1: Discovery

Gather context by asking the user about:
- Business problem or technical challenge
- Key numbers (record volumes, user count, transaction frequency)
- Salesforce products, editions, and existing customizations
- Delivery timeline
- Budget, compliance, or technical constraints

Skip questions already answered in the user's request. Do NOT call MCP tools during discovery.

### Step 2: Solution Options

1. Call `search_docs` (ONE call, limit: 10) for relevant approaches and documentation. Restrict `sources` to dev-guides and architecture content only:
   - **Always include:** `architect`, `best-practices`
   - **Include when relevant:** `apex-dev-guide`, `agentforce-dev-guide`, `agentforce-vibes-dev-guide`, `actions-dev-guide`, `bulk-api-dev-guide`, `cdc-dev-guide`, `data-cloud-dev-guide`, `data-cloud-integration-dev-guide`, `data-cloud-mapping-dev-guide`, `data-cloud-query-dev-guide`, `mobile-sdk-dev-guide`, `mulesoft-docs`, `platform-events-dev-guide`, `rest-api-dev-guide`
   - **Never include:** `admin-blog`, `architect-blog`, `developer-blog`, `admin-youtube`, `architect-youtube`, `developer-youtube`
2. Propose 3-4 options

Read `knowledge/formats/options-format.md` for the exact output format. Follow it precisely.

Ask the user to confirm options before proceeding.

### Step 3: Assessment Criteria

Generate 6-10 assessment criteria from MCP results.

Read `knowledge/formats/criteria-format.md` for the exact output format.

Ask the user to confirm criteria before proceeding.

### Step 4: Risk Assessment

1. Generate 3-5 differentiating questions. Ask the user these to gather context that will inform risk ratings.
2. Rate each option against each criterion.

Read `knowledge/formats/risk-matrix-format.md` for the exact output format.

Ask the user to confirm the risk matrix before proceeding.

### Step 5: Decision Score

1. Read `knowledge/scoring/decision-score-methodology.md` now. Map each criterion from the risk matrix to its most relevant pillar. If multiple criteria map to one pillar, use the worst (highest-risk) rating. Score each pillar using the risk-to-score bands, then calculate the weighted decision score.
2. Ask the user which output format they prefer:
   - "Full (MADR)" — Comprehensive record with pros/cons, consequences, confirmation steps, and implementation notes
   - "Minimal (MADR)" — Concise record with context, outcome, risk matrix, and sources only
   - "No output" — End here, no file written
3. If "No output" selected, present the score table and stop. Otherwise:
   - Read the corresponding format file:
     - Full: `knowledge/formats/decision-format-full.md`
     - Minimal: `knowledge/formats/decision-format-minimal.md`
   - Write the decision record to `output/decisions/YYYY-MM-DD-<slug>.md`

## Hard Constraints

- Risk matrix is PRIMARY input for recommendation
- Highest decision score MUST be recommended — never override with narrative
- Every factual claim must cite `[Source: <url>]`
- No emojis
- Make exactly ONE MCP call (in Step 2)
- If MCP is unavailable, use web search to ground responses in official documentation. If both MCP and web search are unavailable, proceed using training knowledge and state clearly that responses are ungrounded.
- Follow the format files EXACTLY — they define the output structure
