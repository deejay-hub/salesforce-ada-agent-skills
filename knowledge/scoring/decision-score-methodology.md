# Decision Score Methodology

> **This methodology is the ground truth for scoring architectural decisions in the decision pipeline.** All recommendations must follow this scoring rubric exactly.

## Framework Alignment

The decision score uses the five pillars of the [Salesforce Well-Architected Framework](https://architect.salesforce.com/docs/architect/well-architected/guide/framework.html). These pillars replace the legacy Trusted / Easy / Adaptable model. The framework is an architectural decision framework, and its goal is "the right quality profile," not maximizing every pillar.

| Pillar | Framework definition | Pillar guide | Agentic variant |
|--------|---------------------|--------------|-----------------|
| **Trust** | Protect people, data, and systems through secure, compliant design | [trust](https://architect.salesforce.com/docs/architect/well-architected/guide/trust.html) | [agentic trust](https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-trust.html) |
| **Reliability** | Build systems that continue working and recover gracefully when something fails | [reliability](https://architect.salesforce.com/docs/architect/well-architected/guide/reliability.html) | [agentic reliability](https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-reliability.html) |
| **Operational Excellence** | Observe, automate, and continuously improve how systems run | [operational excellence](https://architect.salesforce.com/docs/architect/well-architected/guide/operational-excellence.html) | [agentic operational excellence](https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-operational-excellence.html) |
| **Resource and Cost Optimization** | Invest intentionally to maximize long-term business value | [resource and cost optimization](https://architect.salesforce.com/docs/architect/well-architected/guide/resource-and-cost-optimization.html) | [agentic resource and cost optimization](https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-resource-and-cost-optimization.html) |
| **Fairness** | Ensure systems are transparent, accessible, and equitable | [fairness](https://architect.salesforce.com/docs/architect/well-architected/guide/fairness.html) | [agentic fairness](https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-fairness.html) |

The **Agentic Enterprise lens** is an overlay on these pillars, not a separate track. Whenever a decision involves agents, apply `knowledge/modifiers/agentic-enterprise-lens.md`. It adds agentic risk factors to each pillar without changing the weights.

## Pillars and Weights

The decision score is a weighted sum of the five pillars. Each pillar receives a **confidence level** derived from the risk assessment: lower risk means higher confidence in the option's fitness for that pillar.

The framework itself assigns no numeric weights. The weights below are this repo's convention. They follow the framework's guidance that Trust and Reliability are "non-negotiable foundations" for new designs.

| Pillar | Weight | Focus Areas | Risk-to-Confidence Mapping |
|--------|--------|-------------|---------------------------|
| **Trust** | 25% | Security, identity and access, privacy, compliance, secure development, incident response | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Reliability** | 25% | SLOs, high availability, scalability and capacity, fault tolerance, disaster recovery, health monitoring | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Operational Excellence** | 20% | Observability, DevOps, automation, incident management, continuous improvement, team capability | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Resource and Cost Optimization** | 20% | Governor limit efficiency, query selectivity, caching, code organization, packaging, data lifecycle, TCO, licensing and consumption, build vs. buy | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Fairness** | 10% | Accessibility, data quality and representation, fair automation, transparency, human oversight, consent | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |

### Total Decision Score Formula

```
decisionScore = (trust × 0.25) + (reliability × 0.25) + (operational × 0.20) +
                (resourceAndCost × 0.20) + (fairness × 0.10)
```

Each pillar value is the confidence level (85, 55, or 25) derived from the risk assessment.

**Result range:** 0-100. Higher is better, meaning higher confidence across pillars.

**Decision bands:**
- **80-100:** Strong fit, with high confidence across most pillars
- **60-79:** Acceptable fit. Recommend with caveats or risk mitigation.
- **40-59:** Marginal fit. Recommend an alternative or hybrid approach.
- **0-39:** Poor fit. Do not recommend; propose a different architecture.

### Lens Weight Adjustments

A pillar lens raises its pillar to 35%. The remaining pillars are scaled proportionally to keep the total at 100%. Each lens file lists its exact weights.

When **multiple pillar lenses** are active:
1. Set each lensed pillar to 35.
2. Keep every other pillar at its base weight.
3. Divide every weight by the new total, so the weights sum to 100%.

For example, Trust and Reliability lenses together give 35 + 35 + 20 + 20 + 10 = 120. That normalizes to Trust 29.17%, Reliability 29.17%, Operational Excellence 16.67%, Resource and Cost Optimization 16.67%, Fairness 8.33%.

The Agentic Enterprise lens and the concise format modifier do not change weights.

---

## Pillar 1: Trust (25%)

**Definition:** How well does the solution apply security controls, govern identity and access, respect privacy, and satisfy compliance obligations under the Shared Responsibility Model?

### Scoring Criteria (0-100)

| Confidence Level | Identity and Access | Data Protection | Integration Security | Compliance |
|-------------|---------------|--------------|------------|------------|
| **High confidence (85)** | MFA and SSO, private OWDs, least-privilege permission sets, per-user identity propagation | Data classified (Public / Internal / Confidential / Restricted), Shield Platform Encryption for Restricted data | Named Credentials, External Client Apps, JWT Bearer or Web Server Flow with PKCE | Health Check 80%+, Event Monitoring, audit trail strategy |
| **Moderate confidence (55)** | MFA enforced, broad profiles, some shared integration users | Standard encryption at rest, classification incomplete | Legacy Connected Apps with scoped OAuth | Partial controls, Setup Audit Trail only |
| **Low confidence (25)** | No MFA, admin-level or shared service accounts | Restricted data unencrypted, no classification | Username-Password Flow, hardcoded secrets | No compliance controls |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Data exposure** | Apex runs `with sharing` and queries use `WITH USER_MODE` or `Security.stripInaccessible()` | Some system-mode access, documented and justified | Undeclared or `without sharing` classes, no CRUD/FLS enforcement |
| **Identity model** | Per-user authentication or OAuth 2.0 Token Exchange, MFA and SSO | Dedicated integration user per integration | Shared service account, Username-Password Flow |
| **Sharing model** | Private OWD with explicit sharing, restriction rules where needed | Public Read Only | Public Read/Write (the framework calls permissive OWDs "the most common trust debt in mature orgs") |
| **Secrets management** | Named Credentials and External Credentials | Secrets in protected Custom Metadata or Custom Settings | Secrets hardcoded in Apex or metadata |
| **Audit and detection** | Event Monitoring streamed to a SIEM, Transaction Security policies | Setup Audit Trail (180 days) reviewed periodically | No audit trail |

### Example Scoring

**High confidence (85):**
- MFA and SSO enforced for all users
- Apex uses `with sharing` and `WITH USER_MODE`
- Shield Platform Encryption on Restricted fields
- Named Credentials with an External Client App per integration
- Event Monitoring logs streamed to a SIEM

**Moderate confidence (55):**
- MFA enforced, but profiles are broader than needed
- Most Apex uses `with sharing`; a few documented system-mode paths
- Standard encryption at rest
- One integration user shared across two integrations

**Low confidence (25):**
- No MFA enforcement
- Apex uses `without sharing` with no justification
- Username-Password OAuth flow or hardcoded credentials
- Public Read/Write OWD on sensitive objects

> `WITH SECURITY_ENFORCED` was removed in API 67.0. Use `WITH USER_MODE` instead. [Source: https://architect.salesforce.com/docs/architect/well-architected/guide/trust.html]

---

## Pillar 2: Reliability (25%)

**Definition:** How well does the solution meet its SLOs, tolerate failure, scale with load, and recover from incidents?

### Scoring Criteria (0-100)

| Confidence Level | Targets | Fault Tolerance | Scaling | Recovery |
|-------------|----------------|---------|----------|--------------------------|
| **High confidence (85)** | SLOs defined per user journey, set looser than the platform SLA (leaving an error budget) | Graceful degradation tiers, circuit breakers, retry with backoff, fallbacks | Bulkified, async where needed, peak usage within the 70% governor-limit operational ceiling | RTO/RPO defined, tested backups, DR drills run |
| **Moderate confidence (55)** | Informal availability expectations | Basic error handling, manual retry | Peak usage occasionally exceeds the 70% ceiling | RTO/RPO defined, but recovery is untested |
| **Low confidence (25)** | No targets, or "five nines for everything" | No error handling, silent failures | SOQL/DML in loops, peak near limits | No recovery plan; backups are "an untested assumption" |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Governor limits** | Peak below the 70% operational ceiling | Peak 70-85% of a limit | Peak above 85%, or unbounded with volume |
| **Async patterns** | Right tool per need: Batch Apex (volume), Queueable (sequence), Platform Events (decoupling) | Future methods or scheduled jobs without monitoring | Synchronous only, no async fallback |
| **Error handling** | Fail fast, gracefully, informatively, and safely; dead-letter handling | Basic try/catch and logging | No error handling, silent failures |
| **Data volume** | Partitioned (date, record type, owner); load-tested in a Full Copy sandbox | Tested only at expected volume | Untested at production volume, or data skew present |
| **External dependencies** | Circuit breaker, timeouts, fallback (cache, default, queue for retry) | Retry with backoff and timeout handling | No timeout or retry; retries on 4xx errors other than 429 |

### Example Scoring

**High confidence (85):**
- All SOQL/DML operations are bulk-safe
- Queueable for sequenced async work (200 SOQL queries per async transaction)
- Platform Events for decoupled pub/sub
- Circuit breaker on external callouts
- Batch Apex for large data volumes (up to 50M records via QueryLocator)

**Moderate confidence (55):**
- Most SOQL/DML is bulk-safe, with some edge cases
- Future methods for async processing
- Basic retry logic for callouts
- RTO/RPO documented but never drilled

**Low confidence (25):**
- SOQL/DML in loops
- All processing synchronous
- No error handling for callouts
- Fails with more than 200 records

---

## Pillar 3: Operational Excellence (20%)

**Definition:** How well can the solution be observed, deployed, automated, supported during incidents, and continuously improved?

### Scoring Criteria (0-100)

| Confidence Level | Observability | DevOps | Automation | Incident Management |
|-------------|------------|---------------|------------|-----------------|
| **High confidence (85)** | Instrumented from v1; SLIs (availability, latency, success rate, throughput) with error budgets; actionable alerts | Source-driven (Salesforce DX), CI/CD with validation gates, test pyramid, rollback runbooks | Declarative-first, automated repetitive work, no manual steps | Severity tiers, on-call, blameless postmortems |
| **Moderate confidence (55)** | Basic logging, reactive monitoring | Manual deploy with validation, limited rollback | Partially automated | Ad-hoc escalation, informal reviews |
| **Low confidence (25)** | Observability retrofitted after problems | Ad-hoc changes in production, no tests | Fully manual | Tribal knowledge, no postmortems |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Deployment complexity** | Metadata in source control, packaged, feature flags for progressive release | Mix of clicks and code, some manual steps | Manual, org-specific changes; one big trigger |
| **Monitoring visibility** | Event Monitoring, Scale Center, Proactive Monitoring, integration health monitored in both directions | Setup Audit Trail and periodic reviews | Reactive only |
| **Change velocity** | DORA metrics tracked (deployment frequency, lead time, change failure rate, failed deployment recovery time) | Weekly deploys with manual validation | Infrequent deploys, no tests |
| **Runbook completeness** | Operational Readiness Review passed, deployment and rollback runbooks | Documented procedures | Tribal knowledge |
| **Team capability** | Team has the skills to build and maintain it; Center of Excellence in place | Team can maintain it with some upskilling | Team lacks the skills and depends on a vendor or contractor |

### Example Scoring

**High confidence (85):**
- CI/CD pipeline with scratch org validation
- Event Monitoring logs streamed to a SIEM
- Automated deployment via the Salesforce CLI
- Test pyramid with unit, integration, and end-to-end tests
- Runbooks and blameless postmortems for incidents

**Moderate confidence (55):**
- Manual deployment with validation
- Setup Audit Trail reviewed monthly
- Some automated tests
- Basic documentation

**Low confidence (25):**
- Ad-hoc changes in production
- No monitoring
- Legacy Process Builder automation (past end of support)
- No tests or documentation

---

## Pillar 4: Resource and Cost Optimization (20%)

**Definition:** How much value does the solution deliver per unit of cost? The framework calls resource optimization "the mechanism" and cost optimization "the outcome."

### Scoring Criteria (0-100)

| Confidence Level | Platform Efficiency | Total Cost of Ownership | Licensing and Consumption | Code and Data Organization |
|-------------|---------|-------------------|---------|--------------|
| **High confidence (85)** | Selective queries, bulkified, caching where justified, under 50% heap and CPU | 3-5 year TCO modeled (direct and indirect costs), spend aligned to business value | Right-sized licenses, consumption monitored with 70%/85% budget alerts | Trigger handler, service, and selector layers; packaged; archival strategy |
| **Moderate confidence (55)** | Some non-selective queries, 50-80% heap and CPU | Implementation cost estimated, run cost not modeled | 1-2 add-ons, consumption not monitored | Partial layering, no archival plan |
| **Low confidence (25)** | Full table scans, above 80% heap and CPU, polling | No TCO; hidden technical-debt interest | Over-licensing, idle sandboxes, unbounded consumption | Monolithic code, data skew (10K+ child records per parent) |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Query selectivity** | Indexed filters, selective queries | Some non-selective queries on moderate volumes | Full table scans on large objects |
| **Heap and CPU** | Under 50% of sync/async limits | 50-80% of limits | Above 80%, or timeouts |
| **Integration efficiency** | Events or CDC, Composite API, Bulk API 2.0 | REST calls with batching | Chatty integrations or scheduled polling |
| **License and consumption cost** | No add-ons, or consumption with budget alerts | 1-2 add-ons (for example Shield or Experience Cloud) | 3+ add-ons, or unmetered consumption |
| **Development effort and time to value** | Declarative, MVP in under 2 weeks | Mix of clicks and code, MVP in 2-4 weeks | Heavy custom code, MVP in more than 4 weeks |
| **Build vs. buy** | Assessed on differentiation, time to value, organizational capability, and exit cost | Assessed on cost only | Not assessed |

### Example Scoring

**High confidence (85):**
- SOQL queries use indexed fields
- Platform Cache for frequently read configuration
- No add-ons required
- Declarative solution with an MVP in under 2 weeks
- License utilization dashboard in place

**Moderate confidence (55):**
- Some non-selective queries
- Shield add-on required for encryption
- Mix of flows and Apex
- Weekly batch job with some manual oversight

**Low confidence (25):**
- Full table scans, no caching
- Multiple add-ons (for example MuleSoft, Data 360, and Shield) with no consumption monitoring
- Heavy Apex/LWC with an MVP in more than 6 weeks
- Scheduled polling that usually finds no work

---

## Pillar 5: Fairness (10%)

**Definition:** How well does the solution deliver inclusive design, fair automation and AI, and governance that keeps outcomes transparent, accessible, and equitable?

### Scoring Criteria (0-100)

| Confidence Level | Accessibility | Fair Automation | Transparency | User Agency |
|-------------|---------------|-----------------|----------------|-----------------|
| **High confidence (85)** | WCAG 2.2 AA (recommended for new work); tested with assistive technology; sa11y in CI | Assignment, routing, and territory rules audited for outcomes | Explanations layered by audience, audit trail of decisions | Granular consent per use case, human handoff available |
| **Moderate confidence (55)** | Standard SLDS components without explicit testing (2.1 AA is the minimum for custom components) | Rules exist but have not been reviewed for bias | Logic documented but not surfaced to users | Blanket consent |
| **Low confidence (25)** | Custom UI with no accessibility consideration | Opaque logic that may disadvantage groups | No explanation or audit trail | No opt-out or human alternative |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Accessibility** | Standard LWC (inherits SLDS accessibility), tested with assistive technology | Standard components, no explicit testing | Custom UI with no accessibility consideration |
| **Data representation** | Required fields justified; name and address fields support global formats | Some Western-centric assumptions | Latin-only validation, required fields that exclude groups |
| **Automation bias** | Assignment and routing rules reviewed and auditable | Rules not reviewed for bias | Opaque logic, or AI routing with no bias review |
| **Human oversight** | Review before high-stakes actions; contestable decisions | Overrides possible but not tracked | No human review of consequential decisions |
| **Data access equity** | OWD and FLS applied for non-discrimination; reporting available to the affected roles | Some roles lack visibility into data that affects them | Data access concentrated and decisions opaque |

### Example Scoring

**High confidence (85):**
- Standard LWC components with assistive technology testing
- All user personas considered in the flow design
- Assignment rules use transparent, auditable criteria
- Multi-language support via custom labels

**Moderate confidence (55):**
- Standard components without explicit accessibility testing
- Primary personas covered; field reps underserved
- Territory rules not reviewed for geographic bias

**Low confidence (25):**
- Custom UI with no accessibility consideration
- AI-driven routing with no bias review
- No visibility into how decisions affect different groups

---

## Cross-Pillar Trade-offs

The framework names specific tensions between pillars. When the winning option trades one pillar against another, the Recommendation Rationale must name the tension it resolves:

| Tension | Framework guidance |
|---------|-------------------|
| Trust vs. Resource and Cost Optimization | Encryption and logging consume resources and cost. Size controls to data classification. |
| Reliability vs. Resource and Cost Optimization | Let SLOs set the level of reliability investment. |
| Operational Excellence vs. Speed | Find the "minimum viable observability" for the risk profile. |
| Fairness vs. Autonomy | Human-in-the-Loop patterns set the balance. |

[Source: https://architect.salesforce.com/docs/architect/well-architected/guide/framework.html]

---

## Consistency Rules

1. **Highest decision score MUST be the recommendation.** The option with the top decision score is always the recommended choice. If the narrative says "Option A is recommended" but Option B has a higher decision score, the data is wrong. Fix it.

2. **Risk-to-confidence mapping is mandatory.** Every pillar uses fixed confidence values: Low risk = 85, Med risk = 55, High risk = 25. No arbitrary values.

3. **Tie-breaking rules:**
   - Prefer fewer "high" risks (a 75 with 1 high risk beats a 75 with 2 high risks).
   - Weight criteria by priority: Trust > Reliability > Operational Excellence > Resource and Cost Optimization > Fairness.
   - Consider development effort as secondary: faster to deploy beats slower if scores are tied.

4. **Data overrides narrative.** If the scoring table shows Option A at 82 and Option B at 78, but the recommendation text says "Option B is recommended," the recommendation is WRONG. Fix it to match the data.

5. **No "best of both worlds" hedging.** Do not say "Option A is recommended, but Option B is also good." Pick the top decision score and commit.

---

## Example Scoring: Record-Triggered Flow vs. Apex Trigger

**Scenario:** Automate lead assignment based on territory rules. 500 leads/day, 10 territories, simple logic (zip code lookup).

### Option A: Record-Triggered Flow

| Pillar | Confidence | Governing Criterion | Reasoning |
|--------|------------|--------------------:|-----------|
| **Trust** (25%) | 85 | Data Exposure = Low risk | Runs in user context and respects sharing rules |
| **Reliability** (25%) | 55 | Error Handling = Med risk | Bulk-safe by default, but error handling is limited to fault paths |
| **Operational Excellence** (20%) | 85 | Deployment Complexity = Low risk | Declarative, no code, easy to change |
| **Resource and Cost Optimization** (20%) | 55 | Caching = Med risk | Zero development cost, but no caching for repeated territory lookups (governs as the highest-risk criterion) |
| **Fairness** (10%) | 85 | Automation Bias = Low risk | Transparent assignment rules, auditable by non-developers |
| **Decision Score** | **71.5** | | (85×0.25) + (55×0.25) + (85×0.20) + (55×0.20) + (85×0.10) |

### Option B: Apex Trigger

| Pillar | Confidence | Governing Criterion | Reasoning |
|--------|------------|--------------------:|-----------|
| **Trust** (25%) | 85 | Data Exposure = Low risk | Uses `with sharing` and `WITH USER_MODE` |
| **Reliability** (25%) | 85 | Governor Limits = Low risk | Bulk-safe, error handling, unit tests |
| **Operational Excellence** (20%) | 55 | Deployment Complexity = Med risk | Requires tests, a deploy pipeline, and code reviews |
| **Resource and Cost Optimization** (20%) | 55 | Development Effort = Med risk | Can cache lookups, but requires developer time (1 week) and ongoing maintenance |
| **Fairness** (10%) | 55 | Data Access Equity = Med risk | Logic is code-reviewable but less transparent to non-developers |
| **Decision Score** | **70.0** | | (85×0.25) + (85×0.25) + (55×0.20) + (55×0.20) + (55×0.10) |

**Recommendation:** **Option A: Record-Triggered Flow** (decision score 71.5 > 70.0)

**Rationale:** Flow wins on Operational Excellence (85 vs 55) because it is declarative and easy to change, and on Fairness (85 vs 55) because its assignment logic is auditable by non-developers. Apex wins on Reliability (85 vs 55) through comprehensive error handling, but that 25%-weighted advantage does not offset Flow's two wins. This resolves the Operational Excellence vs. Speed tension in favor of the simpler, declarative design at this volume.

---

**Last updated:** 2026-10-15 (aligned with the five-pillar Salesforce Well-Architected Framework and the Agentic Enterprise lens)
