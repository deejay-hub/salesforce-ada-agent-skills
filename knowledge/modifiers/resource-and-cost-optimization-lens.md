# Resource and Cost Optimization Lens

This modifier applies a value-per-cost perspective to all architecture decisions. The Salesforce Well-Architected Framework treats resource and cost optimization as one pillar: resource optimization is the mechanism (efficient use of governor limits, queries, compute, storage, and consumption) and cost optimization is the outcome (maximum long-term business value per unit of spend). Analyze both together, never one in isolation.

## Pillar Weight Adjustment

When the Resource and Cost Optimization lens is active:

- **Resource and Cost Optimization pillar weight:** 35% (increased from 20%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Trust: 20.31% (down from 25%)
  - Reliability: 20.31% (down from 25%)
  - Operational Excellence: 16.25% (down from 20%)
  - Fairness: 8.13% (down from 10%)

## Framework Alignment

Source pillars:
- Resource and Cost Optimization: https://architect.salesforce.com/docs/architect/well-architected/guide/resource-and-cost-optimization.html
- Agentic Enterprise Resource and Cost Optimization: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-resource-and-cost-optimization.html

### Principles

Every analysis under this lens must be traceable to one or more of the pillar's seven principles:

1. **Optimize for total cost of ownership.**
2. **Align spend with business value.**
3. **Design within platform limits.**
4. **Optimize the resources you already pay for.**
5. **Right-size licensing and consumption to actual use.**
6. **Establish financial governance and cost-aware culture.**
7. **Practice continuous optimization.**

### Shared Responsibility

Salesforce manages multitenant resource allocation, the governor limit architecture, the query optimizer and execution engine, infrastructure scaling, and platform performance optimization. Salesforce also controls the commercial model (editions, license types, add-ons, credit metering).

The customer is responsible for four interconnected areas, which structure the analysis below:

| Area | Scope |
|------|-------|
| **Performance** | Governor limits, query selectivity, bulkification, async processing, caching, data skew |
| **Code organization** | Trigger handler, service layer, and selector patterns; mixed DML avoidance |
| **Packaging** | 2GP managed, unlocked, and 1GP packages; LWC and Apex modularity |
| **Data** | Data model design, lifecycle and archival (Big Objects, external storage), storage monitoring |

### Platform Tooling

Reference these tools when recommending how to measure or validate resource use: Scale Center, Query Plan Tool, ApexGuru, Salesforce Code Analyzer, Event Monitoring, and Proactive Monitoring.

## Analysis Requirements

### 1. TCO Comparison (Mandatory)

Every decision must start with a total cost of ownership comparison across the framework's cost categories. Use a 3-5 year horizon.

| Option | License and Consumption | Implementation | Operational | Maintenance | Integration | Change | Indirect | 5-Year TCO |
|--------|------------------------|----------------|-------------|-------------|-------------|--------|----------|-----------|
| Option A | [cost] | [cost] | [cost/yr] | [cost/yr] | [cost] | [cost] | [qualitative or cost] | [sum] |
| Option B | [cost] | [cost] | [cost/yr] | [cost/yr] | [cost] | [cost] | [qualitative or cost] | [sum] |

**Direct cost categories:**

1. **License and consumption** — user licenses, feature licenses and add-ons, AppExchange or AgentExchange subscriptions, consumption credits, storage, API capacity
2. **Implementation** — design, build, test, deploy, document
3. **Operational** — admin support, monitoring, incident response, training
4. **Maintenance** — ongoing fixes, release upgrades, test upkeep
5. **Integration** — middleware, API capacity, connector licenses
6. **Change** — cost of modifying the solution as requirements evolve

**Indirect cost categories:**

- **Organizational costs** — skills, hiring, and team structure required to own the solution
- **Technical debt interest** — the ongoing cost of shortcuts taken now
- **Governance overhead** — reviews, approvals, and controls the solution adds
- **Unused functionality** — paying for capability that is never adopted

Never present dollar figures as platform facts. Use customer-supplied rates and licensing quotes, or mark figures as "Illustrative" when no source exists.

### 2. Build vs. Buy

For every custom build proposal, compare against AppExchange packages (and AgentExchange listings for agent components, actions, and templates). Judge on the framework's four factors:

| Factor | Question | Favors Buy When | Favors Build When |
|--------|----------|-----------------|-------------------|
| **Differentiation** | Does this capability set us apart competitively? | Commodity capability | Strategic differentiator |
| **Time to value** | How quickly must this deliver? | Value is needed this quarter | Timeline allows a build |
| **Organizational capability** | Can the team build and maintain this long term? | Skills are scarce | Team has the skills and capacity |
| **Exit cost** | What does it cost to leave this choice later? | Vendor offers clean data export and migration path | Lock-in risk outweighs build cost |

Also capture vendor stability, customization limits, and 5-year TCO for both paths. Record the build-vs-buy rationale in the decision record.

### 3. Performance and Resource Efficiency

Design within platform limits and leave headroom. The Reliability pillar recommends a 70% operational ceiling on governor limits; apply the same ceiling here when assessing efficiency.

**Heap and CPU budget** (verify current limits in documentation before citing):
- Project heap: (average record size) x (result count). Flag any projection that crosses 70% of the sync or async heap limit.
- Project CPU: profile current execution, identify hotspots (nested loops, complex regex, formula evaluation), and project at 10x volume.
- Mitigation: query in chunks, use iterators, reduce algorithmic complexity, offload to async, cache computed values.

**Query selectivity:**
- Prefer filters on indexed fields (Id, Name, OwnerId, RecordTypeId, lookup and master-detail fields, External ID and unique fields, custom indexes).
- Use the Query Plan Tool to confirm index use rather than a table scan.
- Leading-wildcard `LIKE` filters and null checks on non-indexed fields are non-selective; consider SOSL for text search.

**Bulkification:**
- Collect Ids into sets, query once, and work from maps keyed by Id.
- Never place a query or DML statement inside a loop.
- Sort bulk loads by parent Id to reduce lock contention.

**Async selection** — choose async deliberately, not by reflex:

| Need | Pattern |
|------|---------|
| Volume | Batch Apex |
| Sequence (chained steps) | Queueable Apex |
| Decoupling | Platform Events |
| Simple fire-and-forget | Future methods |
| Time-based | Scheduled Apex |

Sync remains correct when the result is needed immediately in the UI and volume is guaranteed low.

**Caching:**

| Data Type | Change Frequency | Strategy |
|-----------|------------------|----------|
| Record types, describe data | Rarely | Static variable, plus Org Cache across transactions |
| Configuration | Occasionally | Custom Metadata Types (platform-cached), static variable per transaction |
| Reference records | Daily | Org Cache with TTL |
| Per-user preferences | Per session | Session Cache |
| UI record data | Varies | Lightning Data Service |

Do not cache PII in the org partition, and do not cache volatile data where invalidation overhead exceeds the savings.

**Data skew:** flag any parent with more than 10,000 child records, and any ownership skew (one user owning very large record volumes). Both degrade sharing recalculation and locking.

**Code organization:** prefer a trigger handler, a service layer, and selectors over logic scattered across triggers. Avoid mixed DML.

**Packaging:** prefer modular unlocked or 2GP packages and composable LWC over one monolithic metadata set, so that change cost stays proportional to the change.

**Data lifecycle:** define retention and archival (Big Objects or external storage) up front, and monitor storage against allocation.

### 4. Licensing and Consumption

Right-size to actual use:

1. **Licenses** — identify underused or over-provisioned licenses (users who only view reports, add-ons assigned org-wide but used by a few people). Recommend the lowest license type that meets the need.
2. **Consumption credits** — Agentforce, Data 360, and other AI capabilities are metered per use. The pillar warns that "consumption can spike without any provisioning decision being made." Model expected burn and set thresholds that trigger review.
3. **Sandboxes** — match sandbox type to purpose (Developer or Developer Pro for build, Partial or Full Copy for integration testing, UAT, and load testing). Flag idle sandboxes.
4. **Multi-org** — count duplicated licenses, integration, and governance as a multi-org cost.
5. **API and integration** — measure daily API consumption against entitlement. Prefer event-driven patterns (Platform Events, Change Data Capture) over polling, and batch with Composite API or Bulk API 2.0. Verify current API allocation and counting rules in documentation before citing them.

| Pattern | When to Use |
|---------|-------------|
| REST API (single record) | Real-time, very low volume |
| Composite API | Real-time, multi-record updates |
| Bulk API 2.0 | Batch loads, nightly sync |
| Platform Events / Change Data Capture | Event-driven push, data sync |

### 5. Monitoring and Financial Governance

- **Dashboards** — license utilization, capacity (storage, API, credits), and investment against value delivered.
- **Budgets** — set budgets for License, Storage, API, and Sandbox spend, with alerts at 70% and 85% of each.
- **Allocation** — choose showback (visibility only) or chargeback (costs billed to consuming teams), or a hybrid.
- **FinOps practices** — make cost visible, assign owners, and review regularly.
- **Cost-aware architecture review** — every design review records the cost impact and the cost of the alternatives.

### 6. Sustainability

Efficiency and sustainability are the same discipline. Check for:
- Inactive or redundant automation still executing
- Heavy jobs that could be scheduled off-peak
- Recomputation of values that could be stored or cached
- Data retained beyond its purpose
- Chatty integrations that could be batched or event-driven
- Inefficient agents (excess actions, oversized prompts, unbounded retrieval)

### 7. Agentic Cost Considerations

When the decision involves Agentforce or other AI capabilities, add this analysis and apply `knowledge/modifiers/agentic-enterprise-lens.md` for full depth.

- **Pricing model is architecture.** The commercial model determines what to optimize:
  - **Flex Credits** — billed per agent action, with a fixed cost per action up to a defined token ceiling. A retrieve, reason, and update flow consumes three actions. To reduce cost, consolidate actions and cache repeated requests.
  - **Per-Conversation** — a flat fee per conversation regardless of turns. To reduce cost, improve first-conversation resolution and containment.
  - **Per-User** — an unmetered per-user-per-month add-on for employees, layered on a consumption model. To get value, drive adoption.
- **Inefficiency multiplies.** Agents repeat any inefficiency across every conversation.
- **Monitoring** — use Digital Wallet as the primary consumption monitor; do not build custom dashboards that duplicate it. Set credit burn alerts at 70% and 85% per agent or use case, and attribute cost per outcome (per case, lead, or order).
- **Grounding** — cap retrieval with top-k limits and metadata filters (retrieving 50 results when 5 inform the answer is waste). Data 360 grounding also consumes data-processing capacity.
- **Context** — keep prompts short, summarize history instead of carrying it in full, and return only essential fields.
- **Composition** — route deterministic paths to Flow, keep orchestrators narrow, cap chain depth, and cap retries at 2-3 with backoff.
- **SOQL across chains** — chains of 10-15 actions can exhaust synchronous SOQL limits; budget queries across the whole chain.

## Output Enhancements

### Add "Resource and Cost Considerations" Section to Every Output

Every response must include a dedicated section covering:

- 5-year TCO comparison (direct and indirect categories)
- Build vs. buy assessment (when applicable)
- Governor limit headroom at current and 10x volume (against the 70% ceiling)
- Query selectivity, bulkification, and caching strategy
- Licensing, consumption, sandbox, and API impact
- Governance controls (budgets, alerts, showback or chargeback)
- Agentic consumption model and controls (when applicable)

### Risk Assessment Prioritization

When listing risks, prioritize resource exhaustion and cost escalation risks. Use this format:

**Risk:** [resource limit or cost driver]
**Likelihood:** High/Medium/Low
**Impact:** [transaction failure, throttling, overage, unplanned spend]
**Mitigation:** [optimization or governance control]

Example (illustrative numbers):
> **Risk:** Heap limit exceeded when querying 10,000 Accounts synchronously
> **Likelihood:** High (about 1 KB per record x 10,000 records = about 10 MB, above the sync heap limit)
> **Impact:** Transaction rollback; user sees a heap size error
> **Mitigation:** Process in Batch Apex, or query only required fields in bounded chunks

### Trade-off Analysis: Value-per-Cost Bias

When comparing options, apply a value-per-cost bias and make the price of each non-cost benefit explicit:

> **Value-optimized choice:** Option A ([TCO])
> **Performance-optimized choice:** Option B ([TCO], [measured benefit])
>
> **Cost of the benefit:** [delta]
> **Justification required:** The benefit must deliver more than [delta] in business value (reduced downtime, faster cycle time, avoided rework)

Call out the framework's named tensions where they apply:
- **Trust vs. Resource and Cost Optimization** — encryption and logging consume resources.
- **Reliability vs. Resource and Cost Optimization** — let SLOs set the level of investment rather than defaulting to maximum availability.

## Decision Criteria Adjustments

### Reject or Flag as High-Risk

The following patterns are automatically flagged as high-risk under this lens:

1. **A query or DML statement inside a loop**
2. **Non-selective queries** that scan full tables
3. **Testing only against small volumes** in a Developer sandbox
4. **Async by reflex** — async chosen without a volume, sequence, or decoupling reason
5. **Caching too aggressively** — volatile data cached, or PII cached in the org partition
6. **Data skew** — parents with more than 10,000 children, or ownership skew
7. **Over-licensing** — add-ons or license types beyond actual use
8. **Idle sandboxes**
9. **Chatty integrations and polling** where events or batching would do
10. **Leftover Process Builders** — past end of support as of December 31, 2025; migrate to Flow
11. **Unbounded agent loops or retrieval** — agents that can burn credits faster than oversight can intervene

### Favor Value-Efficient Options

When multiple options meet requirements, prefer:

- Existing platform capability over new add-ons ("Optimize the resources you already pay for")
- Declarative over programmatic when both meet requirements, because maintenance and test costs are lower
- AppExchange or AgentExchange over custom build for commodity capability; build only for differentiators
- Selective, indexed queries and bulkified code
- Event-driven integration over polling; Bulk API 2.0 or Composite API over per-record calls
- Custom Metadata Types and Platform Cache over repeated queries
- Archival to Big Objects or external storage over unbounded active storage
- Deterministic Flow paths over agent reasoning when the path is known

### Cost-Justification Thresholds

Require explicit business justification (business value, cost of inaction, ROI) when an option:
- Adds a new license type, add-on, or consumption model
- Requires new external infrastructure (Heroku, MuleSoft, or other hosting)
- Pushes projected API, storage, or credit consumption past the 70% budget threshold
- Chooses a custom build where a package meets most requirements

## Example Output (Resource and Cost Optimization Lens Applied)

### User Question
> Should I cache record type IDs in a static variable or query them every time?

### Standard Response
> Cache record type IDs in a static variable to avoid repeated SOQL queries. Use a singleton pattern so the cache is populated once per transaction, then reused.

### Resource and Cost Optimization Lens Response
> **Recommendation:** Cache record type IDs in a static variable, backed by Org Cache.
>
> **Resource and Cost Considerations:**
>
> **Principle:** "Design within platform limits" and "Optimize the resources you already pay for."
>
> **SOQL headroom (illustrative):**
> - Without cache: 1 query per lookup x 50 lookups per transaction = 50 queries, half the synchronous SOQL limit and close to the 70% ceiling once other automation runs
> - With static cache: 1 query per transaction
>
> **Caching strategy:**
> - **Tier 1 (static variable):** reuse within a transaction, no cost
> - **Tier 2 (Org Cache):** reuse across transactions and users, using existing Platform Cache allocation
> - Record types change rarely, so invalidation overhead is negligible. The data is not PII, so org partition caching is appropriate.
>
> **Implementation:**
> ```apex
> public class RecordTypeCache {
>     private static final String CACHE_KEY = 'RecordTypes';
>     private static Map<String, Id> staticCache;
>
>     public static Id getRecordTypeId(String objectName, String developerName) {
>         if (staticCache == null) {
>             if (Cache.Org.contains(CACHE_KEY)) {
>                 staticCache = (Map<String, Id>) Cache.Org.get(CACHE_KEY);
>             } else {
>                 staticCache = new Map<String, Id>();
>                 for (RecordType rt : [
>                     SELECT Id, DeveloperName, SobjectType
>                     FROM RecordType
>                     WHERE IsActive = true
>                 ]) {
>                     staticCache.put(rt.SobjectType + '.' + rt.DeveloperName, rt.Id);
>                 }
>                 Cache.Org.put(CACHE_KEY, staticCache, 86400);
>             }
>         }
>         return staticCache.get(objectName + '.' + developerName);
>     }
> }
> ```
>
> **TCO:** No new license or add-on cost. A few hours to implement.
>
> **When not to use Org Cache:** if only one lookup happens per transaction, the static variable alone is sufficient.

## Checklist for Resource and Cost Optimization Lens Output

When this lens is active, verify every response includes:

- [ ] Principle(s) from the pillar that the recommendation serves
- [ ] 5-year TCO comparison covering direct and indirect cost categories
- [ ] Build vs. buy assessment on differentiation, time to value, organizational capability, and exit cost (when applicable)
- [ ] Governor limit headroom at current and 10x volume against the 70% ceiling
- [ ] Query selectivity, bulkification, async rationale, and caching strategy
- [ ] Data skew and data lifecycle assessment
- [ ] Licensing, consumption credit, sandbox, multi-org, and API impact
- [ ] Governance controls: budgets with 70% and 85% alerts, showback or chargeback
- [ ] Sustainability check (inactive automation, off-peak scheduling, retention)
- [ ] Agentic consumption model, Digital Wallet monitoring, and retrieval caps (when agents are involved)
- [ ] Flagged anti-patterns
- [ ] Value-per-cost trade-off with the named cross-pillar tensions
- [ ] All dollar figures sourced from the customer or labeled "Illustrative"
