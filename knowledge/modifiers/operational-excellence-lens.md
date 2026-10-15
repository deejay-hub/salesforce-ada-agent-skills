# Operational Excellence Lens

This modifier applies an operational-first perspective to all architecture decisions, prioritizing maintainability, observability, automation, deployment velocity, and team capability alignment.

## Pillar Weight Adjustment

When the Operational Excellence lens is active:

- **Operational Excellence pillar weight:** 35% (increased from 20%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Trust: 20.31% (down from 25%)
  - Reliability: 20.31% (down from 25%)
  - Resource and Cost Optimization: 16.25% (down from 20%)
  - Fairness: 8.13% (down from 10%)

## Framework Alignment

This lens tracks the Operational Excellence pillar of the Salesforce Well-Architected Framework. [Source: https://architect.salesforce.com/docs/architect/well-architected/guide/operational-excellence.html]

### Guiding Principles

1. Evolve with observability.
2. Standardize operational procedures.
3. Embrace a DevOps culture.
4. Automate for efficiency.
5. Learn from all operational events.

### Operational Areas

| Area | What the lens examines |
|------|------------------------|
| Observability and Monitoring | Logs, metrics, traces; SLIs, burn rate, error budgets; alert levels and content |
| DevOps Practices | Source-driven development, code review, test pyramid, CI/CD, safe deployment, configuration management |
| Automation and Efficiency | Declarative-first (Flow Builder), programmatic (trigger frameworks, Batch/Queueable/Scheduled Apex), Platform Events |
| Incident Management | Detection speed, severity tiers, on-call and escalation, blameless postmortems |
| Continuous Improvement | DORA metrics, operational reviews, Operational Readiness Reviews, learning culture |

### Maturity Phases

Rate the team's current phase per area, and do not recommend controls that are two phases ahead of where the team is:

| Phase | Characteristics |
|-------|-----------------|
| Lean Optimized | Small team, minimal viable process, focus on the highest-risk gaps |
| Scale Optimized | Repeatable pipelines, automated testing, shared monitoring |
| Governance Optimized | Center of Excellence, formal change control, audited operations |

### Agentic Systems

When the decision involves Agentforce or other agents, apply `knowledge/modifiers/agentic-enterprise-lens.md` as well. Agents need the Agent Development Lifecycle, behavioral testing in Agentforce Testing Center, and observation of reasoning steps with Agentforce Session Tracing, in addition to the controls below. [Source: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-operational-excellence.html]

## Analysis Requirements

### Deployment Complexity Analysis (Mandatory)

For every architecture decision, assess deployment characteristics:

1. **CI/CD Readiness**
   - All metadata in version control (Git)
   - Automated validation pipeline (GitHub Actions, Jenkins)
   - Test automation coverage (Apex 85%+, Jest 80%+, illustrative targets; the platform minimum for Apex deploys is 75%)
   - Test pyramid: many unit tests, fewer integration tests, few end-to-end tests, plus performance tests for high-volume paths
   - Code review: two approvals, PRs of 200-400 lines, mechanical checks automated (Salesforce Code Analyzer). Review for correctness, maintainability, performance, security, and consistency.
   - Dependency ordering documented (deploy schema before code)

2. **Rollback Capability**
   - Core metadata deployments are all-or-nothing transactions. Blue-green and canary deployment apply to Heroku and MuleSoft CloudHub 2.0, not core metadata. For core, use feature flags (Custom Metadata or Custom Permissions) to release progressively.
   - Non-destructive: Deploy previous commit (< 5 minutes)
   - Destructive: Restore from backup + manual fixes (< 30 minutes)
   - Database changes: Can schema be reverted without data loss?
   - Third-party integrations: Can credentials be rotated quickly?

3. **Environment Parity**
   - Dev sandbox matches production schema (weekly refresh)
   - UAT sandbox has production data subset (monthly refresh)
   - Pre-prod full sandbox for final validation (quarterly refresh)
   - Drift detection: Unmanaged metadata flagged in pre-deploy scan; full configuration drift check at least quarterly
   - Infrastructure as code: Scratch org definitions, sandbox definitions, package definitions, and seed data in source control

4. **Deployment Metrics (DORA)**
   - Throughput:
     - Deployment frequency: How often can you safely deploy? (illustrative target: daily for non-breaking changes)
     - Lead time for changes: Commit to production (illustrative target: < 1 day for hotfix, < 1 week for feature)
   - Instability:
     - Change failure rate: % of deploys causing a failure in production (illustrative target: < 5%)
     - Failed deployment recovery time (FDRT): Time to recover from a failed deployment (illustrative target: < 2 hours)
     - Deployment Rework Rate: Share of deployments that are unplanned fixes to earlier deployments

### Observability Requirements

For every automation, integration, or custom component:

1. **Logging Standards**
   - **Apex:** `System.debug()` at INFO, WARN, ERROR levels
   - **Flow:** Screen component for errors, Platform Event for audit trail
   - **LWC:** `console.error()` captured by browser monitoring (Splunk, Datadog)
   - **Integration:** Log every request/response to custom object or external SIEM

2. **Debug Log Instrumentation**
   - Entry/exit logging: "Entering AccountService.getAccounts()", "Exiting with X records"
   - Variable state: Log critical variables before/after operations
   - Exception context: Log full stack trace + user context (UserId, profile)
   - Performance markers: `System.currentTimeMillis()` at method start/end

3. **Monitoring Dashboards**
   - **Real-time:** Lightning dashboard with error counts, API usage, batch job status
   - **Historical:** Event Monitoring queries for trends (failed logins, slow queries)
   - **Alerting:** Email/Slack on threshold breach (illustrative: >10 errors/hour, API usage >70%)

4. **SLIs and Error Budgets**
   - Define SLIs for availability, latency, success rate, and throughput on critical user processes
   - Instrument critical processes with milestone markers, and monitor integration health in both directions
   - Track error budget burn rate against the SLO; alert on burn rate, not single errors
   - Use percentiles (p95, p99) for latency, never averages

5. **Alerting Levels and Content**
   - **Critical alerts:** Production down, data loss, security breach (page on-call immediately)
   - **Warning alerts:** Batch job failure, integration degradation, error budget burning fast (respond within business hours or per runbook)
   - **Informational alerts:** Deprecation warnings, unused licenses, technical debt (weekly report)
   - Every alert must answer: what is wrong, why it matters, and what to do. Alerts without an owner cause alert fatigue.

6. **Platform Tools**
   - Event Monitoring, Proactive Monitoring, Scale Center, Setup Audit Trail, Health Check
   - Shield add-ons: Data Detect, Field Audit Trail

### Maintainability Assessment

For every custom component, evaluate long-term maintainability:

1. **Code Complexity Metrics**
   - **Cyclomatic complexity:** <10 per method (Salesforce Code Analyzer)
   - **Method length:** <50 lines per method
   - **Class length:** <500 lines per class
   - **Nested depth:** <3 levels of indentation

2. **Documentation Standards**
   - **Apex:** JavaDoc on all public methods (purpose, params, return, exceptions)
   - **Flow:** Description on every element (decision criteria, action purpose)
   - **LWC:** JSDoc on public properties/methods
   - **README:** Setup instructions, architecture diagram, troubleshooting guide

3. **Naming Conventions**
   - **Classes:** `PascalCase`, descriptive (`AccountService`, not `AcctSvc`)
   - **Methods:** `camelCase`, verb-noun (`getAccounts`, not `fetch`)
   - **Variables:** `camelCase`, descriptive (`accountList`, not `al`)
   - **Consistency:** Enforce via linter (ESLint for LWC, PMD for Apex)

4. **Coupling Assessment**
   - **Tight coupling (high-risk):** Class A directly instantiates Class B
   - **Loose coupling (preferred):** Dependency injection, interfaces, event-driven
   - **Test:** Can you change Class B without breaking Class A tests?

### Automation Opportunities

Identify manual processes that should be automated:

1. **Manual Deployment Steps** → CI/CD Pipeline
   - "Copy metadata from Dev to UAT" → `git push` triggers auto-deploy
   - "Run tests manually" → Pre-merge validation in GitHub Actions
   - "Assign permission sets" → Metadata-driven assignment (PermissionSetGroup)

2. **Manual Testing** → Automated Test Suites
   - "Click through UI to verify" → Selenium or Salesforce CLI tests
   - "Check data quality" → Validation rules + scheduled Apex tests
   - "Smoke test after deploy" → Post-deploy script with assertions

3. **Manual Monitoring** → Alerting + Dashboards
   - "Check batch job status daily" → Email on failure from `finish()` method
   - "Review debug logs for errors" → Event Monitoring query + Slack alert
   - "Monitor API usage" → Dashboard with threshold alerts

4. **Manual Data Operations** → Scheduled Batch Jobs
   - "Archive old records monthly" → Scheduled Apex batch
   - "Export reports weekly" → Scheduled Flow + email attachment
   - "Sync external data daily" → Platform Events or REST API callout

### Change Velocity Impact

For every decision, assess impact on release cadence:

1. **Blocking Changes**
   - Requires org downtime (avoid; prefer zero-downtime deploys)
   - Requires manual data migration (minimize; automate with scripts)
   - Requires third-party coordination (bundle with release; test in advance)

2. **Non-Blocking Changes**
   - Additive only (new fields, classes, flows)
   - Backward compatible (old code still works)
   - Feature-flagged (deploy off, enable later)

3. **Release Frequency Impact**
   - **High complexity:** Slows releases (weeks of testing)
   - **Low complexity:** Fast-tracks releases (hours to prod)
   - **Goal:** Favor low-complexity patterns when features are equivalent

### Incident Management Readiness

For every production change, confirm:

1. **Severity tiers:** Severity 1, 2, and 3 are defined with response expectations
2. **On-call and escalation:** Rotation, escalation paths (time, complexity, and severity triggers), and on-call access to tooling
3. **Blameless postmortems:** Timeline, root cause, impact, prevention, and detection improvement. Prevention items must be specific ("add alert on X"), not vague ("improve monitoring").
4. **Operational Readiness Review (ORR) before go-live:** Monitoring and alerting, documentation, deployment and rollback, performance and scalability, security and compliance, dependencies

### Technical Debt Scoring

For every shortcut or workaround, quantify the debt:

1. **Interest Rate (Maintenance Burden)**
   - How many hours/month does this cost to maintain?
   - Example: Hardcoded list of record types = 2 hours/month when new type added

2. **Principal (Fix Effort)**
   - How many hours to refactor to best-practice pattern?
   - Example: Refactor to query record types = 4 hours one-time

3. **Payoff Period**
   - Principal ÷ Interest Rate = months to break even
   - Example: 4 hours ÷ 2 hours/month = 2 months

4. **Decision Rule**
   - Payoff < 6 months → Prioritize refactor
   - Payoff > 12 months → Accept debt (document in README)

## Output Enhancements

### Add "Operational Considerations" Section to Every Output

Every response must include a dedicated section covering:

- Maturity phase (Lean, Scale, or Governance Optimized) and whether the option fits it
- Deployment complexity (CI/CD readiness, rollback, environment parity)
- Observability (SLIs, error budgets, logging, monitoring, alerting)
- Incident readiness (severity tiers, escalation, ORR)
- Maintainability (team skill fit, documentation, coupling)
- Automation opportunities (what manual steps remain?)
- Change velocity impact (does this slow or accelerate releases?)
- Technical debt (is this incurring interest? when to pay it down?)

### Risk Assessment Prioritization

When listing risks, prioritize operational risks (deployment failures, maintainability nightmares, manual toil). Operational risk format:

**Risk:** [operational failure mode]
**Likelihood:** High/Medium/Low
**Impact:** [deployment velocity, maintenance burden, incident frequency]
**Mitigation:** [control to implement]

Example:
> **Risk:** No rollback plan for Apex class changes
> **Likelihood:** Medium (complex logic, high change frequency)
> **Impact:** 30-minute MTTR if deploy breaks production
> **Mitigation:** Version control + CI/CD pipeline with one-click rollback

### Trade-off Analysis: Operational Bias

When comparing options, apply an operational bias:

- Favor declarative over code (when features are equivalent)
- Favor automated over manual (even if automation takes 2x upfront time)
- Favor observable patterns over black-box (add logging even if slower)
- Call out operational shortcuts: "This approach sacrifices [maintainability] for [speed]. Acceptable only if [one-time use]."

## Decision Criteria Adjustments

### Reject or Flag as High-Risk

The following patterns are automatically flagged as high-risk under the operational lens:

1. **No test coverage** — cannot validate changes, slows releases
2. **No monitoring/logging** — incidents invisible, high MTTR
3. **Manual deployment steps** — error-prone, slow, not auditable
4. **Tribal knowledge dependencies** — single points of failure (document or automate)
5. **Tightly coupled components** — changes cascade, slow velocity
6. **No version control** — cannot rollback, no audit trail
7. **Hardcoded values** — requires code change for config, slows releases
8. **No documentation** — onboarding slow, maintenance burden high
9. **"One big trigger"** — multiple triggers or all logic in a trigger body instead of a trigger framework with handler classes
10. **Process Builder or Workflow Rules for new automation** — Process Builder is past end of support (Dec 31, 2025); migrate to Flow
11. **Assuming blue-green or canary for core metadata** — core deploys are all-or-nothing; use feature flags
12. **Retrofitted observability** — monitoring planned after go-live instead of built into the first version
13. **Single-reviewer approval or huge PRs** — review quality collapses
14. **Silent batch failures** — Batch Apex with no failure reporting from `finish()`

### Favor Operationally-Optimized Options

When multiple options exist, prefer:

- Metadata-driven over hardcoded (custom metadata, custom settings)
- Declarative over code (Flow over Apex when features equivalent)
- Managed packages over custom build (vendor handles maintenance)
- Feature flags over branches (deploy off, enable in prod without redeploy)
- Trigger frameworks over logic in trigger bodies
- Idempotent Batch Apex over jobs that cannot be safely rerun
- Versioned Platform Event schemas over ad hoc payload changes
- Configuration as code over manual setup (all in Git)
- Self-service over admin intervention (user-facing tools, Knowledge articles)

## Example Output (Operational Excellence Lens Applied)

### User Question
> Should I build account deduplication in Apex or use Salesforce duplicate rules?

### Standard Response
> Use Salesforce duplicate rules. They're a standard feature, easier to configure than custom Apex, and maintained by Salesforce. Create a matching rule on Account Name and Website, then a duplicate rule that blocks or alerts on creation.

### Operational Excellence Lens Response
> **Recommendation:** Salesforce duplicate rules.
>
> **Operational Considerations:**
>
> **Deployment Complexity:**
> - Duplicate rules: Metadata XML, version-controlled, CI/CD-ready
> - Custom Apex: Requires test class (85% coverage), trigger framework setup
> - Rollback: Duplicate rule can be deactivated in 30 seconds; Apex requires redeploy
>
> **Observability:**
> - Duplicate rules: Built-in reporting (Setup > Duplicate Management > Duplicate Reports)
> - Custom Apex: Must build custom logging (debug logs, custom object for audit)
> - Monitoring: Duplicate rule errors visible in Setup Audit Trail; Apex requires Platform Events
>
> **Maintainability:**
> - Team skill: Admins can edit duplicate rules; Apex requires developer
> - Documentation: Duplicate rules self-documenting in UI; Apex needs README
> - Coupling: Duplicate rules standalone; Apex trigger couples to trigger framework
>
> **Automation Opportunities:**
> - Duplicate rules: No manual steps (deploy and activate)
> - Custom Apex: Manual test execution, manual monitoring setup
>
> **Change Velocity:**
> - Duplicate rules: Non-blocking change (deploy in minutes, no downtime)
> - Custom Apex: Blocking (requires test run, 10-20 min deploy time)
>
> **Technical Debt:**
> - Duplicate rules: Zero debt (Salesforce maintains)
> - Custom Apex: 20% annual maintenance (rule changes, API upgrades, test updates)
>
> **When to Use Custom Apex:**
> Only if duplicate rules don't meet requirements:
> - Cross-object matching (Contact duplicates trigger Account merge)
> - Complex matching logic (fuzzy + phonetic + nickname matching)
> - Custom UX (inline merge UI, not standard alert)

## Checklist for Operational Excellence Lens Output

When Operational Excellence lens is active, verify every response includes:

- [ ] Maturity phase identified and recommendation sized to it
- [ ] Deployment complexity analysis (CI/CD readiness, rollback via feature flags, parity, drift)
- [ ] Observability plan (SLIs, error budget burn rate, alert levels and content)
- [ ] Incident readiness (severity tiers, escalation, blameless postmortem, ORR)
- [ ] DORA metrics impact (throughput and instability)
- [ ] Maintainability assessment (team skill fit, documentation, coupling)
- [ ] Automation opportunities (what manual steps can be eliminated?)
- [ ] Change velocity impact (blocking vs. non-blocking, release frequency)
- [ ] Technical debt score (interest rate, principal, payoff period)
- [ ] Flagged high-risk patterns (no tests, no monitoring, tribal knowledge)
- [ ] Operationally-biased trade-off (favor declarative, metadata-driven, observable)
