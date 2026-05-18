# Decision Score Methodology

> **This methodology is the ground truth for scoring architectural decisions in the decision pipeline.** All recommendations must follow this scoring rubric exactly.

## Pillars and Weights

The decision score is a weighted sum of six pillars aligned with the Salesforce Well-Architected Framework. Each pillar receives a **confidence level** derived from risk assessment — lower risk means higher confidence in the option's fitness for that pillar.

| Pillar | Weight | Focus Areas | Risk-to-Confidence Mapping |
|--------|--------|-------------|---------------------------|
| **Trust** | 20% | Data protection, authentication, authorization, encryption, compliance, least privilege, threat surface | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Reliability** | 20% | Fault tolerance, scaling, recovery, governor limit resilience, async patterns, error handling | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Operational Excellence** | 20% | Operations, observability, automation, maintainability, deployment velocity, team capability | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Resource Optimization** | 15% | Compute efficiency, query selectivity, heap/CPU usage, caching, latency, right-sizing | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Cost Optimization** | 15% | Licensing costs, development effort, operational overhead, API usage, time to value | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |
| **Fairness** | 10% | Accessibility, equitable access, bias avoidance, inclusive design, multi-persona consideration | Low risk → High confidence (85), Med risk → Moderate confidence (55), High risk → Low confidence (25) |

### Total Decision Score Formula

```
decisionScore = (trust × 0.20) + (reliability × 0.20) + (operational × 0.20) + 
                (resource × 0.15) + (cost × 0.15) + (fairness × 0.10)
```

Where each pillar value is the confidence level (85, 55, or 25) derived from risk assessment.

**Result range:** 0-100 (higher is better — higher confidence across pillars)

**Decision bands:**
- **80-100:** Strong fit — high confidence across most pillars
- **60-79:** Acceptable fit — recommend with caveats or risk mitigation
- **40-59:** Marginal fit — recommend alternative or hybrid approach
- **0-39:** Poor fit — do not recommend; propose different architecture

---

## Pillar 1: Trust (20%)

**Definition:** How well does the solution protect data, enforce least privilege, and maintain compliance?

### Scoring Criteria (0-100)

| Confidence Level | Authentication | Authorization | Encryption | Compliance |
|-------------|---------------|--------------|------------|------------|
| **High confidence (85)** | MFA enforced, SSO with IdP, session management | CRUD/FLS enforced, sharing rules, least privilege | Shield Platform Encryption, TLS 1.2+ | GDPR, HIPAA, PCI-DSS controls |
| **Moderate confidence (55)** | Password + security questions, IP restrictions | Basic profile permissions, some FLS gaps | Encryption at rest (standard) | Partial compliance controls |
| **Low confidence (25)** | Password only, no MFA | System context (`without sharing`), no FLS | No encryption at rest | No compliance controls |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Data exposure** | All queries use `USER_MODE` or `WITH SECURITY_ENFORCED` | Some queries skip FLS checks | All queries in system context |
| **Authentication** | MFA enforced, SSO with IdP (Okta, Azure AD) | Password + IP restrictions | Password only, no MFA |
| **Sharing model** | Private with explicit sharing rules | Public Read/Write | Public Read/Write with `without sharing` code |
| **Secrets management** | Named Credentials, External Credentials | Hardcoded in Custom Settings | Hardcoded in Apex/metadata |
| **Audit trail** | Shield Event Monitoring, 10-year field history | Setup Audit Trail (6 months) | No audit trail |

### Example Scoring

**High confidence (85):**
- MFA enforced for all users
- All Apex uses `with sharing` and `USER_MODE`
- Shield Platform Encryption for PII fields
- Named Credentials for external system auth
- Event Monitoring logs streamed to SIEM

**Moderate confidence (55):**
- Password + IP restrictions
- Most Apex uses `with sharing`, some FLS gaps
- Standard encryption at rest
- Custom Settings for external credentials

**Low confidence (25):**
- No MFA enforcement
- Apex uses `without sharing`
- No encryption at rest
- Hardcoded credentials in code

---

## Pillar 2: Reliability (20%)

**Definition:** How well does the solution handle failures, scale with load, and recover from incidents?

### Scoring Criteria (0-100)

