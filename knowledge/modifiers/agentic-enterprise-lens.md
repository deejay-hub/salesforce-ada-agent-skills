# Agentic Enterprise Lens

This modifier applies the Salesforce Well-Architected **Agentic Enterprise** lens to architecture decisions. In the framework, the agentic lens is an overlay on the five core pillars, not a separate track. It adds the concerns that appear when solutions become agentic: non-deterministic reasoning, autonomous action within permission boundaries, Human-in-the-Loop (HITL) oversight, multi-agent orchestration, and consumption-based cost.

> Agents combine "probabilistic reasoning (LLM inference) with deterministic execution (workflows and orchestration)." Moving from an assistant that suggests to an agent that acts is "an architectural shift, not a configuration change." [Source: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise.html]

## Activation

- **On request:** "agentic lens", "agentic enterprise lens", "agentforce lens", "agent lens"
- **Auto-activated** when the decision involves any of the following:
  - Agentforce, or autonomous or semi-autonomous agents
  - LLM-driven actions, meaning reasoning that changes state
  - MCP or Agent2Agent (A2A) agent integrations
  - Choosing between agentic and traditional automation

When this lens is auto-activated, state it on the first line after the heading of the Step 2 output: "Agentic Enterprise lens active (auto-activated: [reason])."

This lens is composable with every other modifier. When a pillar lens is also active, that pillar lens sets the weights and this lens adds the agentic analysis on top.

## Pillar Weight Adjustment

**None.** This is an overlay lens. Base weights stay unchanged:

| Pillar | Weight |
|--------|--------|
| Trust | 25% |
| Reliability | 25% |
| Operational Excellence | 20% |
| Resource and Cost Optimization | 20% |
| Fairness | 10% |

The lens works by changing what gets assessed. It adds discovery questions, agentic risk factors, and anti-patterns that are automatically rated High risk. It does not change how pillars are weighted.

## Reference Material

Search and cite these during Steps 2-4:

| Resource | URL |
|----------|-----|
| The Agentic Enterprise (overview) | https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise.html |
| Agentic Trust | https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-trust.html |
| Agentic Reliability | https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-reliability.html |
| Agentic Operational Excellence | https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-operational-excellence.html |
| Agentic Resource and Cost Optimization | https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-resource-and-cost-optimization.html |
| Fairness for Agentic Systems | https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-fairness.html |
| Architect's Guide To Determining Agentic and Traditional Workflow Automation | https://architect.salesforce.com/docs/architect/decision-guides/guide/determining-agentic-vs-traditional-workflow-automation |

If the decision is "agent vs Flow/Apex", read the decision guide first. It provides an **orchestration density framework** that weighs execution path, goal complexity, and modality mix. Use it to frame the options in Step 2.

## Analysis Requirements

### Mandatory Discovery Questions

Add these to Step 1 Discovery, or to the Step 4 differentiating questions if discovery is already used up. Skip any question the user has already answered.

1. **Assist or act?** Does the agent only suggest, recommend, or draft, so a human approves every state change? Or does it decide and execute within permission boundaries?
2. **HITL scope.** Which actions need human approval? Check against the framework categories:
   - Irreversible actions
   - Actions above a financial threshold
   - External communication
   - Regulated data
   - Areas with a history of errors
3. **Identity.** Who is the running user? Consider:
   - Employee agent vs customer agent
   - A dedicated least-privilege Agent User or integration user
   - Whether user identity is propagated downstream
4. **Pricing model.** Is the org on Flex Credits (per action), Per-Conversation, or Per-User (an unmetered add-on for employees)? The two consumption models cannot be mixed in one org.
5. **Deterministic fallback.** For each agent, what Flow, Apex, or rule-engine path handles known patterns when the LLM tier fails? What is the human handoff route?
6. **Reuse of business logic.** Does the agent call existing workflows (Flow, Apex, APIs)? Or does it duplicate business logic in prompts or generated code?
7. **Semantic layer.** For analytical agents, is there a governed semantic layer (for example Tableau Semantics)? Or would the agent query raw tables?
8. **Grounding data.** Is the grounding data accurate, permissioned, and traceable? Name the sources: Data 360, Knowledge, external retrieval.

### Agent Development Lifecycle (ADLC)

Assess each option against the ADLC phases:

1. **Ideation and design**
2. **Development**
3. **Testing and validation**
   - Use Agentforce Testing Center for batch test sets. Cover happy paths, edge cases, adversarial inputs, and historical problem scenarios.
   - Use native evaluators, LLM-as-judge, or custom evaluators.
   - Test functional behavior, response quality, safety, performance, and security.
   - Assert output characteristics, not exact text.
