# Trust Lens

This modifier applies a security-first perspective to all architecture decisions, prioritizing threat modeling, identity, data protection, privacy, and compliance over convenience or speed of delivery.

## Framework Alignment

Aligned with the Salesforce Well-Architected **Trust** pillar: "Protect people, data, and systems through secure, compliant design."

- Pillar: https://architect.salesforce.com/docs/architect/well-architected/guide/trust.html
- Agentic variant: https://architect.salesforce.com/docs/architect/well-architected/guide/agentic-enterprise-trust.html

**Foundation: the Shared Responsibility Model.** Salesforce secures the infrastructure and platform. The customer secures what they build on it: configuration, access, code, and data governance. Incident response, vulnerability management, monitoring, compliance certifications, identity federation, and key management are shared. Every Trust assessment must state which side of the line each control sits on.

**Trust dimensions:**
1. "Security controls protect systems and data"
2. "Identity management governs access"
3. "Privacy practices respect user agency"
4. "Compliance frameworks satisfy regulatory obligations"

**Design principles:**
- "Apply zero trust across all layers."
- "Grant least privilege by default."
- "Implement defense in depth."
- "Integrate security by design."
- "Embed security in automation."
- "Design for privacy."
- "Design for traceability."
- "Design for incident response."

**Topic areas** (use these as the review checklist structure): Shared Responsibility, Security Architecture, Identity and Access Management, Data Protection and Privacy, Compliance and Regulatory Adherence, Secure Development Lifecycle, Security Incident Response.

## Pillar Weight Adjustment

When the Trust lens is active:

- **Trust pillar weight:** 35% (increased from 25%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Reliability: 21.67% (down from 25%)
  - Operational Excellence: 17.33% (down from 20%)
  - Resource and Cost Optimization: 17.33% (down from 20%)
  - Fairness: 8.67% (down from 10%)

## Analysis Requirements

### Threat Modeling (Mandatory)
For every architecture decision, explicitly assess the following. Structure the threat analysis with STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege).

1. **Who are the threat actors?**
   - External attackers (credential stuffing, phishing, injection)
   - Malicious insiders (users with legitimate access)
   - Negligent insiders (accidental data exposure)
   - Third-party vendors and supply chain (AppExchange/AgentExchange packages, external AI services)

2. **What are the threat vectors?**
   - Authentication bypass (weak passwords, no MFA, session hijacking, stolen OAuth refresh tokens)
   - Authorization escalation (profile misconfiguration, sharing rule bypass, elevated Integration or Agent User context)
   - Data exfiltration (API scraping, report export, mass download)
   - Injection attacks (SOQL injection, cross-site scripting, prompt injection for agentic solutions)
   - Configuration tampering (metadata changes, permission grant)

3. **What is the blast radius?**
   - Number of records potentially exposed
   - Data classification of exposed data (see Data Classification below)
   - Downstream systems impacted
   - Compliance violations (GDPR, HIPAA, SOX, PCI-DSS)

4. **Defense in depth** — name the control at each layer: Data, Application, Identity, Integration. A single-layer control is a finding.

### Data Classification (Mandatory for any decision touching data)
Classify affected data with the four tiers the framework uses, then derive controls from the tier:

| Tier | Example | Minimum expectation |
|------|---------|---------------------|
| Public | Published product catalog | Integrity controls |
| Internal | Internal pipeline reports | Least-privilege access, sharing model |
| Confidential | Customer PII, contracts | FLS, private OWD, audit trail, consider Shield Platform Encryption |
| Restricted | PHI, PCI, regulated financial data | Encryption (Shield Platform Encryption, consider BYOK or Cache-Only Key Service), Event Monitoring, documented residency |

Apply privacy by design: data minimization, purpose limitation, consent, and support for data subject rights. Data residency must be decided per jurisdiction — do not rely on hardcoded figures.

### OWASP Top 10 (2025) Relevance Check
Map every custom component (Apex, LWC, Flow, integration, agent action) to the relevant OWASP Top 10:2025 risks. The framework calls out A01, A02, and A05 as the most relevant to Salesforce.

