# Fairness Lens

This modifier applies an equity-first perspective to all architecture decisions, prioritizing multi-persona coverage, accessibility, representational data quality, equitable data access, bias detection, transparency, user agency, and human oversight.

## Framework Alignment

Aligned with the Salesforce Well-Architected **Fairness** pillar: "Ensure systems are transparent, accessible, and equitable."

- Pillar: https://architect.salesforce.com/docs/architect/well-architected/guide/fairness.html
- Agentic variant: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-fairness.html

**Fairness dimensions:**
1. "Inclusive design" — accessible LWC, Visualforce, and Experience Cloud
2. "AI fairness practices" — bias detection, diverse training data, ongoing monitoring
3. "Governance" — processes and controls that sustain fairness over time

**Guiding principles:**
- "Design for Lightning accessibility patterns."
- "Reduce and monitor bias with the right tools for each model type."
- "Leverage Shield Platform Encryption for data protection controls."
- "Design review workflows for high-stakes predictions."
- "Apply Organization-Wide Defaults (OWD) and Field-Level Security (FLS) for non-discrimination."
- "Enable user agency through platform consent."

**Review areas** (use these as the checklist structure): Data Quality and Representational Fairness, Accessibility and Inclusive Design, Fairness in Salesforce Automation, AI Fairness and Bias Detection, Transparency and Auditability, Human Oversight with Salesforce Automation, Non-Discrimination with Data Access Controls, User Agency and Consent, Ethical AI Governance.

## Pillar Weight Adjustment

When the Fairness lens is active:

