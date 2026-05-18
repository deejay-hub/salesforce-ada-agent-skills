# Cost Optimization Lens

This modifier applies a cost-first perspective to all architecture decisions, prioritizing total cost of ownership (TCO), licensing efficiency, and operational cost reduction.

## Pillar Weight Adjustment

When the Cost Optimization lens is active:

- **Cost Optimization pillar weight:** 35% (increased from 15%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Trust: 15.29% (down from 20%)
  - Reliability: 15.29% (down from 20%)
  - Operational Excellence: 15.29% (down from 20%)
  - Resource Optimization: 11.47% (down from 15%)
  - Fairness: 7.65% (down from 10%)

## Analysis Requirements

### Lead with TCO Comparison (Mandatory)

Every architecture decision must start with a total cost of ownership comparison table:

| Option | Licensing Cost | Development Effort | Operational Overhead | Maintenance Burden | 5-Year TCO |
|--------|---------------|-------------------|---------------------|-------------------|-----------|
| Option A | [cost] | [hours × rate] | [hours/year × rate] | [% of dev cost] | [sum] |
| Option B | [cost] | [hours × rate] | [hours/year × rate] | [% of dev cost] | [sum] |

**Cost categories to include:**

1. **Licensing Costs**
   - User licenses (Sales Cloud, Service Cloud, Platform, Community)
   - Feature licenses (Shield, CPQ, Marketing Cloud, etc.)
   - AppExchange subscription fees
   - Storage overages ($10-25/GB/month)
   - API call overages (if applicable)

2. **Development Effort**
   - Initial build time (developer hours × $100-200/hour)
   - Testing time (QA hours × $75-150/hour)
   - Deployment time (DevOps hours × $100-150/hour)
   - Documentation time (tech writer hours × $75-125/hour)

3. **Operational Overhead**
   - Admin time for ongoing support (hours/month × rate)
   - Integration maintenance (hours/year × rate)
   - User training (hours/user × rate)
   - Incident response (estimated hours/year × rate)

4. **Maintenance Burden**
   - Annual maintenance cost (typically 20% of development cost)
   - Upgrade effort (hours per major release × 3 releases/year)
   - Technical debt service (hours/quarter × rate)
   - License true-up costs (projected growth × license cost)

### AppExchange vs. Custom Build Cost Analysis

For every custom build proposal, compare against AppExchange alternatives:

#### AppExchange Option
- **License cost:** $X/user/month (base) + $Y/user/month (add-ons)
- **Implementation cost:** 40-80 hours (configuration + training)
- **Vendor stability:** [founded year, customer count, reviews, Salesforce ISV status]
- **Customization limits:** [what can't be changed?]
- **Exit cost:** [data export effort, migration path]
- **5-year TCO:** License cost × 60 months + implementation cost

#### Custom Build Option
- **Development cost:** 200-2000 hours (depending on complexity)
- **Ongoing maintenance:** 20% of dev cost annually
- **No vendor lock-in:** Full control, but full maintenance burden
- **100% customizable:** Can meet any requirement, but at a cost
- **5-year TCO:** Dev cost + (maintenance cost × 5 years)

**Decision rule:** Choose AppExchange if it meets >80% of requirements and 5-year TCO is <2x custom build. Choose custom build only for strategic differentiators.

### Flag Unnecessary Premium Features

Identify and flag features that incur premium costs without delivering proportional value:

1. **Shield Platform Encryption** ($40/user/month)
   - Required for: PII in healthcare/finance, compliance mandates (HIPAA, PCI-DSS)
   - **Flag if:** No PII, no compliance requirement, security team doesn't need it

2. **Shield Event Monitoring** ($40/user/month)
   - Required for: Security forensics, compliance audit trails (SOX, GDPR)
   - **Flag if:** Assigned org-wide (200 users) but only security team (5 users) uses it
   - **Cost saving:** Limit to 5 users → save $93,600/year

3. **Salesforce CPQ** ($75-150/user/month)
   - Required for: Complex pricing rules, guided selling, quote generation
   - **Flag if:** Simple product catalog (<50 products), no discounting, no bundles
   - **Alternative:** Standard Price Books + Opportunity Products (free)

4. **Marketing Cloud Engagement** ($400-1250/user/month)
   - Required for: Multi-channel campaigns, journey builder, AI personalization
   - **Flag if:** Only sending monthly newsletters (<10K contacts)
   - **Alternative:** Pardot ($1250/month flat) or Mailchimp ($300/month)

5. **Tableau CRM** ($75-100/user/month)
   - Required for: Predictive analytics, Einstein Discovery, embedded analytics
   - **Flag if:** Standard Reports & Dashboards meet requirements
   - **Alternative:** Lightning Reports (free) + Excel exports

### Edition Downgrade Opportunities

Evaluate whether the current Salesforce edition is right-sized:

| Feature | Professional | Enterprise | Unlimited | Performance |
|---------|-------------|------------|-----------|-------------|
| Custom Objects | 10 | Unlimited | Unlimited | Unlimited |
| API Access | ✅ | ✅ | ✅ | ✅ |
| Workflow Rules | ✅ | ✅ | ✅ | ✅ |
| Approval Processes | ✅ | ✅ | ✅ | ✅ |
| Sandboxes | 1 Dev | 1 Dev, 1 Partial | 1 Full, 5 Dev, 2 Partial | 1 Full, Unlimited Dev |
| Premier Support | ❌ | Add-on | ✅ | ✅ |
| Config Services | ❌ | ❌ | ✅ | ✅ |
| **Cost per user** | $75 | $150 | $300 | $500 |

**Downgrade scenarios:**
- Unlimited → Enterprise if you don't need 24/7 support or Configuration Services (save $150/user/month)
- Enterprise → Professional if you have <10 custom objects and don't need Partial sandboxes (save $75/user/month)

**Upgrade scenarios (when cost-justified):**
- Professional → Enterprise if you need >10 custom objects (blocker) or advanced automation (flows)
- Enterprise → Unlimited if downtime costs exceed $10K/hour (24/7 support justifies $150/user/month)

### API Call Budget Impact

For every integration, calculate API call consumption and cost impact:

1. **Measure current consumption**
   - Use Event Monitoring to track API calls per integration user
   - Calculate: calls per day × 365 days

2. **Compare against entitlement**
   - Base allocation: 15,000 calls per user per 24 hours
   - Additional allocation: 1,000 calls per user per 24 hours per additional license

3. **Identify optimization opportunities**
   - Replace polling (API call every 5 minutes = 288 calls/day) with streaming (Platform Events, Change Data Capture)
   - Batch operations (Bulk API 2.0: 1 call for 10,000 records vs. 10,000 calls for REST API)
   - Cache responses (reduce redundant queries by 50-80%)

**Example:**
> Current: Integration polls every 5 minutes (288 calls/day) to check for new cases.
>
> **Cost impact:** Consumes 1.9% of daily API budget for a single integration. With 50 integrations, this pattern would consume 96% of API budget.
>
> **Optimization:** Use Platform Events to push case creation events. Cost: 0 API calls (events are not counted).
>
> **Savings:** 288 API calls/day × 365 days = 105,120 calls/year per integration. At scale, prevents need for additional API capacity.

### Implementation Cost Ranges

Provide estimated implementation cost ranges for all options:

- **Declarative automation (Flow):** 4-16 hours ($400-3,200)
- **Apex class (simple):** 8-24 hours ($800-4,800)
- **Apex class (complex):** 24-80 hours ($2,400-16,000)
- **LWC component (simple):** 16-40 hours ($1,600-8,000)
- **LWC component (complex):** 40-120 hours ($4,000-24,000)
- **Integration (REST API):** 16-40 hours ($1,600-8,000)
- **Integration (MuleSoft):** 40-160 hours ($4,000-32,000)
- **Custom object + fields:** 4-16 hours ($400-3,200)
- **Permission set:** 2-8 hours ($200-1,600)
- **Report/dashboard:** 2-8 hours ($200-1,600)
- **AppExchange implementation:** 40-80 hours ($4,000-16,000)

Use these ranges to compare build vs. buy TCO.

## Output Requirements

### Lead with TCO Summary

Every response must start with a cost summary:

> **5-Year TCO:**
> - Option A: $X (license) + $Y (dev) + $Z (ops) = **$Total**
> - Option B: $X (license) + $Y (dev) + $Z (ops) = **$Total**
>
> **Recommendation:** [lower-cost option unless higher cost is justified by strategic value]

### Break Down Cost Components

For the recommended option, provide a detailed cost breakdown:

1. **Licensing Costs** (monthly/annual)
   - User licenses: [count] × $[rate] = $[total]
   - Feature licenses: [count] × $[rate] = $[total]
   - AppExchange: [count] × $[rate] = $[total]
   - Storage: [GB] × $[rate] = $[total]

2. **Development Effort** (one-time)
   - Design: [hours] × $[rate] = $[total]
   - Build: [hours] × $[rate] = $[total]
   - Test: [hours] × $[rate] = $[total]
   - Deploy: [hours] × $[rate] = $[total]
   - Document: [hours] × $[rate] = $[total]

3. **Operational Overhead** (annual)
   - Admin support: [hours/month] × 12 × $[rate] = $[total]
   - Training: [hours/user] × [user count] × $[rate] = $[total]
   - Incident response: [estimated hours] × $[rate] = $[total]

4. **Maintenance Burden** (annual)
   - Ongoing maintenance: 20% of dev cost = $[total]
   - Upgrade effort: [hours/release] × 3 releases × $[rate] = $[total]
   - Technical debt: [hours/quarter] × 4 × $[rate] = $[total]

### Trade-off Analysis: Cost Bias

When comparing options, explicitly call out cost vs. non-cost trade-offs:

> **Cost-optimized choice:** Option A ($50K TCO)  
> **Performance-optimized choice:** Option B ($120K TCO, 2x faster)
>
> **Cost penalty for performance:** $70K (140% increase)  
> **Justification required:** Performance improvement must deliver >$70K in business value (e.g., reduced downtime, faster sales cycles)

## Decision Criteria Adjustments

### Favor Low-Cost Options

When multiple options meet requirements, prefer:

1. **Declarative over programmatic** (Flow over Apex) — no test coverage cost, faster to build
2. **Platform features over AppExchange** (standard objects over managed packages) — no license fees
3. **AppExchange over custom build** (when meets >80% requirements) — lower TCO in most cases
4. **Self-service over support escalation** (knowledge base, chatbot) — reduce admin load
5. **Scheduled batch over real-time** (when latency acceptable) — lower compute cost
6. **Archive to Big Objects over active storage** (for historical data) — 1/10th storage cost
7. **Community licenses over internal licenses** (for external users) — $5-15 vs. $75-150 per user

### Flag Cost Inefficiencies

Automatically flag these patterns:

1. **Underutilized licenses** — Users with <1 login per month
2. **Over-provisioned licenses** — Sales Cloud users who only view reports (Platform Viewer is sufficient)
3. **Unused feature licenses** — Shield/CPQ/MC assigned but no usage in 90 days
4. **Inefficient API usage** — Polling integrations (suggest streaming)
5. **Full sandboxes for development** — Dev/DevPro are sufficient; Full is for UAT only
6. **AppExchange overlap** — Multiple packages with overlapping features
7. **Custom builds with AppExchange alternatives** — Reinventing the wheel

### Cost-Justification Thresholds

For costs exceeding these thresholds, require explicit business justification:

- **Licensing:** >$10K/year (e.g., 10 Enterprise users, or 5 Unlimited, or 1 add-on)
- **Custom development:** >40 hours ($4K)
- **AppExchange subscription:** >$5K/year
- **Infrastructure (Heroku, MuleSoft):** >$1K/month
- **Professional services:** >$20K

Justification must include: business value delivered, cost of inaction, ROI calculation.

## Example Output (Cost-Optimized Applied)

### User Question
> Should I build a custom LWC for account search, or use the standard Salesforce lookup?

### Standard Response
> Use the standard lookup component. It's maintained by Salesforce, handles large data volumes efficiently, and integrates with Salesforce search features out of the box. Build a custom LWC only if the standard lookup doesn't meet specific UI/UX requirements.

### Cost-Optimized Response
> **5-Year TCO:**
> - Standard lookup: $0 (platform feature)
> - Custom LWC: $8,000 (build) + $8,000 (maintenance over 5 years) = **$16,000**
>
> **Recommendation:** Standard lookup.
>
> **Cost breakdown (custom LWC option):**
> - Development: 40 hours × $200/hour = $8,000
>   - Design: 4 hours
>   - Build: 24 hours
>   - Test: 8 hours (Jest tests required)
>   - Documentation: 4 hours
> - Maintenance: $1,600/year × 5 years = $8,000 (20% of dev cost)
>   - 3 releases/year × 2 hours/release = 6 hours/year
>   - Bug fixes: 2 hours/year
>
> **Cost justification required to build custom:**
> The custom LWC must deliver >$16K in business value vs. the standard lookup. Example justifications:
> - Custom UI saves 5 seconds per search × 100 searches/day/user × 50 users × 250 days × $50/hour ÷ 3600 seconds = $17K/year in time savings
> - Custom filtering prevents 20 incorrect account selections/month × $500 avg correction cost = $120K over 5 years
>
> **Unless you have one of these justifications, use the standard lookup.**

## Checklist for Cost-Optimized Output

When cost-optimized modifier is active, verify every response includes:

- [ ] TCO comparison table (licensing + dev + ops + maintenance)
- [ ] Cost breakdown by category
- [ ] AppExchange vs. custom build analysis (when applicable)
- [ ] Flagged unnecessary premium features
- [ ] Edition downgrade opportunities (when applicable)
- [ ] API call budget impact (for integrations)
- [ ] Implementation cost ranges
- [ ] Cost-biased trade-off analysis
- [ ] Cost-justification threshold check
- [ ] Recommendation leads with lowest-cost option (unless justified)
