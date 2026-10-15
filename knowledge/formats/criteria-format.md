# Assessment Criteria Output Format

Present criteria as a markdown table using pipe syntax.

## Required Format

```markdown
| # | Criterion | Description | Source | Pillar |
|---|-----------|-------------|--------|--------|
| 1 | [Short Name] | [What this evaluates — one sentence] | [Where this comes from] | [Pillar name] |
| 2 | [Short Name] | [What this evaluates — one sentence] | [Where this comes from] | [Pillar name] |
| 3 | [Short Name] | [What this evaluates — one sentence] | [Where this comes from] | [Pillar name] |
```

## Rules

- Output MUST be a markdown pipe-delimited table
- Do NOT use numbered lists or bullet points
- Generate 6-10 criteria
- Criterion name: 2-5 words maximum
- Description: ONE sentence maximum
- Source: where the criterion comes from (e.g., "Governor Limits", "Platform Constraint", "Well-Architected: Trust", "Decision Guide: Async Processing", "Business Requirement", "Team Constraint", "Licensing Model")
- Pillar: assign each criterion to exactly ONE of the five Salesforce Well-Architected pillars: Trust, Reliability, Operational Excellence, Resource and Cost Optimization, Fairness
- Include criteria from this list where relevant to the decision: scalability, governor limits, licensing cost, implementation complexity, maintainability, team skillset, time to deliver
- Each criterion must be grounded in a specific capability, limit, or trade-off discovered during Step 2 research — not generic filler

## Pillar Assignment Guidance

| Pillar | Typical criteria |
|--------|-----------------|
| Trust | Security controls, identity and access, OAuth flow and integration identity, privacy, data residency, compliance, encryption, agent identity and action governance |
| Reliability | SLOs, availability, scalability, governor limit headroom (70% ceiling), fault tolerance, retry and fallback, disaster recovery, agent fallback chains |
| Operational Excellence | Observability, DevOps and deployment, maintainability, incident management, team capability, agent testing and tracing |
| Resource and Cost Optimization | Performance, query selectivity, API and consumption usage, licensing cost, development effort, time to value, TCO, build vs. buy, Flex Credit consumption |
| Fairness | Accessibility (WCAG 2.2 AA), data representation, fair automation and routing, transparency, human oversight, consent, data access equity |

## Confirmation Prompt

After the table, use `AskUserQuestion`:

Question: "Are these assessment criteria and pillar assignments complete?"
Options:
- "Yes, proceed" — Move to risk assessment
- "Add a criterion" — Let me specify one to add
- "Remove one" — Let me tell you which to drop
- "Modify one" — Let me refine a description or change a pillar assignment