4. **Deployment and release**
   - Roll out gradually: feature flags, canary, A/B or champion-challenger.
   - Version prompts and configurations.
   - Use tested rollback procedures.
5. **Monitoring and tuning**
   - Use Agentforce Session Tracing (off by default; stores trace data in Data 360) to capture step-by-step reasoning.
   - Detect performance, distribution, and behavioral drift.

[Source: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-operational-excellence.html]

### Multi-Tier Fallback Chain

Every option with an agent must describe its degradation path. The framework's reference chain:

1. **Primary agent with full context.** Best quality, but the highest failure risk.
2. **Secondary agent with reduced context.** For example, the last 30 days of data or the top 5 vector results.
3. **Deterministic rule engine.** Flow or Apex logic for known patterns.
4. **Human handoff via Omni-Channel.** Use skill-based routing and pass the full context, the strategies already attempted, and the failure reasons.

Orchestrate the chain in Flow or Apex triggered by Platform Events, and log which tier succeeded. Graceful human handoffs are not failures. [Source: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-reliability.html]

### Durable Checkpoints and Retries

- Use **high-volume Platform Events** for checkpoints and retry queues. They have a 72-hour replay window; standard-volume events keep only 24 hours.
- Retry with exponential backoff, using the framework's example delays of 30s, 2m, 8m and 30m.
- After about 5 retries, send the request to a dead-letter queue and alert.
- Retries need idempotent subscribers and correlation IDs.

### Einstein Trust Layer Boundaries

- The Trust Layer provides PII masking, toxicity detection, prompt defense filtering, zero data retention, and audit events.
- It works only on **data in motion during inference**. It does not govern:
  - Data at rest
  - Access to grounding sources
  - What happens to outputs afterward
- Zero data retention is a contractual commitment, not a control you can verify yourself.
- Treat the Trust Layer as one layer of defense in depth, never as the whole security design.

[Source: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-trust.html]

### Confidence Thresholds (Framework Guidance to Calibrate)

The Fairness for Agentic Systems page suggests these starting thresholds. They are framework guidance, not platform defaults. Calibrate them against production outcomes.

| Confidence | Behavior |
|------------|----------|
| High (>90%) | Agent proceeds autonomously |
| Moderate (70-90%) | Human reviews before action |
| Low (<70%) | Agent defers to a human |

[Source: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-fairness.html]

## Agentic Risk Factors by Pillar

Use these risk factors alongside the base risk factors in `knowledge/scoring/decision-score-methodology.md` when building the Step 4 risk matrix. Fixed confidence values still apply: Low risk = 85, Med risk = 55, High risk = 25.

### Trust (25%)

| Risk Factor | Low Risk -> High Confidence (85) | Med Risk -> Moderate Confidence (55) | High Risk -> Low Confidence (25) |
|-------------|----------------------------------|--------------------------------------|----------------------------------|
| **Running user / agent identity** | Dedicated least-privilege Agent User or integration user per agent; JWT-based agent auth; user identity propagated downstream | Shared integration user across agents; permissions reviewed but not trimmed via Event Monitoring | System administrator as running user; static API keys or shared secrets |
| **Action governance** | Subagent and action assignment treated as a permission boundary; Apex actions use `with sharing` or `inherited sharing`; tool parameters constrained | Most actions scoped; some system-mode actions documented | Undocumented system-mode actions; orchestrator gains privilege through more privileged specialists |
| **Prompt injection defense** | Instructions and data separated; input validation contracts at every boundary; high-risk free text preprocessed | Partial separation; external content not consistently treated as untrusted | System instructions concatenated with record content; raw instruction strings passed between agents |
| **Audit and accountability chain** | Agent instance, bot definition, running user, and human owner documented; Trust Layer and Event Monitoring routed to a SIEM | Audit events captured but retention not validated | Agent activity reconstructed from record side effects only |

### Reliability (25%)

| Risk Factor | Low Risk -> High Confidence (85) | Med Risk -> Moderate Confidence (55) | High Risk -> Low Confidence (25) |
|-------------|----------------------------------|--------------------------------------|----------------------------------|
| **Fallback chain** | Full multi-tier chain (primary, reduced context, deterministic, human) with tier logging | Agent with direct human handoff only | No fallback; LLM failure is a user-facing failure |
| **Circuit breakers** | Breaker with durable tripped state (for example a Custom Object), thresholds in Custom Metadata Types | Breaker state in Platform Cache with a durable backup planned | No breaker, or state only in Platform Cache (evictable) |
| **Checkpoint and retry** | High-volume Platform Events, capped exponential backoff, dead-letter queue, idempotent subscribers | Capped retries without durable checkpoints | Unbounded retries, standard-volume events for retry queues, or retrying logical failures |
| **Agent SLOs** | SLOs defined before build (conversation success, grounding accuracy, MTBF, MTTR) | SLOs defined but not instrumented | No SLOs; reliability discovered in production |
| **Bounded context** | Hard caps on retrieval, indexed filters, sampling on skewed parents | Some caps; skew not assessed | Unbounded "all history" retrieval on skewed data |

