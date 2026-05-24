# Decision Record Output Format — Full (MADR)

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
decision-makers: [list names or roles]
consulted: [list names, roles, or teams]
informed: [list names, roles, or teams]
---

# [Short Decision Title]

## Context and Problem Statement

[2-3 paragraphs: business problem, technical challenge, what prompted this decision, relevant constraints]

## Decision Drivers

- [Driver 1 — e.g., scalability requirement]
- [Driver 2 — e.g., compliance constraint]
- [Driver 3 — e.g., timeline pressure]
- [Driver 4 — e.g., team skillset]

## Considered Options

- [Option 1 Name]
- [Option 2 Name]
- [Option 3 Name]

## Decision Outcome

Chosen option: **[Option Name]** (decision score: XX/100), because [1-2 sentences referencing risk data and decision scores].

### Consequences

#### Good

- [Positive consequence 1]
- [Positive consequence 2]

#### Bad

- [Accepted trade-off 1]
- [Accepted trade-off 2]

#### Neutral

- [Side effect or observation]

### Confirmation

To confirm the decision is implemented correctly:
- [ ] [Verification step 1]
- [ ] [Verification step 2]
- [ ] [Verification step 3]

## Pros and Cons of the Options

### [Option 1 Name]

[1-2 sentence description]

| Pros | Cons |
|------|------|
| [pro 1] | [con 1] |
| [pro 2] | [con 2] |
| [pro 3] | [con 3] |

**Decision Score:** XX/100 | **Effort:** [low/medium/high] | **Risk:** [low/medium/high]

### [Option 2 Name]

[1-2 sentence description]

| Pros | Cons |
|------|------|
| [pro 1] | [con 1] |
| [pro 2] | [con 2] |
| [pro 3] | [con 3] |

**Decision Score:** XX/100 | **Effort:** [low/medium/high] | **Risk:** [low/medium/high]

### [Option 3 Name]

[1-2 sentence description]

| Pros | Cons |
|------|------|
| [pro 1] | [con 1] |
| [pro 2] | [con 2] |
| [pro 3] | [con 3] |

**Decision Score:** XX/100 | **Effort:** [low/medium/high] | **Risk:** [low/medium/high]

## Risk Matrix

[Include the full risk matrix table from Step 4]

[Include the summary count table from Step 4]

## More Information

### Implementation Notes

- [ ] [Action item 1]
- [ ] [Action item 2]
- [ ] [Action item 3]
- [ ] [Action item 4]

### Sources

- [All cited documentation URLs from the analysis]
```

## Rules

- Frontmatter uses MADR fields: status, date, decision-makers, consulted, informed
- Recommended option MUST have highest decision score
- Include the full risk matrix AND summary table in the output file
- All sources cited in the document must be listed in Sources section
- Pros/Cons as a table, not bullet lists
- "Confirmation" section contains concrete verification steps
- Leave decision-makers/consulted/informed as placeholders if unknown
