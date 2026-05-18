# Decision Record Output Format — Minimal (MADR)

## Score Presentation

Before writing the file, present scores as a table:

```markdown
| Option | Decision Score | Recommendation |
|--------|-----------|----------------|
| [Option 1] | XX/100 | |
| [Option 2] | XX/100 | **Recommended** |
| [Option 3] | XX/100 | |
```

The highest decision score row gets **Recommended**.

## File Output Format

Write to `output/decisions/YYYY-MM-DD-<slug>.md` with this exact structure:

```markdown
---
status: proposed
date: [YYYY-MM-DD]
---

# [Short Decision Title]

## Context and Problem Statement

[1-2 paragraphs: what is the problem, what constraints apply]

## Considered Options

- [Option 1 Name]
- [Option 2 Name]
- [Option 3 Name]

## Decision Outcome

Chosen option: **[Option Name]** (decision score: XX/100), because [1-2 sentences referencing risk data and decision scores].

## Risk Matrix

[Include the full risk matrix table from Step 4]

| Option | Low | Medium | High | Risk Profile |
|--------|-----|--------|------|--------------|
| [Option 1] | X | Y | Z | [low-risk / moderate / high-risk] |
| [Option 2] | X | Y | Z | [low-risk / moderate / high-risk] |
| [Option 3] | X | Y | Z | [low-risk / moderate / high-risk] |

## Sources

- [All cited documentation URLs]
```

## Rules

- Minimal frontmatter: status and date only
- Recommended option MUST have highest decision score
- Include the risk matrix and summary table (the data that drives the decision)
- All sources must be listed
- No pros/cons breakdown, no consequences, no implementation notes
- This format is for quick decisions that need traceability without full ceremony