### Operational Excellence (20%)

| Risk Factor | Low Risk -> High Confidence (85) | Med Risk -> Moderate Confidence (55) | High Risk -> Low Confidence (25) |
|-------------|----------------------------------|--------------------------------------|----------------------------------|
| **Testing** | Agentforce Testing Center test sets (happy path, edge, adversarial, historical), with behavioral suites rerun on every release and model change | Manual Prompt Builder previews plus limited batch tests | No systematic testing; exact-text assertions on non-deterministic output |
| **Observability** | Agentforce Session Tracing enabled; p50/p95/p99 latency, token, and error metrics; drift detection | Trust Layer audit only, no step-level reasoning traces | Custom transactional objects as the main logging layer, or no tracing |
| **Release management** | Versioned prompts and configurations, gradual rollout, tested rollback | Versioned but released all at once | Unversioned changes on "Salesforce Default" where stability is needed; untested rollback |
| **Lifecycle governance** | Agents treated as managed assets; CoE review; pre-deployment checklist | Ad-hoc review per agent | Shadow AI: unclear permissions, no audit trail, unknown cost |

### Resource and Cost Optimization (20%)

| Risk Factor | Low Risk -> High Confidence (85) | Med Risk -> Moderate Confidence (55) | High Risk -> Low Confidence (25) |
|-------------|----------------------------------|--------------------------------------|----------------------------------|
| **Pricing model fit** | Design optimized for the chosen model: consolidated or cached actions (Flex Credits), first-conversation resolution (Per-Conversation), adoption (Per-User) | Pricing model known but not reflected in design | Pricing model unknown, or the design multiplies metered actions |
| **Action efficiency** | Bulkified actions; SOQL budgeted across 10-15-action chains; heavy work moved to async | Some chained actions near limits | Action chains that can hit the 100-query synchronous limit; unbounded reasoning loops |
| **Grounding efficiency** | Top-k limits, metadata filters, semantic chunking; context summarized after several turns | Retrieval capped but over-fetching (for example 50 retrieved, 5 used) | Raw conversation history forwarded between agents; unbounded retrieval |
| **Cost governance** | Digital Wallet as primary monitor; budget alerts at 70%/85% per agent; cost per outcome; 3-5 year TCO | Spend monitored, not attributed to outcomes | No consumption monitoring; custom dashboards duplicating Digital Wallet |
| **Composition** | Specialist agents with narrow orchestrators and capped chain depth; deterministic paths moved to Flow | Some decomposition | Monolithic universal agent |

### Fairness (10%)

| Risk Factor | Low Risk -> High Confidence (85) | Med Risk -> Moderate Confidence (55) | High Risk -> Low Confidence (25) |
|-------------|----------------------------------|--------------------------------------|----------------------------------|
| **Human oversight** | HITL gates as Flow checkpoints outside the agent's reasoning path; oversight proportional to impact | HITL defined but enforced only for some categories | Approvals relying on agent instructions alone; fully autonomous high-stakes decisions |
| **Explainability and contestability** | Layered explanations; "Why did you recommend this?" on demand; appeal authority; historical reconstruction | Explanations available to admins only | No explanation or appeal path |
| **Bias monitoring** | Continuous bias detection; fairness as a release gate; RAG and training data audited | Pre-launch fairness check only | No bias review; consensus between agents treated as validation |
| **Disclosure and consent** | Prominent, continuous bot disclosure; persistent "Connect with human agent"; granular, dynamic consent | Disclosure shown once; blanket AI consent | No disclosure; escalation routed to overloaded queues |

## Automatically High-Risk Anti-Patterns

If any option contains one of these, rate the related criterion **High** and cite the agentic pillar page:

1. **System administrator as the running user.** It avoids permission troubleshooting at the cost of least privilege. (Trust)
2. **Instructions concatenated with record data.** String-building prompts from Case Description, Email Body, chat transcripts, or Knowledge content. (Trust)
3. **Approvals relying on agent instructions alone.** Mandatory approvals must be deterministic Flow checkpoints. (Trust / Fairness)
4. **Static API keys, shared secrets, or one shared identity across agents.** (Trust)
5. **No fallback chain.** LLM unavailability or validation failure surfaces directly to users. (Reliability)
6. **Circuit-breaker state only in Platform Cache.** Cache entries can be evicted. (Reliability)
7. **Unbounded retrieval or retries.** "All history" queries, retries with no maximum, tight-loop or non-idempotent retries. (Reliability / Resource and Cost Optimization)
8. **Monolithic universal agent,** or an orchestrator that reasons about the underlying task. (Resource and Cost Optimization)
9. **Agent reimplementing business logic** in prompts or generated code instead of calling existing workflows. (Operational Excellence)
10. **Agents querying raw tables without governed semantics.** (Trust / Operational Excellence)
11. **Shadow AI.** Agents with unclear permissions, no audit trail, and unknown cost. (Operational Excellence)
12. **Governance retrofitted after deployment.** (Operational Excellence)
13. **Fully autonomous high-stakes decisions.** No human review before execution in employment, credit, healthcare, legal rights, or housing. (Fairness)
14. **Trusting the Einstein Trust Layer as the whole security design,** for example sending full records to the LLM and assuming PII masking catches everything. (Trust)