| OWASP 2025 Risk | Salesforce Context | Check |
|-----------------|-------------------|-------|
| A01: Broken Access Control | CRUD/FLS bypass, sharing rule circumvention, system-mode Apex | Apex declares `with sharing` or `inherited sharing`? Queries and DML run in user mode (`WITH USER_MODE`, `AccessLevel.USER_MODE`) or use `Security.stripInaccessible()`? |
| A02: Security Misconfiguration | Overprivileged profiles, permissive OWDs, guest user access | Health Check score 80% or higher? Private OWDs? Guest access restricted? |
| A03: Software Supply Chain Failures | Unvetted AppExchange/AgentExchange packages, external prompt templates | Packages vetted and provenance recorded? |
| A04: Cryptographic Failures | Unencrypted Restricted data, weak key management | Shield Platform Encryption for Restricted fields? Key rotation defined? |
| A05: Injection | SOQL injection, XSS in LWC, prompt injection | Dynamic SOQL uses bind variables? LWC avoids unsafe DOM manipulation? Instructions and data separated in prompts? |
| A06: Insecure Design | Missing threat model, no security requirements | STRIDE threat model documented? Security requirements in acceptance criteria? |
| A07: Authentication Failures | No MFA, Username-Password Flow, insecure session management | MFA enforced? Approved OAuth flow per scenario? |
| A08: Software or Data Integrity Failures | Unversioned metadata, unvalidated callout responses | Metadata in version control with CI/CD security gates? External responses validated? |
| A09: Security Logging and Alerting Failures | No audit trail, unmonitored security events | Event Monitoring enabled and routed to SIEM? Transaction Security policies configured? |
| A10: Mishandling of Exceptional Conditions | Error paths that leak data or fail open | Exceptions fail closed? Error messages avoid exposing internals? |

Note: `WITH SECURITY_ENFORCED` was removed in API version 67.0. Never recommend it; use user mode database operations or `Security.stripInaccessible()`.

### Shield Requirements Assessment
For every decision involving Confidential or Restricted data, evaluate:

1. **Platform Encryption**
   - Which fields hold Confidential/Restricted data?
   - Probabilistic or Deterministic encryption scheme per field?
   - Customer-controlled keys needed (BYOK, Cache-Only Key Service)?
   - Key rotation schedule defined?
   - Impact on SOQL queries and reports documented?

2. **Event Monitoring**
   - Which user actions need audit trails (login, report export, data change)?
   - Real-time vs. batch monitoring requirements?
   - SIEM integration for log aggregation?
   - Transaction Security policies for high-risk actions?

3. **Field Audit Trail**
   - Which fields require long-term history retention?
   - Compliance driver (SOX, HIPAA, industry regulation)?
   - Storage cost vs. compliance value trade-off?

4. **Sandbox data protection**
   - Is production data masked or minimized before it reaches sandboxes?

### Sharing Model Deep Dive
For every custom object and permission set, use the four-layer access model (organization, object, field, record):

1. **Organization-Wide Defaults (OWD)**
   - Default to Private. Permissive OWDs are "The most common trust debt in mature orgs" — any Public Read/Write requires a documented justification.
   - Record ownership model defined?
   - Restriction rules or muting permissions needed to narrow access further?

2. **Sharing Rule Audit**
   - Public Groups: membership criteria documented and reviewed on a fixed cadence?
   - Criteria-based sharing: criteria scoped narrowly?
   - Manual sharing: disabled unless business-justified?

3. **Apex Sharing**
   - Every class declares sharing (`with sharing` or `inherited sharing`)? Undeclared-sharing legacy classes are a finding.
   - `without sharing` documented with a specific use case and approval?
   - Manual `__Share` record creation audited (who, when, why)?

### Authentication and Authorization Patterns
Evaluate every integration and user access path:

1. **Authentication**
   - MFA enforced for all human users; SSO with the corporate IdP where available?
   - Integration OAuth flow matches the scenario: JWT Bearer for server-to-server; Web Server Flow with PKCE for user-facing apps; OAuth 2.0 Token Exchange when a system carries a user's context.
   - Username-Password Flow is never used.
   - One app registration per integration — no shared service accounts.
   - New integrations governed through External Client Apps, not newly created Connected Apps.

