---
name: decide
description: Generate a Decision Record through structured analysis pipeline. Use when comparing architectural options, choosing between technical approaches, evaluating build-vs-buy trade-offs, or making any design decision that needs evidence-based scoring.
argument-hint: [decision topic]
disable-model-invocation: true
allowed-tools:
  - AskUserQuestion
  - mcp__salesforce-content__search_docs
  - mcp__salesforce-content__fetch_page
  - Read
  - Write
---

You are in DECIDE mode. Guide the user through a structured decision pipeline for "$ARGUMENTS".

## Pipeline

Execute these steps IN ORDER. Pause for user input between each step. Do not skip steps.

### Step 1: Discovery

Gather context using `AskUserQuestion`. Ask about:
- Business problem or technical challenge
- Key numbers (record volumes, user count, transaction frequency)
- Salesforce products, editions, and existing customizations
- Delivery timeline
- Budget, compliance, or technical constraints

Skip questions already answered in "$ARGUMENTS". Do NOT call MCP tools during discovery.

**Limit discovery to 2 rounds of questions (max 8 questions total).** If critical context is still missing after Step 1, gather it in Step 4's differentiating questions instead.

### Step 2: Solution Options

#### 2a. Research

Call `mcp__salesforce-content__search_docs` directly (limit: 10) for relevant approaches and documentation. Restrict `sources` to dev-guides and architecture content only:
   - **Always include:** `architect`
   - **Include when relevant:** `apex-dev-guide`, `agentforce-dev-guide`, `agentforce-vibes-dev-guide`, `actions-dev-guide`, `bulk-api-dev-guide`, `cdc-dev-guide`, `data-cloud-dev-guide`, `data-cloud-integration-dev-guide`, `data-cloud-mapping-dev-guide`, `data-cloud-query-dev-guide`, `mobile-sdk-dev-guide`, `mulesoft-docs`, `platform-events-dev-guide`, `rest-api-dev-guide`
   - **Never include:** `admin-blog`, `architect-blog`, `developer-blog`, `admin-youtube`, `architect-youtube`, `developer-youtube`

Multiple search calls are allowed in this step. Use as many as needed to adequately research the option space.

#### 2b. Propose Options

Propose 3-4 options based on research results.

Read `knowledge/formats/options-format.md` for the exact output format. Follow it precisely.

#### 2c. Validate Options

Before presenting options to the user, validate EVERY proposed option against official documentation:

1. For each option, call `mcp__salesforce-content__search_docs` with a targeted query to confirm the option is a documented, supported capability (e.g., product compatibility, supported deployment targets, feature availability).
2. Use `mcp__salesforce-content__fetch_page` to expand on search results when a snippet is insufficient to confirm support.
3. For each option, record a validation status:
   - **Confirmed** — Documentation explicitly supports this option's technical approach
   - **Unconfirmed** — Documentation does not explicitly address this combination; requires vendor confirmation before committing
4. **Drop any option that documentation explicitly contradicts** (e.g., a product combination documented as unsupported). Explain why it was removed.
5. Present the options table with a "Docs" column showing the validation status and a citation.

If the user adds or modifies an option during confirmation, run the same validation process on the new/modified option before accepting it.

Use `AskUserQuestion` to confirm options before proceeding.

### Step 3: Assessment Criteria

Generate 6-10 assessment criteria from MCP results. Each criterion MUST:
- Be grounded in a specific capability, limit, or trade-off discovered during Step 2 research
- Be tagged with its **target pillar** (Trust, Reliability, Operational Excellence, Resource Optimization, Cost Optimization, or Fairness) — this pillar mapping will be used in Step 5 scoring

Read `knowledge/formats/criteria-format.md` for the exact output format.

Use `AskUserQuestion` to confirm criteria (and their pillar assignments) before proceeding.

### Step 4: Risk Assessment

1. Generate 3-5 differentiating questions using `AskUserQuestion` with selectable options where possible. Use these to gather context that will inform risk ratings.