- **Fairness pillar weight:** 35% (increased from 10%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Trust: 18.06% (down from 25%)
  - Reliability: 18.06% (down from 25%)
  - Operational Excellence: 14.44% (down from 20%)
  - Resource and Cost Optimization: 14.44% (down from 20%)

## Analysis Requirements

### Multi-Persona Coverage Analysis (Mandatory)

For every architecture decision, assess whether all user personas are served equitably:

1. **Persona Inventory**
   - List all user types: Sales reps, service agents, managers, customers, partners, admins
   - Map features to personas: Which personas can use this feature?
   - Identify gaps: Which personas are excluded? Why?

2. **Feature Parity Assessment**
   | Persona | Can Access? | Can Use Effectively? | Gaps |
   |---------|-------------|---------------------|------|
   | Sales Rep | Yes | Yes | None |
   | Service Agent | Yes | Partial (mobile limits) | No offline mode |
   | Manager | Yes | No (missing dashboard) | Reporting not designed for role |
   | Customer (portal) | No | N/A | Feature not exposed to customers |
   | Partner (community) | No | N/A | No API access |

3. **Equity vs. Equality**
   - **Equality:** Every persona gets same feature (may not be fair if needs differ)
   - **Equity:** Each persona gets what they need (different features, same outcome)
   - **Example:** Sales reps need mobile CRM; managers need desktop analytics. Fair ≠ identical.

4. **Inclusion Criteria**
   - Can users with disabilities access this feature? (see Accessibility section)
   - Can users in different geographies use this? (language, data residency)
   - Can users with varying technical skills use this? (complexity, training)
   - Can users on different devices use this? (desktop, mobile, tablet)

### Data Quality and Representational Fairness

Data models encode assumptions about who users and customers are. For every data model change:

1. **Required fields** — "Review required fields for data availability." and "Audit business justifications for required fields." A required SSN, email, or street address excludes people who don't have one.
2. **Name fields** — Western First/Last name structures and Latin-only validation exclude single names, patronymics, and non-Latin scripts.
3. **Address validation** — Strict validation fails for rural, informal, or international addresses.
4. **Communication channels** — Single-channel designs (email only or phone only) exclude segments of the population.
5. **Demographic data and third-party enrichment** — Collect demographic data only with a documented purpose; review enrichment sources for coverage gaps across groups.

### Accessibility Assessment (WCAG 2.2 AA)

Standard SLDS components provide built-in accessibility. Custom components must explicitly meet WCAG 2.1 AA at minimum; target WCAG 2.2 AA for new work.

For every UI component, evaluate accessibility:

1. **Perceivable (Can users perceive the content?)**
   - **Text alternatives:** Images have alt text, icons have aria-label
   - **Color contrast:** 4.5:1 ratio for normal text, 3:1 for large text
   - **Audio/video:** Captions for audio, transcripts for video
   - **Adaptable:** Content works in different layouts (mobile, zoom)

2. **Operable (Can users interact with the interface?)**
   - **Keyboard navigation:** All actions accessible via keyboard (no mouse-only)
   - **Focus indicators:** Visible focus state (outline on focused element)
   - **No seizure triggers:** No flashing content >3 times/second
   - **Navigation aids:** Skip links, breadcrumbs, consistent menus

3. **Understandable (Can users understand the interface and content?)**
   - **Language:** `lang` attribute on HTML, plain language (no jargon)
   - **Predictable:** Consistent navigation, no surprise pop-ups
   - **Error prevention:** Validation with clear error messages, confirm before destructive actions
   - **Help text:** Field-level help, tooltips, inline instructions

4. **Robust (Can content be reliably interpreted by assistive tech?)**
   - **Valid HTML:** Semantic elements (<button>, not <div> with click handler)
   - **ARIA roles:** `role="navigation"`, `role="alert"`, `aria-live="polite"`
   - **Screen reader testing:** Test with NVDA (Windows) or VoiceOver (Mac)

5. **Salesforce-Specific Accessibility Checks**
   - Lightning components use SLDS (Salesforce Lightning Design System) — built-in accessibility
   - LWC: Use base components (`lightning-input`, `lightning-button`). In custom components, "Use semantic HTML in custom Lightning Web Components." and "Implement ARIA in custom components."
   - Placeholder text is never the only label
   - Keyboard: "Use logical focus order in Lightning pages.", "Use visible focus indicators to meet contrast requirements." (no `outline: none` without a replacement), "Use keyboard-trap mitigations in modals and overlays.", and "Use shortcut keys without conflicts." (no single-character global shortcuts)
   - Flow: Screen elements have labels, help text, error messages; "Use focus management in Lightning flows."
   - Experience Cloud: accessible templates, WCAG-compliant theming, keyboard-navigable menus, accessible forms, alternative text for images and icons

6. **Accessibility testing in CI/CD**
   - "Use sa11y for Lightning Web Component accessibility testing."
   - "Use Lighthouse CI for Experience Cloud."
   - "Use the Accessibility Agent, available through the Salesforce DX MCP package."
   - Automated checks are not sufficient on their own — also test with assistive technology (JAWS, NVDA, VoiceOver)

### Data Access Equity and Non-Discrimination

The framework treats data access controls as a non-discrimination mechanism: "Enable restrictive OWD as the default.", "Apply Field-Level Security for sensitive attributes.", "Design assignment and sharing rules for equitable outcomes.", and "Provide permission sets for temporary access." Audit with Shield Event Monitoring (access to sensitive data), sharing rule distribution reports, and CRUD/FLS violation reviews.

For every data model and reporting strategy:

1. **Reporting Visibility Across Roles**
   - Can every role see the data they need to do their job?
   - Example: Service agents need case metrics; currently restricted to manager role
   - Mitigation: Create role-specific dashboards, adjust sharing rules

2. **OWD (Organization-Wide Default) Fairness**
   - Private OWD: Only owner sees record (may hide data from collaborators)
   - Public Read: Everyone sees all records (may expose sensitive data)
   - **Fair default:** Private with sharing rules for team visibility

3. **Permission Set Equity**
   - Are high-value features only accessible to senior roles?
   - Example: AI-powered lead scoring only for managers; reps would benefit too
   - Mitigation: Expand access, add guardrails (e.g., read-only for reps)

4. **Field-Level Security (FLS) Balance**
   - Do FLS restrictions prevent legitimate use cases?
   - Example: Service agents can't see Account revenue; needed for upsell routing
   - Mitigation: Grant read access, keep edit restricted

### Fairness in Salesforce Automation

Deterministic automation is not automatically objective. Review:

- **Assignment and routing** — "Audit assignment rule outcomes regularly." and "Review omni-channel skill-based routing."
- **Flows** — "Review Flow decisions with fairness in mind."
- **Validation rules** — "Test validation rules using diverse data."
- **Territories** — "Analyze territory demographics before finalizing designs."
- **High-stakes automation** — "Review high-stakes automation through an ethical lens." and keep the logic transparent.

### Bias Detection in AI and Automation

For every automated decision (assignment rules, scoring models, AI predictions). Detection practices: "Audit Salesforce CRM data before training.", "Calculate fairness metrics across demographic groups.", "Capture predictive-model decision data for bias detection.", "Detect proxy features in predictive models." Mitigation practices: "Balance data before model training.", "Remove proxies in feature engineering.", "Adjust thresholds during post-processing.", "Retrain models periodically using updated data." Note that per-group threshold adjustment is prohibited in employment decisions.

1. **Input Data Bias**
   - Training data: Does historical data reflect biased human decisions?
   - Example: Lead scoring trained on past wins; if sales team focused on large companies, model learns to deprioritize small businesses
   - Mitigation: Audit training data for representation, balance dataset

2. **Algorithmic Bias**
   - Logic bias: Do rules encode unfair assumptions?
   - Example: Case routing rule: "VIP accounts get routed first" → non-VIPs wait longer
   - Mitigation: Round-robin routing, SLA-based priority (not account-based)

3. **Output Bias**
   - Outcome bias: Does the automation produce unequal results for different groups?
   - Example: Opportunity forecasting AI predicts lower close rates for female sales reps (reflects historical bias, not ability)
   - Mitigation: Test predictions by demographic group, adjust model

4. **Transparency Requirements**
   - Explainability: Can users see why an automated decision was made?
   - Example: "This lead was scored 85/100 because: industry match (30), company size (25), engagement (30)"
   - Mitigation: Add "Show Why" button that displays decision factors

5. **Proxy features**
   - Zip code, name patterns, communication timing, device type, and territory can act as proxies for protected traits
   - Mitigation: Detect and remove proxies during feature engineering

### Inclusive Design Review

For every feature, assess inclusiveness across dimensions:

1. **Language Support**
   - Is the UI available in the user's language?
   - Salesforce supports 30+ languages; are labels/help text translated?
   - Translation quality: Are translations culturally appropriate? (Not just machine-translated)

2. **Device Support**
   - Desktop only: Excludes mobile-first users (field reps, delivery drivers)
   - Mobile optimized: Responsive design, touch targets >44px, no hover-only
   - Tablet: Often overlooked; test on iPad/Android tablet

3. **Cognitive Load**
   - Complexity: Is the UI overwhelming for novice users?
   - Progressive disclosure: Show basic options first, "Advanced" section for power users
   - Defaults: Pre-fill forms with smart defaults to reduce decisions

4. **Cultural Sensitivity**
   - Imagery: Avoid cultural symbols that may offend (religious, political)
   - Names: Support non-Western name formats (single name, patronymic, accents)
   - Date/time: Respect local formats (MM/DD/YYYY vs. DD/MM/YYYY)

### Transparency Requirements

For every automated process, ensure auditability:

1. **Decision Logging**
   - What: Log every automated decision (assignment, scoring, approval)
   - Who: Log the system user or automation context
   - Why: Log decision factors (rule criteria, AI confidence score)
   - When: Timestamp with timezone

2. **User-Facing Transparency**
   - Notifications: "Your case was assigned to Agent X because [criteria]"
   - Dashboards: Show automation metrics (cases auto-closed, leads auto-scored)
   - Audit trail: Users can view history of automated actions on their records

3. **Admin Transparency**
   - Setup Audit Trail: Track changes to automation rules
   - Debug logs: Capture automation execution for troubleshooting
   - Reporting: "Automation Health" dashboard (error rates, processing times)

4. **Explainability and audit for AI**
   - "Surface Einstein Discovery explanations at decision points.", "Layer explanations for different audiences.", "Communicate confidence appropriately." (plain-language confidence, not raw probability scores)
   - "Use the Field Audit Trail for long-term retention.", "Use Event Monitoring for Agentforce interactions.", "Audit Agentforce conversations with the Einstein Trust Layer."
   - Regulatory drivers: EU AI Act transparency requirements, GDPR right to explanation, algorithmic accountability laws

### Human Oversight

For any automation or AI that makes consequential decisions (employment, credit, healthcare, legal rights, housing), require human review before execution. For confidence-scored decisions, the agentic Fairness guidance uses these bands (calibrate against production outcomes):

| Confidence | Behavior |
|------------|----------|
| High (>90%) | Proceed autonomously |
| Moderate (70-90%) | Human reviews before action |
| Low (<70%) | Defer to a human |

Name accountable roles by person, not title: model owner, decision owner, appeal authority, audit authority. Escalation must route to a queue that actually has capacity.

### User Agency and Consent

- "Enable granular consent per AI use case." — blanket AI consent is a finding
- "Use Data 360 consent for personalization." and "Enable Marketing Cloud consent integration."
- "Provide accessible opt-outs in profile settings." and "Provide persistent preference options across channels."
- Query consent (for example, in Flow) before AI affects a user; new capabilities require new consent

### Ethical AI Governance

- "Provide model documentation." and "Enable Pre-deployment fairness gates." — a fairness gate carries the same authority as security review
- Define the review board's composition, the triggers that require ethics board approval, and who has the authority to block deployments (advisory-only reviews are a finding)
- "Create CRM Analytics fairness dashboards.", "Schedule fairness audits.", and "Implement incident response for fairness failures."
- Bias detection is continuous, not a one-time pre-launch check

### Agentic Fairness (Brief)
When the decision involves agents, apply these minimums and also activate `knowledge/modifiers/agentic-enterprise-lens.md` for full depth:
- Bot disclosure is prominent and continuous, with a persistent "Connect to Human Agent" option and no service penalty for choosing a human.
- Users can contest agent decisions; an appeal authority can reverse them, supported by historical reconstruction (model version, inputs, configuration state).
- Measure fairness at each stage of a multi-agent chain, not only on the final outcome; agreement between agents does not prove the absence of shared bias.

## Output Enhancements

### Add "Fairness Considerations" Section to Every Output

Every response must include a dedicated section covering:

- Multi-persona coverage (which personas served, which excluded, why)
- Representational data quality (required fields, name and address handling, channels)
- Accessibility compliance (WCAG 2.2 AA target, 2.1 AA minimum; sa11y/Lighthouse CI; assistive tech testing)
- Data access equity and non-discrimination (OWD, sharing rules, FLS, reporting visibility)
- Bias detection (input data, algorithm, proxies, output, mitigation)
- Inclusive design (language, device, cognitive load, cultural sensitivity)
- Transparency and human oversight (decision logging, explanations, HITL thresholds, contestability)
- User agency and consent (per-use-case consent, opt-outs)

### Risk Assessment Prioritization

When listing risks, prioritize fairness risks (accessibility barriers, bias, inequitable access). Fairness risk format:

**Risk:** [fairness concern]
**Affected Personas:** [who is excluded or disadvantaged]
**Impact:** [inability to perform job, poor user experience, legal/compliance risk]
**Mitigation:** [inclusive design change to implement]

Example:
> **Risk:** Mobile users (service agents) cannot access dashboard (desktop-only)
> **Affected Personas:** 50 field service agents (20% of user base)
> **Impact:** Agents cannot see case queue on-site; must call dispatch for assignments
> **Mitigation:** Build mobile-optimized Lightning dashboard or mobile-native app

### Trade-off Analysis: Fairness Bias

When comparing options, apply a fairness bias:

- Favor inclusive patterns over exclusive (accessible to all personas)
- Favor transparent over opaque (users understand automated decisions)
- Favor equitable over equal (meet different needs, not identical features)
- Name the framework trade-off explicitly: Fairness vs. Autonomy. Fairness enforcement (HITL review) slows agent throughput; Human-in-the-Loop patterns set the balance.
- Call out fairness shortcuts: "This approach excludes [persona]. Acceptable only if [that persona does not need this feature]."

## Decision Criteria Adjustments

### Reject or Flag as High-Risk

The following patterns are automatically flagged as high-risk under the fairness lens:

1. **Solution designed for single persona** — excludes other legitimate users
2. **No accessibility testing** — may be unusable by people with disabilities
3. **Opaque automation logic** — users don't understand why decisions are made
4. **Data visibility concentrated to admins** — prevents self-service, creates bottleneck
5. **English-only interface with multi-language user base** — excludes non-English speakers
6. **Desktop-only UI in mobile-first scenario** — field workers cannot access
7. **No error messages for validation failures** — users don't know what they did wrong
8. **No alternative for mouse-only interactions** — keyboard users cannot complete task
9. **Required fields or validation that exclude valid identities** — Western-only name format, Latin-only characters, strict address validation, mandatory SSN or email
10. **Blanket AI consent** — instead of granular, per-use-case consent
11. **Consequential AI decisions without human review** — employment, credit, healthcare, legal rights, housing
12. **Raw probability scores shown to end users** — instead of plain-language confidence
13. **Advisory-only ethics review** — no authority to block deployment

### Favor Fairness-Optimized Options

When multiple options exist, prefer:

- Declarative over code (admins can modify, not just developers)
- SLDS components over custom HTML (built-in accessibility)
- Mobile-responsive over desktop-only (inclusive device support)
- Multi-language support over English-only (Translation Workbench)
- Self-service over admin intervention (empower users)
- Transparent scoring over black-box AI (explainable decisions)
- Granular, revocable consent over blanket consent
- Human review at decision points for high-stakes outcomes over full autonomy
- Universal design over persona-specific hacks (one solution serves all)

## Example Output (Fairness Lens Applied)

### User Question
> Should I build a custom dashboard for sales managers to track team performance?

### Standard Response
> Build a Lightning dashboard with components showing team revenue, pipeline, win rates, and activity metrics. Use standard reporting components and filter by manager's team using the "My Team's Accounts" filter. Deploy to a custom app for managers.

### Fairness Lens Response
> **Recommendation:** Build Lightning dashboard, but design for multi-persona access with role-based filtering.
>
> **Fairness Considerations:**
>
> **Multi-Persona Coverage:**
> - **Managers:** Need team aggregate metrics (current design serves)
> - **Reps:** Need personal metrics for self-coaching → Add "My Performance" view
> - **Executives:** Need org-wide rollup → Add "Company Performance" view
> - **Missing:** Service team has no equivalent dashboard → Design parity needed
>
> **Accessibility Compliance:**
> - [ ] Use `lightning-card` components (ARIA landmarks built-in)
> - [ ] Chart alt text: `<c-chart title="Revenue by Rep" description="Bar chart showing..."/>`
> - [ ] Keyboard navigation: Tab order logical (top to bottom, left to right)
> - [ ] Color contrast: SLDS tokens support WCAG AA contrast (no custom colors)
> - [ ] Screen reader: Test with VoiceOver and NVDA; charts must announce data points
> - [ ] CI: sa11y tests on any custom LWC used on the dashboard page
>
> **Data Access Equity:**
> - **Current:** Only managers see team data (sales reps cannot self-assess)
> - **Fair:** Reps see own performance vs. team average (privacy-preserving)
> - **Implementation:** Use dynamic filtering: `Running User` determines view scope
>
> **Bias Detection:**
> - **Risk:** Dashboard shows "Top Performers" → social comparison pressure
> - **Mitigation:** Make "leaderboard" component optional (managers can hide)
> - **Alternative:** Show improvement trends (illustrative: "You closed 15% more deals this quarter")
>
> **Inclusive Design:**
> - **Language:** Use Translation Workbench to translate labels (ES, FR, DE)
> - **Device:** Lightning dashboards are mobile-responsive by default; test on phone
> - **Cognitive load:** Use progressive disclosure (default view = 4 key metrics; "Advanced" = 12 metrics)
>
> **Transparency:**
> - **Data source:** Add footnote: "Data refreshed hourly; shows closed deals from last 30 days"
> - **Calculations:** Hover tooltip: "Win rate = Closed Won / Total Opportunities"
> - **Audit:** Setup Audit Trail logs dashboard view activity for compliance
>
> **Implementation Adjustments:**
> ```
> Dashboard filter logic:
> IF User.Profile = 'Sales Manager'
>   → Show My Team's Data
> ELSE IF User.Profile = 'Sales Rep'
>   → Show My Data
> ELSE IF User.Profile = 'Executive'
>   → Show All Data
> ```
>
> **When to Separate Dashboards:**
> If persona needs diverge significantly (e.g., sales vs. service), create separate dashboards. But prefer single dashboard with role-based filtering when possible (reduces maintenance).

## Checklist for Fairness Lens Output

When Fairness lens is active, verify every response includes:

- [ ] Multi-persona coverage analysis (which personas served, which excluded, justification)
- [ ] Representational data quality review (required fields, names, addresses, channels)
- [ ] Accessibility assessment (WCAG 2.2 AA target / 2.1 AA minimum, sa11y or Lighthouse CI, assistive tech testing)
- [ ] Data access equity and non-discrimination (OWD, sharing, FLS, reporting visibility across roles)
- [ ] Fairness in automation (assignment, routing, territory, validation rules)
- [ ] Bias detection (input data, algorithm logic, proxies, output fairness, mitigation)
- [ ] Inclusive design review (language support, device support, cognitive load, cultural sensitivity)
- [ ] Transparency and human oversight (decision logging, explanations, HITL thresholds, contestability)
- [ ] User agency and consent (per-use-case consent, opt-outs)
- [ ] Ethical AI governance (pre-deployment fairness gate, accountable roles) if AI is involved
- [ ] Agentic fairness minimums (if agents are involved)
- [ ] Flagged exclusionary patterns (single-persona design, no accessibility, opaque automation)
- [ ] Fairness-biased trade-off naming the Fairness vs. Autonomy tension