| Confidence Level | Fault Tolerance | Scaling | Recovery | Governor Limit Resilience |
|-------------|----------------|---------|----------|--------------------------|
| **High confidence (85)** | Graceful degradation, retry logic, circuit breakers | Handles 10x expected load | Automated failover, RTO <1 hour | Async patterns, bulk-safe, <50% of limits |
| **Moderate confidence (55)** | Basic error handling, manual retry | Handles 2x expected load | Manual recovery, RTO <4 hours | Some bulk-safe patterns, 50-80% of limits |
| **Low confidence (25)** | No error handling, crashes on failure | Only handles expected load | No recovery plan, RTO >24 hours | SOQL/DML in loops, >80% of limits |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Governor limits** | Uses <50% of limits (SOQL, DML, CPU) | Uses 50-80% of limits | Uses >80% of limits, frequent failures |
| **Async patterns** | Queueable, Batch Apex, Platform Events | Future methods, scheduled flows | Synchronous only, no async fallback |
| **Error handling** | Try/catch, fault connectors, DLQ | Basic try/catch, some error logging | No error handling, silent failures |
| **Data volume** | Handles 10x expected load via bulkification | Handles 2x expected load | Only handles expected load |
| **External dependencies** | Circuit breaker, fallback cache | Retry logic, timeout handling | No error handling for callouts |

### Example Scoring

**High confidence (85):**
- All SOQL/DML operations are bulk-safe
- Uses Queueable for async processing (200 SOQL limit)
- Platform Events for decoupled pub/sub
- Circuit breaker for external callouts
- Batch Apex for large data volumes (50M records)

**Moderate confidence (55):**
- Most SOQL/DML is bulk-safe, some edge cases
- Uses Future methods for async processing
- Basic retry logic for callouts
- Handles 2x expected load

**Low confidence (25):**
- SOQL/DML in loops
- All processing is synchronous
- No error handling for callouts
- Fails with >200 records

---

## Pillar 3: Operational Excellence (20%)

**Definition:** How easy is the solution to operate, monitor, and evolve over time?

### Scoring Criteria (0-100)

| Confidence Level | Deployment | Observability | Automation | Maintainability |
|-------------|------------|---------------|------------|-----------------|
| **High confidence (85)** | CI/CD with validation gates, rollback capability | Comprehensive logging, real-time alerting, dashboards | Fully automated (zero manual steps) | Self-documenting, modular, well-tested |
| **Moderate confidence (55)** | Manual deploy with validation, limited rollback | Basic logging, reactive monitoring | Partially automated (some manual steps) | Documented, some coupling, partial test coverage |
| **Low confidence (25)** | Ad-hoc deploy, no validation, no rollback | Minimal logging, no monitoring | Fully manual | Undocumented, tightly coupled, no tests |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Deployment complexity** | Metadata-only, declarative, packaged | Mix of clicks and code, multi-org | Manual scripts, org-specific |
| **Monitoring visibility** | Event Monitoring, Shield audit, real-time alerts | Setup Audit Trail, periodic reviews | No audit trail, reactive only |
| **Change velocity** | Deploy daily, automated tests | Deploy weekly, manual validation | Deploy monthly, no tests |
| **Runbook completeness** | Automated runbooks, playbooks | Documented procedures | Tribal knowledge |
| **Team capability** | Team has skills to build and maintain | Team can maintain with some upskilling | Team lacks skills, dependent on vendor/contractor |

### Example Scoring

**High confidence (85):**
- CI/CD pipeline with scratch org validation
- Event Monitoring logs streamed to SIEM
- Automated deployment via Salesforce CLI
- Comprehensive test suite (85%+ coverage)
- Runbooks for incident response

**Moderate confidence (55):**
- Manual deployment with validation
- Setup Audit Trail reviewed monthly
- Some automated tests (60% coverage)
- Basic documentation

**Low confidence (25):**
- Ad-hoc changes in production
- No audit trail or monitoring
- No tests or documentation

---

## Pillar 4: Resource Optimization (15%)

**Definition:** How efficiently does the solution use platform resources and deliver fast response times?

### Scoring Criteria (0-100)

| Confidence Level | Latency | Compute Efficiency | Caching | Right-Sizing |
|-------------|---------|-------------------|---------|--------------|
| **High confidence (85)** | <1s response, <2s for complex queries | <50% heap, <50% CPU | Platform Cache, CDN, efficient patterns | Uses lowest edition and fewest add-ons needed |
| **Moderate confidence (55)** | 1-3s response, 3-5s for complex queries | 50-80% heap, 50-80% CPU | Custom object cache, no CDN | Slightly over-provisioned |
| **Low confidence (25)** | >3s response, >5s for complex queries | >80% heap, >80% CPU, timeouts | No caching | Significantly over-provisioned |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Query selectivity** | Indexed filters in WHERE, <10K rows | Some non-selective queries, <50K rows | Full table scans, >50K rows |
| **Heap usage** | <3 MB (sync), <6 MB (async) | 3-6 MB (sync), 6-12 MB (async) | >6 MB (sync), >12 MB (async) |
| **CPU time** | <5s (sync), <30s (async) | 5-10s (sync), 30-60s (async) | >10s (sync), >60s (async) |
| **Infrastructure** | Native Salesforce (Flow, LWC) | Some external apps or APIs | Heavy external infrastructure (Heroku, AWS, VMs) |
| **Async patterns** | Queueable, Batch Apex, Platform Events | Some Future methods | All synchronous processing |

