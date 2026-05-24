# Solution Options Output Format

Present options as a markdown table using pipe syntax. ONE sentence max per description.

## Required Format

```markdown
| # | Option | Description | Approach | Docs |
|---|--------|-------------|----------|------|
| 1 | [Short Name] | [One sentence describing the approach] | [Category or pattern name] | Confirmed — [Source: url] |
| 2 | [Short Name] | [One sentence describing the approach] | [Category or pattern name] | Confirmed — [Source: url] |
| 3 | [Short Name] | [One sentence describing the approach] | [Category or pattern name] | Unconfirmed — [reason] |
```

## Rules

- Output MUST be a markdown pipe-delimited table
- Do NOT use numbered lists
- Do NOT use bullet points
- Do NOT use key-value pairs
- Description column: ONE sentence maximum
- Option name: 2-4 words maximum
- Approach column: name the architectural pattern or category (e.g., Fire-and-Forget, Declarative, Managed Package, Single-Org)
- Docs column: validation status — "Confirmed" with a source citation, or "Unconfirmed" with a brief reason why vendor confirmation is needed. Options contradicted by documentation must be dropped before presenting the table.

## Confirmation Prompt

After the table, use `AskUserQuestion`:

Question: "Are you happy with these options to compare?"
Options:
- "Yes, proceed with these" — Move to assessment criteria
- "I want to add an option" — Let me specify an additional option
- "I want to remove one" — Let me tell you which to drop
- "I want to modify one" — Let me refine a description