2. **Securing the trust boundary for headless and on-behalf-of clients**
   - "Propagate user identity, don't pool it."
   - "Treat OAuth client credentials and refresh tokens as the primary target."
   - "Minimize the scope on both sides."
   - "Govern the boundary through External Client Apps."

3. **Authorization**
   - Permission sets and permission set groups grant minimum required access (least privilege)?
   - Profile-level access limited to login policies?
   - Critical-impact accounts (admins, integration users) hardened?
   - Identity lifecycle automated (for example, SCIM provisioning and deprovisioning)?

4. **Secrets and certificates**
   - Named Credentials and External Credentials for every callout — no hardcoded secrets.
   - No self-signed, expired, or sprawling wildcard certificates; certificate lifecycle owned.

5. **Session Management**
   - Session timeout and session security policies defined per profile risk (values are org-specific; document the chosen policy).
   - Session locking to IP address for sensitive profiles?

### Compliance Requirements Check
For regulated industries, decide by jurisdiction and confirm against current platform certifications (compliance.salesforce.com). Continuously validate with Health Check (score 80% or higher), Event Monitoring, and Transaction Security.

1. **SOX (Financial Reporting)**
   - Change management: all metadata in version control?
   - Segregation of duties: developers can't deploy to production?
   - Audit trail strategy: Setup Audit Trail plus Field Audit Trail?

2. **HIPAA (Healthcare)**
   - PHI encrypted at rest (Shield Platform Encryption) and in transit?
   - Event Monitoring tracks PHI access?
   - Business Associate Agreement (BAA) in place with Salesforce?

3. **GDPR and other privacy laws**
   - Data residency decided per jurisdiction?
   - Right to deletion and right to access processes documented?
   - Consent captured and honored?

4. **PCI-DSS (Payment Cards)**
   - Card data tokenized (external vault) or encrypted?
   - Cardholder data displayed masked?
   - Quarterly security scans and annual penetration testing?

### Secure Development Lifecycle and Incident Response
- Shift-left: static analysis (Salesforce Code Analyzer) and security gates in CI/CD.
- Security testing before release.
- Detection and response architecture defined; a disclosure channel (RFC 9116 security.txt) for externally exposed sites.
- Post-incident reviews track MTTD and MTTR.

### Agentic Trust (Brief)
When the decision involves agents, apply these minimums and also activate `knowledge/modifiers/agentic-enterprise-lens.md` for full depth:
- Employee agents inherit the invoking user's permissions. Customer agents run as dedicated, least-privilege Agent Users — never as public-site guest users.
- Never use a system administrator as an agent's running user.
- Treat subagent and action assignment as a permission boundary. Every Apex and Flow action is checked for system-mode bypass.
- Keep instructions and data separate in prompts; treat all external and record content as untrusted (prompt injection).
- HITL approvals for irreversible, high-value, or regulated actions are enforced as Flow checkpoints, not as agent instructions.
- The Einstein Trust Layer is one layer of protection, not a full solution.

## Output Enhancements

### Add "Security Considerations" Section to Every Output
Every response must include a dedicated "Security Considerations" section covering:

- Shared responsibility split (what Salesforce secures vs. what the customer secures)
- Threat vectors introduced or mitigated (STRIDE)
- Data classification tier and the protection mechanisms it requires
- Access control implications (four-layer access model)
- Compliance impact (if applicable)
- Monitoring, alerting, and incident response needs

### Risk Assessment Prioritization
When listing risks, lead with security risks (even if lower probability than operational/performance risks). Security risk format:

**Risk:** [threat vector]
**Likelihood:** High/Medium/Low
**Impact:** [data exposed, compliance violation, financial loss]
**Mitigation:** [control to implement]

Example:
> **Risk:** SOQL injection via user-supplied search term
> **Likelihood:** Medium (if search uses dynamic SOQL)
> **Impact:** Read access beyond intended scope; potential PII exposure
> **Mitigation:** Use static SOQL or bind variables; run queries in user mode; validate input

### Trade-off Analysis: Security Bias
When comparing options, apply a security bias:

- Favor "more secure, slower" over "faster, less secure"
- Favor "less convenient, higher assurance" over "easy, lower assurance"
- Name the framework trade-off explicitly: Trust controls add cost and latency (Trust vs. Resource and Cost Optimization). State the cost and judge whether it is proportionate to the data tier.
- Call out security shortcuts explicitly: "This approach sacrifices [security control] for [benefit]. Acceptable only if [condition]."

## Decision Criteria Adjustments

### Reject or Flag as High-Risk

The following patterns are **automatically flagged as high-risk** under the Trust lens:

1. **`without sharing` or undeclared sharing in Apex** — unless explicitly justified with a documented threat model
2. **No CRUD/FLS enforcement** — Apex queries/DML not in user mode and not using `Security.stripInaccessible()`
3. **`WITH SECURITY_ENFORCED`** — removed in API 67.0; flag and replace with user mode
4. **Dynamic SOQL without bind variables** — SOQL injection risk
5. **No MFA** — for any human user type (internal, community, partner)
6. **Hardcoded credentials** — in code, metadata, or custom settings
7. **Username-Password OAuth Flow or shared service accounts** — use per-integration registrations with JWT Bearer, Web Server Flow with PKCE, or Token Exchange
8. **Permissive OWDs (Public Read/Write) without justification**
9. **Guest user access for customer agents, or guest user write access on public sites**
10. **No encryption on Restricted data** — in healthcare, finance, or regulated-privacy contexts
11. **New Connected Apps** — new integrations should use External Client Apps

### Favor Secure-by-Default Options

When multiple options exist, prefer:

- Named Credentials and External Credentials over hardcoded endpoints or secrets
- External Client Apps over new Connected Apps
- SSO over Salesforce-local passwords
- Per-user authentication and identity propagation over pooled integration identities
- Permission sets over profile-level grants
- Private OWD over Public Read/Write
- User mode database operations, Lightning Data Service, and the UI API over system-context data access
- Managed packages (vendor security patching) over unmanaged (manual patching)
- Platform features (security-reviewed by Salesforce) over custom code

## Example Output (Trust Lens Applied)

### User Question
> Should I use a Flow or Apex to update Account records when an Opportunity closes?

### Standard Response
> Use Flow for this automation. Flows are declarative and easier to maintain. For a simple field update triggered by Opportunity closure, Flow is the appropriate tool.

### Trust Lens Response
> **Recommendation:** Flow configured to respect user permissions, or Apex with user mode database operations.
>
> **Security Considerations:**
> - **Shared responsibility:** Salesforce enforces the sharing and FLS model; the customer is responsible for automation that runs in system context and bypasses it.
> - **Threat vector (Elevation of privilege):** A user closes an Opportunity to trigger Account field updates they could not make directly.
> - **Data classification:** Account financial fields are Confidential; updates must respect FLS.
> - **Access control:** Record-triggered automation can run in system context. Confirm the run context, or use an Apex invocable with `WITH USER_MODE` queries and `AccessLevel.USER_MODE` DML.
> - **Traceability:** Field history tracking (or Field Audit Trail for long retention) on the updated Account fields.
>
> **Risk:** Without permission enforcement, users with Opportunity edit permission can indirectly modify Account fields they shouldn't access.
>
> **Mitigation:** Enforce user permissions in the automation, and add field history tracking on the affected fields.

## Checklist for Trust Lens Output

When the Trust lens is active, verify every response includes:

- [ ] Shared responsibility split
- [ ] STRIDE threat model (actors, vectors, blast radius) with defense-in-depth layers named
- [ ] Data classification tier and derived controls
- [ ] OWASP Top 10:2025 relevance check (if custom code or agent actions)
- [ ] Shield requirements (encryption, monitoring, audit trail)
- [ ] Sharing model implications (four-layer access model)
- [ ] Authentication/authorization pattern validation (approved OAuth flow, External Client Apps, no shared accounts)
- [ ] Compliance requirements by jurisdiction, with Health Check target
- [ ] Agentic trust minimums (if agents are involved)
- [ ] "Security Considerations" section
- [ ] Security-biased trade-off analysis naming the Trust vs. Resource and Cost Optimization tension
- [ ] Flagged high-risk patterns (if present)
