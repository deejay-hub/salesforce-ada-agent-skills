# Resource Optimization Lens

This modifier applies a resource-efficiency perspective to all architecture decisions, prioritizing compute efficiency, query selectivity, heap/CPU optimization, caching strategies, and right-sizing.

## Pillar Weight Adjustment

When the Resource Optimization lens is active:

- **Resource Optimization pillar weight:** 35% (increased from 15%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Trust: 15.29% (down from 20%)
  - Reliability: 15.29% (down from 20%)
  - Operational Excellence: 15.29% (down from 20%)
  - Cost Optimization: 11.47% (down from 15%)
  - Fairness: 7.65% (down from 10%)

## Analysis Requirements

### Heap and CPU Budget Analysis (Mandatory)

For every Apex transaction, project resource consumption:

1. **Heap Budget (6 MB sync / 12 MB async)**
   - Calculate: (avg record size) × (query result count) = heap consumed
   - Example: 1 KB per Account × 10,000 results = 10 MB → exceeds sync limit
   - Mitigation: Query in chunks (1,000 at a time), use iterator, or Batch Apex

2. **CPU Budget (10 sec sync / 60 sec async)**
   - Profile current execution (Debug log → CUMULATIVE_PROFILING)
   - Identify hotspots: nested loops (O(n²)), complex regex, formula evaluation
   - Project: If 100 records take 2 seconds, 1,000 records = 20 seconds → exceeds sync
   - Mitigation: Reduce algorithm complexity, offload to async, cache computed values

3. **Per-Transaction Projection**
   - Current volume: [X] records
   - Heap usage: [Y] MB
   - CPU time: [Z] seconds
   - 10x projection: [10Y] MB, [10Z] seconds → Flag if exceeds limits

4. **Governor Limit Dashboard**
   - Track trends: Are we approaching limits over time?
   - Alert thresholds: >70% heap, >5 sec CPU (leaves buffer for spikes)
   - Optimization triggers: When crossing 70%, prioritize resource optimization

### Query Selectivity Assessment

For every SOQL query, assess selectivity (how many rows scanned vs. returned):

1. **Indexed Fields (Selective)**
   - Standard indexed: Id, Name, OwnerId, RecordTypeId, Master-Detail, External ID
   - Custom indexed: Fields marked "External ID" or "Unique", manual index request
   - Compound indexes: Two fields with custom index (request via Salesforce Support)

2. **Selective Filter Criteria**
   - **Highly selective (<1000 rows):** Id, External ID, Unique field
   - **Selective (<10% of table):** Indexed field with narrow filter (Industry = 'Technology')
   - **Non-selective (>10% of table):** Non-indexed field, LIKE '%term%', null checks on non-indexed

3. **Query Plan Review**
   - Use Query Plan tool (Developer Console → Query Editor → Query Plan tab)
   - Look for: "Table Scan" (bad), "Index Scan" (good)
   - Optimize: Add indexed field to WHERE, reorder filters (indexed first), use LIMIT

4. **Selectivity Rules of Thumb**
   | Filter | Selectivity | Index Required | Example |
   |--------|-------------|----------------|---------|
   | Id = 'xxx' | Highest | Yes (automatic) | `WHERE Id = '001xx000003GYRA'` |
   | RecordTypeId = 'xxx' | High | Yes (automatic) | `WHERE RecordTypeId = '012xx000000XYZ'` |
   | External_ID__c = 'xxx' | High | Yes (automatic) | `WHERE Order_Number__c = 'ORD-12345'` |
   | CreatedDate = LAST_N_DAYS:7 | Medium | Yes (automatic) | `WHERE CreatedDate = LAST_N_DAYS:7` |
   | Status__c = 'Active' | Low-Medium | Custom index | `WHERE Status__c = 'Active'` (if <10% of records) |
   | Description LIKE '%term%' | Non-selective | SOSL instead | Use SOSL: `FIND {term} IN ALL FIELDS` |

### Caching Strategy Evaluation

For every data access pattern, assess caching opportunity:

1. **Platform Cache (Org Cache / Session Cache)**
   - **When:** Reference data (record types, picklist values, config metadata)
   - **Org Cache:** Shared across users, sessions (1 MB–100 MB allocation)
   - **Session Cache:** Per-user session (250 KB–1 MB per session)
   - **Expiration:** Time-based (TTL) or explicit invalidation

2. **Static Variables**
   - **When:** Repeated lookups within same transaction
   - **Scope:** Single Apex execution context (resets per transaction)
   - **Pattern:** Singleton selector with static `Map<Id, SObject>` cache

3. **Custom Metadata Cache**
   - **When:** Configuration data (API endpoints, thresholds, feature flags)
   - **Built-in:** Salesforce caches custom metadata type records
   - **Pattern:** Query once, cache in static variable for transaction reuse

4. **Caching Decision Matrix**
   | Data Type | Change Frequency | Caching Strategy | TTL |
   |-----------|------------------|------------------|-----|
   | Record types | Never | Static variable + Org Cache | 24 hours |
   | Picklist values | Monthly | Org Cache | 1 hour |
   | Custom metadata | Weekly | Static variable (automatic) | N/A (Salesforce manages) |
   | User preferences | Per session | Session Cache | Session duration |
   | External API data | Hourly | Org Cache | 1 hour |
   | Frequently queried records | Daily | Org Cache | 30 minutes |

### API Call Budget Impact

For every integration, calculate and optimize API consumption:

1. **Measure Current Consumption**
   - Use Event Monitoring: `SELECT COUNT() FROM ApiEvent WHERE CreatedDate = TODAY`
   - Calculate: Daily calls ÷ daily limit = % consumed
   - Limit: 15,000 calls per user per 24 hours (base) + add-ons

2. **Polling vs. Streaming**
   - **Polling (inefficient):** HTTP callout every 5 min = 288 calls/day
   - **Streaming (efficient):** Platform Events, Change Data Capture = 0 API calls
   - **Break-even:** If polling >20 times/day, use streaming

3. **Batching Opportunities**
   - **Single REST call:** 1 record = 1 API call
   - **Composite API:** 25 records = 1 API call (25x reduction)
   - **Bulk API 2.0:** 10,000 records = 1 API call (10,000x reduction)

4. **Optimization Patterns**
   | Pattern | API Calls | When to Use |
   |---------|-----------|-------------|
   | REST API (single) | 1 per record | Real-time, <10 records |
   | Composite API | 1 per 25 records | Real-time, batch updates |
   | Bulk API 2.0 | 1 per 10,000 records | Batch loads, nightly sync |
   | Platform Events | 0 (not counted) | Event-driven, real-time push |
   | Change Data Capture | 0 (not counted) | Data sync, audit trail |

### Async vs. Sync Decision

For every operation, evaluate async suitability:

1. **Sync is Acceptable When:**
   - Latency critical (<1 second required)
   - Volume guaranteed low (<200 records)
   - Result needed in UI immediately
   - Simple operation (no complex computation)

2. **Async is Required When:**
   - Volume >200 records
   - CPU time >5 seconds (leaves buffer)
   - Heap usage >3 MB (leaves buffer)
   - External callout with >2 sec latency
   - Not user-initiated (scheduled, event-triggered)

3. **Async Pattern Selection**
   | Scenario | Pattern | Why |
   |----------|---------|-----|
   | Single operation, <5 sec | Queueable | Simple, chainable |
   | Bulk processing, hours OK | Batch Apex | Scales to 50M records |
   | Event-driven | Platform Events | Decoupled, scalable |
   | Scheduled | Scheduled Apex | Runs at specific time |

### Right-Sizing

For every environment and feature, assess whether it's over-provisioned:

1. **Salesforce Edition**
   - Current: Unlimited Edition ($300/user/month)
   - Required features: API access, custom objects, workflows
   - Assessment: Do you need 24/7 Premier Support? Full sandboxes?
   - Right-size: Enterprise Edition ($150/user) if features sufficient

2. **Storage Allocation**
   - Current: 100 GB file storage, 80 GB data storage
   - Usage: 20 GB files, 30 GB data
   - Assessment: 5x over-provisioned
   - Right-size: Reduce to 40 GB file, 50 GB data (saves $150/month per 10 GB)

3. **Feature Licenses**
   - Shield Platform Encryption: 200 licenses, 5 users active
   - Assessment: 195 unused licenses ($8,000/month waste)
   - Right-size: Assign to 5 users only, unassign from inactive

4. **Sandbox Sizes**
   - Full sandbox for every developer (10 devs = 10 Full sandboxes)
   - Assessment: Devs only need schema, not full data
   - Right-size: Developer Pro for devs, 1 Full for UAT ($50K/year savings)

## Output Enhancements

### Add "Resource Optimization Considerations" Section to Every Output

Every response must include a dedicated section covering:

- Heap/CPU budget analysis (current consumption, 10x projection)
- Query selectivity (indexed fields, query plan, optimization)
- Caching strategy (what data, which cache, TTL)
- API call budget impact (polling vs. streaming, batching)
- Async vs. sync decision (latency tolerance, volume threshold)
- Right-sizing opportunities (edition, storage, licenses, sandboxes)

### Risk Assessment Prioritization

When listing risks, prioritize resource exhaustion risks (heap overflow, CPU timeout, API limits). Resource risk format:

**Risk:** [resource limit]
**Likelihood:** High/Medium/Low
**Impact:** [transaction failure, throttling, performance degradation]
**Mitigation:** [optimization to implement]

Example:
> **Risk:** Heap size limit (6 MB) exceeded when querying 10,000 Accounts
> **Likelihood:** High (average Account record = 1 KB, 10K × 1 KB = 10 MB)
> **Impact:** Transaction rollback, user sees "Apex heap size too large" error
> **Mitigation:** Query in batches of 1,000 using SOQL LIMIT + OFFSET, or switch to Batch Apex

### Trade-off Analysis: Resource Efficiency Bias

When comparing options, apply a resource efficiency bias:

- Favor selective queries over full-table scans
- Favor cached lookups over repeated queries
- Favor async over sync (when latency acceptable)
- Favor batched operations over individual calls
- Call out resource inefficiencies: "This approach consumes [2x CPU] vs. [alternative]. Acceptable only if [one-time operation]."

## Decision Criteria Adjustments

### Reject or Flag as High-Risk

The following patterns are automatically flagged as high-risk under the resource lens:

1. **Full table scans** — Non-selective queries (>10% of records)
2. **No caching for repeated lookups** — Query record types 100 times per transaction
3. **Synchronous callouts blocking UI** — User waits 5 seconds for external API
4. **Excessive heap from large collections** — Loading 50,000 records into memory
5. **Polling integrations** — HTTP callout every minute (1,440 calls/day)
6. **Over-provisioned infrastructure** — Full sandboxes for developers, unused licenses

### Favor Resource-Efficient Options

When multiple options exist, prefer:

- SOQL with indexed WHERE clause over non-indexed
- Batch API over REST API for bulk operations
- Platform Events over polling for integrations
- Queueable over synchronous for >200 records
- Static variables over repeated SOQL
- Custom metadata over custom settings (automatic caching)
- Composite API over multiple REST calls

## Example Output (Resource Optimization Lens Applied)

### User Question
> Should I cache record type IDs in a static variable or query them every time?

### Standard Response
> Cache record type IDs in a static variable to avoid repeated SOQL queries. Use a singleton pattern to ensure the cache is populated once per transaction, then reused. This is a best practice for performance optimization.

### Resource Optimization Lens Response
> **Recommendation:** Cache record type IDs in static variable + Org Cache.
>
> **Resource Optimization Considerations:**
>
> **SOQL Budget Impact:**
> - Without cache: 1 SOQL per lookup × 50 lookups/transaction = 50 SOQL (half the 100 limit)
> - With static cache: 1 SOQL per transaction (99% reduction)
>
> **Heap Budget Impact:**
> - Record type cache: ~20 record types × 100 bytes = 2 KB (negligible)
> - ROI: 50 SOQL queries cost ~50 KB heap; cache costs 2 KB (96% heap savings)
>
> **CPU Budget Impact:**
> - Query time: ~10 ms per SOQL × 50 = 500 ms
> - Cache lookup: <1 ms × 50 = 50 ms
> - Savings: 450 ms per transaction (45% of 1-second user wait target)
>
> **Caching Strategy:**
> - **Tier 1 (static variable):** Single transaction reuse
> - **Tier 2 (Org Cache):** Cross-transaction, cross-user reuse (24-hour TTL)
> - **Why both:** Static is free, Org Cache survives transaction boundaries
>
> **Implementation:**
> ```apex
> public class RecordTypeCache {
>     private static final String CACHE_KEY = 'RecordTypes';
>     private static Map<String, Id> staticCache;
>     
>     public static Id getRecordTypeId(String objectName, String developerName) {
>         // Tier 1: Static cache (transaction-scoped)
>         if (staticCache == null) {
>             staticCache = new Map<String, Id>();
>             
>             // Tier 2: Org Cache (org-scoped)
>             if (Cache.Org.contains(CACHE_KEY)) {
>                 staticCache = (Map<String, Id>) Cache.Org.get(CACHE_KEY);
>             } else {
>                 // Tier 3: SOQL (last resort)
>                 for (RecordType rt : [
>                     SELECT Id, DeveloperName, SObjectType 
>                     FROM RecordType 
>                     WHERE IsActive = true
>                 ]) {
>                     staticCache.put(
>                         rt.SObjectType + '.' + rt.DeveloperName, 
>                         rt.Id
>                     );
>                 }
>                 Cache.Org.put(CACHE_KEY, staticCache, 86400); // 24-hour TTL
>             }
>         }
>         
>         return staticCache.get(objectName + '.' + developerName);
>     }
> }
> ```
>
> **Right-Sizing:**
> - Org Cache allocation: 1 MB minimum (record types = ~2 KB, well under limit)
> - No additional license cost
>
> **When NOT to Cache:**
> - Record types change frequently (daily) → Cache stale, invalidation overhead
> - Single lookup per transaction → Static cache sufficient, Org Cache overkill

## Checklist for Resource Optimization Lens Output

When Resource Optimization lens is active, verify every response includes:

- [ ] Heap/CPU budget analysis (current usage, 10x projection, mitigation)
- [ ] Query selectivity assessment (indexed fields, query plan, optimization)
- [ ] Caching strategy (static variable, Org Cache, Session Cache, TTL)
- [ ] API call budget impact (polling vs. streaming, batching, % consumed)
- [ ] Async vs. sync decision (latency threshold, volume threshold, pattern)
- [ ] Right-sizing opportunities (edition, storage, licenses, sandboxes)
- [ ] Flagged inefficient patterns (full table scans, no caching, polling, over-provisioning)
- [ ] Resource-biased trade-off (favor selective, cached, async, batched)
