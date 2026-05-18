# Risk Matrix Output Format

Present the risk assessment as a markdown table with a summary.

## Required Format — Risk Matrix

```markdown
| Criterion | [Option 1 Name] | [Option 2 Name] | [Option 3 Name] |
|-----------|-----------------|-----------------|-----------------|
| [criterion] | **Low** — [rationale] | **Med** — [rationale] | **High** — [rationale] |
| [criterion] | **Med** — [rationale] | **Low** — [rationale] | **Low** — [rationale] |
```

## Rules for Risk Matrix

- Output MUST be a markdown pipe-delimited table
- One row per criterion, one column per option
- Each cell format: **[Level]** — [1-2 sentence rationale]
- Rationale MUST reference specific evidence: platform capabilities, limits, constraints, documentation, or known trade-offs
- Levels: **Low**, **Med**, **High** (use these exact words)

## Required Format — Summary Table

Below the risk matrix, always show a summary count:

```markdown
| Option | Low | Medium | High | Risk Profile |
|--------|-----|--------|------|--------------|
| [Option 1] | X | Y | Z | [low-risk / moderate / high-risk] |
| [Option 2] | X | Y | Z | [low-risk / moderate / high-risk] |
| [Option 3] | X | Y | Z | [low-risk / moderate / high-risk] |
```

## Risk Profile Rules

- 0-1 High ratings = "low-risk"
- 2-3 High ratings = "moderate"
- 4+ High ratings = "high-risk"

## Confirmation Prompt

After both tables, use `AskUserQuestion`:

Question: "Are you happy with the risk assessment?"
Options:
- "Yes, calculate the decision score" — Proceed to scoring
- "I disagree with a rating" — Let me challenge a specific cell
- "Re-assess one criterion" — Ask me more context on a specific area
- "Add another criterion" — I think something is missing