### Example Scoring

**High confidence (85):**
- SOQL queries use indexed fields (AccountId, OwnerId)
- LWC uses lazy loading for large datasets
- Platform Cache for frequently accessed data
- Heap usage <3 MB, CPU time <5s
- Native Salesforce only, Enterprise edition

**Moderate confidence (55):**
- Some non-selective queries (Industry, CreatedDate)
- LWC loads in 1-3s
- Custom object cache for config data
- Heap usage 3-6 MB, CPU time 5-10s

**Low confidence (25):**
- Full table scans (LIKE '%...', no WHERE clause)
- LWC blocks on slow Apex calls
- No caching
- Heavy external infrastructure, over-provisioned

---

## Pillar 5: Cost Optimization (15%)

**Definition:** How cost-effective is the solution in terms of licensing, development effort, and operational overhead?

### Scoring Criteria (0-100)

| Confidence Level | Licensing Costs | Development Effort | Operational Overhead | API Usage |
|-------------|----------------|-------------------|---------------------|-----------|
| **High confidence (85)** | No add-ons, standard edition | Clicks-only, <1 week | Managed services, zero maintenance | <50% of daily limit, Bulk API |
| **Moderate confidence (55)** | Some add-ons, Enterprise edition | Mix of clicks and code, 1-4 weeks | Some manual operations | 50-80% of daily limit, REST API |
| **Low confidence (25)** | Many add-ons, Unlimited edition | Heavy Apex/LWC, >4 weeks | High manual overhead | >80% of daily limit, polling |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **License cost** | No add-ons (Sales/Service Cloud only) | 1-2 add-ons (Shield, Experience Cloud) | 3+ add-ons (MuleSoft, Data 360, Shield) |
| **Edition requirement** | Professional or Enterprise | Enterprise or Unlimited | Unlimited or custom SKU |
| **Development effort** | Clicks-only (flows, declarative) | Mix of clicks and code (<1K LOC) | Heavy code (>5K LOC, complex logic) |
| **Operational overhead** | Managed services (Flow, declarative) | Some manual operations (Batch Apex) | High manual operations (cron jobs, scripts) |
| **Time to value** | Deploy MVP in <2 weeks | Deploy MVP in 2-4 weeks | Deploy MVP in >4 weeks |

### Example Scoring

**High confidence (85):**
- No add-ons required (Sales Cloud only)
- Professional or Enterprise edition
- Clicks-only solution (flows, validation rules)
- Zero operational overhead
- MVP delivered in <2 weeks

**Moderate confidence (55):**
- Shield add-on for encryption
- Enterprise edition required
- Mix of flows and Apex (500 LOC)
- Some manual operations (weekly batch job)
- MVP in 3 weeks

**Low confidence (25):**
- MuleSoft, Data 360, Shield add-ons
- Unlimited edition required
- Heavy Apex/LWC (10K LOC)
- High manual overhead (daily scripts)
- MVP in >6 weeks

---

## Pillar 6: Fairness (10%)

**Definition:** How well does the solution provide equitable access, avoid bias, and consider diverse user needs?

### Scoring Criteria (0-100)

| Confidence Level | Accessibility | Equitable Access | Bias Avoidance | Inclusive Design |
|-------------|---------------|-----------------|----------------|-----------------|
| **High confidence (85)** | WCAG 2.1 AA compliant, screen reader tested | All user personas served equally | AI/automation outputs reviewed for bias | Multi-language, multi-device, multi-persona |
| **Moderate confidence (55)** | Basic accessibility (standard components) | Most personas served, some gaps | Some bias review, not systematic | Single language, desktop-focused |
| **Low confidence (25)** | No accessibility consideration | Some personas excluded or disadvantaged | No bias review, potential for unfair outcomes | Single persona assumption |

### Risk Assessment

| Risk Factor | Low Risk → High Confidence (85) | Med Risk → Moderate Confidence (55) | High Risk → Low Confidence (25) |
|-------------|-------------------------|---------------------------|-------------------------|
| **Accessibility** | Standard LWC (inherits SLDS accessibility), tested with assistive tech | Standard components without explicit testing | Custom UI without accessibility consideration |
| **Persona coverage** | All user roles considered in design | Primary personas covered, edge cases missed | Solution designed for one persona, others excluded |
| **Automation bias** | Assignment/routing rules reviewed for fairness, auditable criteria | Rules exist but not reviewed for bias | Opaque logic that may disadvantage groups |
| **Data equity** | Reporting and dashboards available to all relevant roles | Some roles lack visibility into data affecting them | Data access concentrated, decisions opaque |

