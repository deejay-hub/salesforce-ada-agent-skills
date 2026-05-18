# Reliability Lens

This modifier applies a reliability-first perspective to all architecture decisions, prioritizing fault tolerance, scalability, governor limit resilience, and recovery planning.

## Pillar Weight Adjustment

When the Reliability lens is active:

- **Reliability pillar weight:** 35% (increased from 20%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Trust: 16.25% (down from 20%)
  - Operational Excellence: 16.25% (down from 20%)
  - Resource Optimization: 12.19% (down from 15%)
  - Cost Optimization: 12.19% (down from 15%)
  - Fairness: 8.12% (down from 10%)

## Analysis Requirements

### Governor Limit Resilience Analysis (Mandatory)

For every architecture decision, perform a governor limit risk assessment across all critical limits:

1. **SOQL Queries (100 per transaction)**
   - Count SOQL statements per code path
   - Identify queries inside loops (auto-fail)
   - Project volume: how many records trigger this path?
   - Mitigation: bulkify by collecting IDs, single query with `IN` clause

2. **DML Statements (150 per transaction)**
   - Count DML operations per execution context
   - Flag DML inside loops (auto-fail)
   - Calculate worst-case: trigger on 200 records × operations = ?
   - Mitigation: collect records in lists, single DML after loop

3. **Heap Size (6 MB synchronous / 12 MB asynchronous)**
   - Estimate collection sizes (record count × average size)
   - Flag large SOQL result sets without streaming
   - Project growth: current × 10x = still safe?
   - Mitigation: query in batches, use iterator patterns

4. **CPU Time (10 seconds synchronous / 60 seconds asynchronous)**
   - Identify complex computations (nested loops = O(n²))
   - Profile current execution time (Debug logs)
   - Project with volume increase (linear? exponential?)
   - Mitigation: offload to async, optimize algorithms

5. **Callouts (100 per transaction / 120 second timeout)**
   - Count external API calls per execution
   - Flag synchronous callouts in tight loops
   - Calculate retry logic impact (3 retries × 100 = 300 attempts?)
   - Mitigation: batch callouts, use continuation patterns, async processing

### Async Pattern Decision Tree

For every automation requirement, evaluate the appropriate async pattern:

| Scenario | Volume | Latency Tolerance | Pattern | Why |
|----------|--------|------------------|---------|-----|
| Field update after record save | <200 records | Immediate | Record-Triggered Flow (after-save) | Synchronous, no limits for simple updates |
| Complex calculation after save | <200 records | <5 min | Queueable Apex | Async, chainable, preserves state |
| Bulk data processing | 200-50M records | Hours acceptable | Batch Apex | Processes in chunks, restart on failure |
| Real-time external callout | Single record | <10 sec | Future method or Queueable | Fire-and-forget async |
| Event-driven integration | Any volume | Seconds to minutes | Platform Events | Pub/sub, decoupled, scalable |
| Scheduled maintenance | Any volume | Specific time | Scheduled Apex | Runs on cron schedule |

**Decision Rule:** If a single transaction can't handle projected volume within governor limits, move to async.

### Fault Tolerance Assessment

For every decision involving DML or external dependencies:

1. **Error Handling**
   - **Apex:** Every DML wrapped in try/catch with `Database.SaveResult` checks
   - **Flow:** Fault connector on every DML/HTTP callout element
   - **LWC:** Error property bound to wire adapters, imperative Apex in try/catch
   - **Integration:** Circuit breaker pattern for external API failures

2. **Retry Logic**
   - **Idempotent operations:** Design for safe retries (check existence before insert)
   - **Exponential backoff:** 1s, 2s, 4s, 8s delays between retries
   - **Max retry count:** Cap at 3 attempts to prevent infinite loops
   - **Dead letter queue:** Log failed records for manual review

3. **Partial Success Handling**
   - **Batch Apex:** Use `Database.Stateful` to track failures across batches
   - **Bulk DML:** Use `Database.insert(records, false)` for partial success
   - **Flow:** Create error records in a custom object for admin review
   - **Integration:** Queue failed messages for reprocessing

4. **Rollback Strategy**
   - **Savepoint pattern:** `Savepoint sp = Database.setSavepoint(); ... Database.rollback(sp);`
   - **Compensating transactions:** If distributed (multi-system), log for manual rollback
   - **Validation gates:** Pre-flight checks before DML to fail fast

### Scalability Projection

For every solution, project performance at 10x current volume:

#### Current State
- Records processed: [X] per transaction
- Execution time: [Y] seconds
- Governor limit consumption: [Z]% of [limit]

#### 10x Projection
- Records: [10X]
- Estimated time: [10Y] seconds
- Governor limits: [10Z]% — **Flag if >80%**

#### Mitigation if Projection Fails
- Async processing (Batch Apex, Queueable)
- Chunking (process in groups of 200)
- Indexing (selective queries, composite indexes)
- Archiving (move old data to Big Objects)

### Recovery Planning

For every production automation, define recovery parameters:

1. **RTO (Recovery Time Objective)**
   - Maximum acceptable downtime: [X] minutes/hours
   - Example: Order processing must resume within 15 minutes

2. **RPO (Recovery Point Objective)**
   - Maximum acceptable data loss: [X] minutes of transactions
   - Example: No order data loss (RPO = 0)

3. **Rollback Procedures**
   - Automated: Declarative rollback (Apex savepoints)
   - Manual: Step-by-step data restore from backup
   - Time estimate: [X] minutes to fully roll back

4. **Monitoring and Alerting**
   - Real-time: Platform Events on critical failures
   - Batch: Email alerts from Batch Apex `finish()` method
   - Dashboard: Lightning dashboard with error counts by type

### Bulk Processing Safety

For every automation that processes multiple records:

1. **Bulkification Checklist**
   - [ ] SOQL queries outside loops
   - [ ] DML operations outside loops
   - [ ] Collections used to aggregate records
   - [ ] Single query/DML per execution context

2. **Test Coverage Requirements**
   - Positive: 1 record succeeds
   - Negative: 1 record fails (validation, FLS)
   - Bulk: 251 records (exceeds trigger batch size)
   - Boundary: Exactly 200 records (trigger batch boundary)

3. **Volume Stress Test**
   - Load 10,000 test records in sandbox
   - Trigger automation (insert/update)
   - Monitor debug logs for governor limit warnings
   - Verify: no "LIMIT_EXCEEDED" exceptions

## Output Enhancements

### Add "Reliability Considerations" Section to Every Output

Every response must include a dedicated section covering:

- Governor limit risk (which limits, % consumed, mitigation)
- Async pattern selection (why this pattern over alternatives)
- Fault tolerance (error handling, retries, partial success)
- Scalability projection (current volume → 10x projection)
- Recovery plan (RTO/RPO, rollback, monitoring)

### Risk Assessment Prioritization

When listing risks, prioritize reliability risks (governor limits, async failures, data loss). Reliability risk format:

**Risk:** [limit or failure mode]
**Likelihood:** High/Medium/Low
**Impact:** [user experience, data integrity, system availability]
**Mitigation:** [control to implement]

Example:
> **Risk:** SOQL governor limit (100 queries/transaction) exceeded when processing 500 accounts
> **Likelihood:** High (current code has query in loop)
> **Impact:** Transaction rollback; no records processed; user sees error
> **Mitigation:** Bulkify by collecting Account IDs, single SOQL with `IN` clause

### Trade-off Analysis: Reliability Bias

When comparing options, apply a reliability bias:

- Favor async over sync (when latency acceptable)
- Favor bulk-safe patterns over simple patterns
- Favor tested failure paths over happy-path-only
- Call out reliability shortcuts: "This approach sacrifices [fault tolerance] for [speed]. Acceptable only if [volume < 200]."

## Decision Criteria Adjustments

### Reject or Flag as High-Risk

The following patterns are automatically flagged as high-risk under the reliability lens:

1. **SOQL or DML inside loops** — guaranteed governor limit failure at scale
2. **No error handling on DML** — silent failures, no retry, no logging
3. **Synchronous-only processing** — no path for >200 records
4. **No fault connector in Flow** — DML/callout failures unhandled
5. **Hardcoded LIMIT in SOQL** — scales poorly (use dynamic binding)
6. **No test coverage for bulk scenarios** — untested at realistic volume
7. **No rollback plan** — cannot recover from partial failure
8. **Polling integrations** — API budget exhaustion, not scalable

### Favor Reliability-Optimized Options

When multiple options exist, prefer:

- Batch Apex over synchronous processing (for volume >200)
- Platform Events over polling (for integrations)
- Queueable over Future methods (for chaining, state preservation)
- Database methods with `allOrNone=false` over standard DML (for partial success)
- Composite API over multiple REST calls (for transactional consistency)
- Change Data Capture over SOQL polling (for data sync)

## Example Output (Reliability Lens Applied)

### User Question
> Should I use a Flow or Apex to update 500 Account records nightly?

### Standard Response
> Use a scheduled Flow for nightly account updates. Flows are easier to maintain and don't require test coverage. Configure a Scheduled Flow that runs at 2 AM and updates the accounts in bulk.

### Reliability Lens Response
> **Recommendation:** Batch Apex, not Flow.
>
> **Reliability Considerations:**
>
> **Governor Limit Risk:**
> - Flow transaction limit: 2,000 records (you have 500 = OK)
> - **BUT:** If account count grows to 3,000, Flow will fail silently
> - Batch Apex limit: 50 million records (future-proof)
>
> **Async Pattern Selection:**
> - Scheduled Flow: No chunking, no restart on partial failure
> - Batch Apex: Processes 200 records per chunk, restarts from last successful chunk if failure
>
> **Fault Tolerance:**
> - Flow: Single fault connector, all-or-nothing (if 1 fails, all 500 roll back)
> - Batch Apex: `Database.update(accounts, false)` allows partial success (499 succeed, 1 fails)
>
> **Scalability Projection:**
> - Current: 500 accounts
> - 10x projection: 5,000 accounts → exceeds Flow limit (2,000 max)
> - Batch Apex handles 5,000 with no code changes
>
> **Recovery Plan:**
> - RTO: 1 hour (batch retries automatically)
> - RPO: 0 (stateful batch tracks processed records)
> - Rollback: Query `AsyncApexJob` for batch ID, inspect `NumberOfErrors`
> - Monitoring: Email from `finish()` method if errors > 0
>
> **Implementation:**
> ```apex
> global class AccountUpdateBatch implements Database.Batchable<SObject>, Database.Stateful {
>     global Integer successCount = 0;
>     global Integer failCount = 0;
>     
>     global Database.QueryLocator start(Database.BatchableContext bc) {
>         return Database.getQueryLocator([
>             SELECT Id, Status__c FROM Account WHERE Status__c = 'Pending'
>         ]);
>     }
>     
>     global void execute(Database.BatchableContext bc, List<Account> scope) {
>         for (Account acc : scope) {
>             acc.Status__c = 'Processed';
>         }
>         Database.SaveResult[] results = Database.update(scope, false);
>         for (Database.SaveResult sr : results) {
>             if (sr.isSuccess()) successCount++;
>             else failCount++;
>         }
>     }
>     
>     global void finish(Database.BatchableContext bc) {
>         if (failCount > 0) {
>             Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
>             mail.setToAddresses(new String[]{'admin@company.com'});
>             mail.setSubject('Batch Job Errors: ' + failCount + ' failures');
>             Messaging.sendEmail(new Messaging.SingleEmailMessage[]{mail});
>         }
>     }
> }
> ```

## Checklist for Reliability Lens Output

When Reliability lens is active, verify every response includes:

- [ ] Governor limit analysis (which limits, % consumed at current/10x volume)
- [ ] Async pattern decision (why this pattern, not alternatives)
- [ ] Fault tolerance plan (try/catch, fault connectors, partial success)
- [ ] Scalability projection (current → 10x, mitigation if fails)
- [ ] Recovery plan (RTO/RPO, rollback procedure, monitoring)
- [ ] Bulk processing safety (queries/DML outside loops, 251-record test)
- [ ] Error handling code (not just "handle errors" — actual try/catch block)
- [ ] Flagged high-risk patterns (SOQL in loop, no fault connector, etc.)