## Criteria Guidance

When generating Step 3 criteria for agentic decisions, include at least one agentic criterion per relevant pillar:

| Pillar | Agentic criteria |
|--------|------------------|
| Trust | Agent identity and running user, action governance, prompt injection surface, Trust Layer coverage, accountability chain, EU AI Act / GDPR Article 22 risk level |
| Reliability | Fallback chain depth, circuit-breaker durability, checkpoint/retry durability, agent SLOs, grounding accuracy |
| Operational Excellence | Agent testability (Testing Center), reasoning observability (Session Tracing), prompt/config versioning, drift detection, lifecycle governance |
| Resource and Cost Optimization | Pricing model fit, actions per business transaction, grounding/context efficiency, cost per outcome, build vs buy (AgentExchange) |
| Fairness | HITL proportionality, explainability, contestability, bias monitoring, disclosure and consent |

Use "Well-Architected: Agentic [Pillar]" as the criterion Source when it comes from an agentic pillar page.

## Output Enhancements

### Add "Agentic Considerations" Section to Every Output

Every response must include a dedicated "Agentic Considerations" section covering:

- **Assist vs act classification**, and what that implies for oversight
- **Identity model:** running user, agent identity, and the accountability chain
- **HITL design:** which action categories are gated and where the gate lives (Flow checkpoint, not agent instructions)
- **Fallback chain:** which tiers exist and the human handoff route
- **Cost model:** pricing model, expected actions per transaction, and monitoring via Digital Wallet
- **ADLC readiness:** the testing (Testing Center) and observability (Session Tracing) approach
- **Flagged agentic anti-patterns**, if any

In decision records (full format), include this section under "More Information".

### Trade-off Analysis: Agentic Bias

- Favor "deterministic where possible, agentic where needed." Paths that are predictable belong in Flow or Apex.
- Favor proportional oversight over either full autonomy or blanket approval.
- Call out non-determinism explicitly: "This option relies on LLM reasoning for [step]. Failure mode: [mode]. Fallback: [tier]."

## Example Output (Agentic Lens Applied)

### User Question
> Should we use an Agentforce agent or a record-triggered Flow to process inbound refund requests?

### Agentic Lens Response (excerpt)
> Agentic Enterprise lens active (auto-activated: decision compares agentic vs traditional automation).
>
> **Agentic Considerations:**
> - **Assist vs act:** Issuing refunds changes state and is financially consequential, so the agent acts. Refunds above the financial threshold require a HITL gate implemented as a Flow checkpoint outside the agent's reasoning path.
> - **Identity:** Customer-facing agent runs as a dedicated least-privilege Agent User, not a guest user.
> - **Fallback chain:** Primary agent, then reduced-context agent, then the deterministic refund Flow for standard cases, then Omni-Channel handoff with full context.
> - **Cost model:** Under Flex Credits, a retrieve, reason, and update flow consumes three actions. Consolidate actions and route deterministic cases to Flow.
> - **Anti-pattern flagged:** Option B lets agent instructions enforce the approval rule. Rated High risk on Human Oversight.

## Checklist for Agentic Lens Output

When the agentic lens is active, verify every response includes:

- [ ] Activation statement (requested or auto-activated with reason)
- [ ] Assist vs act classification
- [ ] Running user and agent identity per agent; accountability chain documented
- [ ] HITL categories (irreversible, financial threshold, external communication, regulated data, past errors) mapped to gates
- [ ] Deterministic fallback and human handoff per agent
- [ ] Circuit-breaker and retry design with durable state and caps
- [ ] Pricing model identified and reflected in design
- [ ] ADLC testing (Testing Center) and observability (Session Tracing) approach
- [ ] Trust Layer boundaries acknowledged (data in motion only)
- [ ] Confidence thresholds stated as framework guidance to calibrate
- [ ] Agentic anti-patterns flagged and rated High
- [ ] "Agentic Considerations" section
- [ ] Agentic pillar pages and decision guide cited