### Example Scoring

**High confidence (85):**
- Standard LWC components (SLDS accessibility built-in)
- All user personas considered in flow design
- Assignment rules use transparent, auditable criteria
- Multi-language support via custom labels
- Dashboards available to all affected roles

**Moderate confidence (55):**
- Standard components without explicit accessibility testing
- Primary user personas covered, field reps underserved
- Territory assignment rules not reviewed for geographic bias
- English-only, desktop-optimised

**Low confidence (25):**
- Custom UI without accessibility consideration
- Solution designed for admins only, end users excluded
- AI-driven routing with no bias review
- No visibility into how decisions affect different groups

---

## Consistency Rules

1. **Highest decision score MUST be the recommendation.** The option with the top decision score is always the recommended choice. If the narrative says "Option A is recommended" but Option B has a higher decision score, the data is wrong — fix it.

2. **Risk-to-confidence mapping is mandatory.** Every pillar uses fixed confidence values: Low risk = 85, Med risk = 55, High risk = 25. No arbitrary values.

3. **Tie-breaking rules:**
   - Prefer fewer "high" risks (a 75 with 1 high risk beats a 75 with 2 high risks)
   - Weight criteria by priority (Trust > Reliability > Operational Excellence > Resource > Cost > Fairness)
   - Consider development effort as secondary (faster to deploy beats slower if scores are tied)

4. **Data overrides narrative.** If the scoring table shows Option A at 82 and Option B at 78, but the recommendation text says "Option B is recommended," the recommendation is WRONG. Fix it to match the data.

5. **No "best of both worlds" hedging.** Do not say "Option A is recommended, but Option B is also good." Pick the top decision score and commit.

---

## Example Scoring: Record-Triggered Flow vs. Apex Trigger

**Scenario:** Automate lead assignment based on territory rules. 500 leads/day, 10 territories, simple logic (zip code lookup).

### Option A: Record-Triggered Flow

| Pillar | Confidence | Governing Criterion | Reasoning |
|--------|------------|--------------------:|-----------|
| **Trust** (20%) | 85 | Data Exposure = Low risk | Runs in user context, respects sharing rules |
| **Reliability** (20%) | 55 | Error Handling = Med risk | Bulk-safe by default, but limited to fault connectors for error handling |
| **Operational Excellence** (20%) | 85 | Deployment Complexity = Low risk | Declarative, no code, easy to change |
| **Resource Optimization** (15%) | 55 | Caching = Med risk | Fast for simple logic but no caching for repeated territory lookups |
| **Cost Optimization** (15%) | 85 | License Cost = Low risk | Zero dev cost, no Apex licenses needed |
| **Fairness** (10%) | 85 | Persona Coverage = Low risk | Transparent assignment rules, auditable by non-developers |
| **Decision Score** | **74.5** | | (85×0.20) + (55×0.20) + (85×0.20) + (55×0.15) + (85×0.15) + (85×0.10) |

### Option B: Apex Trigger

| Pillar | Confidence | Governing Criterion | Reasoning |
|--------|------------|--------------------:|-----------|
| **Trust** (20%) | 85 | Data Exposure = Low risk | Uses `with sharing` and `USER_MODE` |
| **Reliability** (20%) | 85 | Governor Limits = Low risk | Bulk-safe, error handling, unit tests |
| **Operational Excellence** (20%) | 55 | Deployment Complexity = Med risk | Requires tests, deploy pipeline, code reviews |
| **Resource Optimization** (15%) | 85 | Heap Usage = Low risk | Can cache territory lookups in memory |
| **Cost Optimization** (15%) | 55 | Development Effort = Med risk | Requires developer time (1 week) and ongoing maintenance |
| **Fairness** (10%) | 55 | Data Equity = Med risk | Logic is code-reviewable but less transparent to non-developers |
| **Decision Score** | **71.5** | | (85×0.20) + (85×0.20) + (55×0.20) + (85×0.15) + (55×0.15) + (55×0.10) |

**Recommendation:** **Option A: Record-Triggered Flow** (decision score 74.5 > 71.5)

**Rationale:** Flow wins primarily on Cost Optimization (85 vs 55) due to zero development effort and no Apex dependency. Apex wins on Reliability (85 vs 55) due to comprehensive error handling beyond fault connectors. The 3-point gap is driven by Flow's cost and fairness advantages outweighing Apex's reliability edge.

---

**Last updated:** 2026-05-18 (aligned with Salesforce Well-Architected Framework)