#### 4a. Verify Claims

Before building the risk matrix, verify the factual claims that will inform risk ratings:

1. Identify the **distinct platform facts** (limits, capabilities, behaviors, allocations) that determine risk levels across the matrix. Many cells will share the same underlying fact — group verifications by fact, not by cell.
2. Call `mcp__salesforce-content__search_docs` with targeted queries to confirm each distinct fact. Use `mcp__salesforce-content__fetch_page` when snippets are insufficient.
3. Record findings as evidence. Each fact must have a source URL.
4. If documentation contradicts an assumed fact, adjust the risk rating accordingly.
5. If documentation does not address a claim, mark the rationale as "Unverified — based on training knowledge" and rate conservatively (assume higher risk).

Multiple search calls are expected in this step. Batch related queries where possible.

#### 4b. Build Risk Matrix

Rate each option against each criterion. Every cell's rationale MUST reference a specific documented fact with a `[Source: url]` citation from the verification step. Do not present any rating without a grounding citation.

Read `knowledge/formats/risk-matrix-format.md` for the exact output format.

Use `AskUserQuestion` to confirm the risk matrix before proceeding.

### Step 5: Decision Score

1. Read `knowledge/scoring/decision-score-methodology.md` now. Use the pillar assignments from Step 3 to build the scoring table. If multiple criteria map to one pillar, the pillar confidence is determined by the **highest-risk rating** among them (i.e., the most concerning criterion governs the pillar). Convert risk ratings to confidence levels using these fixed values:
   - **Low** risk → **High confidence** (85)
   - **Med** risk → **Moderate confidence** (55)
   - **High** risk → **Low confidence** (25)

   Present the pillar mapping as a table with these columns:
   - **Pillar** — name and weight
   - **Criteria Mapped** — which risk matrix criteria feed this pillar (from Step 3 tagging)
   - **Scoring Rule** — "Governed by highest-risk criterion" (always this rule)

   Then present the per-option scoring table showing the specific criterion that governed each pillar's confidence (e.g., "Data Sharing Fidelity = High risk → Low confidence (25)").

2. **Recommendation Rationale** — After the score table, write 2-3 sentences explaining: which pillar(s) most differentiated the winner, what documented fact(s) drove those pillar scores, and why the runner-up fell short. This rationale must be consistent with the numbers.

3. Use `AskUserQuestion` to confirm the score and recommendation before asking about output format.

4. Use `AskUserQuestion` to ask the user which output format they prefer:
   - Question: "Which decision record format would you like?"
   - Options:
     - "Full (MADR)" — Comprehensive record with pros/cons, consequences, confirmation steps, and implementation notes
     - "Minimal (MADR)" — Concise record with context, outcome, risk matrix, and sources only
     - "No output" — End here, no file written

5. If "No output" selected, stop. Otherwise:
   - Read the corresponding format file:
     - Full → `knowledge/formats/decision-format-full.md`
     - Minimal → `knowledge/formats/decision-format-minimal.md`
   - Write the decision record to `output/decisions/YYYY-MM-DD-<slug>.md`

## Hard Constraints

- Risk matrix is PRIMARY input for recommendation
- Highest decision score MUST be recommended — never override with narrative
- Every factual claim must cite `[Source: <url>]`
- No emojis
- Use `search_docs` and `fetch_page` for all MCP calls in this mode. Do NOT use curl or spawn agents.
- NEVER present an option to the user without first validating it against official documentation. This applies to both initially proposed options AND user-suggested additions/modifications.
- If MCP is unavailable, use web search to ground responses in official documentation. If both MCP and web search are unavailable, proceed using training knowledge and state clearly that responses are ungrounded.
- Use `AskUserQuestion` at all confirmation points
- Follow the format files EXACTLY — they define the output structure
- Fixed confidence values: Low risk = 85, Med risk = 55, High risk = 25. Do not vary within bands.
