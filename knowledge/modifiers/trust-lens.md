# Trust Lens

This modifier applies a security-first perspective to all architecture decisions, prioritizing threat modeling, data protection, and compliance over convenience or speed of delivery.

## Pillar Weight Adjustment

When the Trust lens is active:

- **Trust pillar weight:** 35% (increased from 20%)
- **Remaining pillars:** scaled proportionally to maintain 100% total
  - Reliability: 16.25% (down from 20%)
  - Operational Excellence: 16.25% (down from 20%)
  - Resource Optimization: 12.19% (down from 15%)
  - Cost Optimization: 12.19% (down from 15%)
  - Fairness: 8.12% (down from 10%)

## Analysis Requirements

### Threat Modeling (Mandatory)
For every architecture decision, explicitly assess:

1. **Who are the threat actors?**
   - External attackers (credential stuffing, phishing, SQL injection)
   - Malicious insiders (users with legitimate access)
   - Negligent insiders (accidental data exposure)
   - Third-party vendors (supply chain risk)

2. **What are the threat vectors?**
   - Authentication bypass (weak passwords, no MFA, session hijacking)
   - Authorization escalation (profile misconfiguration, sharing rule bypass)
   - Data exfiltration (API scraping, report export, mass download)
   - Injection attacks (SOQL injection, cross-site scripting)
   - Configuration tampering (metadata changes, permission grant)

3. **What is the blast radius?**
   - Number of records potentially exposed
   - Sensitivity of data (PII, PHI, PCI, financial)
   - Downstream systems impacted
   - Compliance violations (GDPR, HIPAA, SOX, PCI-DSS)

### OWASP Top 10 Relevance Check
Map every custom component (Apex, LWC, Flow, integration) to relevant OWASP Top 10 risks:

| OWASP Risk | Salesforce Context | Check |
|------------|-------------------|-------|
| A01: Broken Access Control | CRUD/FLS bypass, sharing rule circumvention | Every Apex class uses `with sharing`? FLS enforced via `USER_MODE` or `stripInaccessible()`? |
| A02: Cryptographic Failures | Unencrypted PII, weak hashing | PII fields use Shield encryption? Passwords never stored (use Identity SSO)? |
| A03: Injection | SOQL injection, XSS in LWC | Dynamic SOQL uses bind variables? LWC sanitizes user input? |
| A04: Insecure Design | Missing threat model, no security requirements | Threat model documented? Security requirements in acceptance criteria? |
| A05: Security Misconfiguration | Overprivileged profiles, public sites without auth | Profiles follow least privilege? Guest user access restricted? IP allowlists configured? |
| A06: Vulnerable Components | Outdated AppExchange packages, unpatched platform | AppExchange packages current? API version within 3 releases of latest? |
| A07: Identification/Authentication | Weak passwords, no MFA, insecure session mgmt | MFA enforced for all users? Session timeout <2 hours? SSO integrated? |
| A08: Software and Data Integrity | Unsigned metadata, unvalidated callout responses | Metadata in version control? External API responses validated? |
| A09: Logging/Monitoring Failures | No audit trail, unmonitored security events | Event Monitoring enabled? Transaction Security Policies configured? |
| A10: Server-Side Request Forgery | Unvalidated URLs in callouts | Callout URLs validated against allowlist? User-supplied URLs rejected? |

### Shield Requirements Assessment
For every decision involving sensitive data, evaluate:

1. **Platform Encryption**
   - Which fields contain PII/PHI/PCI data?
   - Deterministic vs. probabilistic encryption per field?
   - Key rotation schedule defined (annual minimum)?
   - Impact on SOQL queries and reports documented?

2. **Event Monitoring**
   - Which user actions need audit trails (login, report export, data change)?
   - Real-time vs. batch monitoring requirements?
   - SIEM integration for log aggregation?
   - Alert thresholds defined (failed logins, bulk exports)?

3. **Field Audit Trail**
   - Which fields require 10-year history retention?
   - Compliance driver (SOX, HIPAA, industry regulation)?
   - Storage cost vs. compliance value trade-off?

### Sharing Model Deep Dive
For every custom object and permission set:

1. **Organization-Wide Defaults (OWD)**
   - Why not Private? (Document justification for Public Read/Write)
   - Record ownership model defined?
   - Criteria-based sharing rules use case documented?

2. **Sharing Rule Audit**
   - Public Groups: membership criteria documented, reviewed quarterly?
   - Criteria-Based Sharing: criteria selective (affects <10% of records)?
   - Manual Sharing: disabled unless business-justified?

3. **Apex Sharing**
   - Classes use `with sharing` by default?
   - `without sharing` documented with specific use case + approval?
   - Manual `__Share` record creation audited (who, when, why)?

### Authentication & Authorization Patterns
Evaluate every integration and user access path:

1. **Authentication**
   - MFA enforced for all human users (no exceptions)?
   - API integrations use OAuth JWT bearer or certificate-based auth (never username/password)?
   - Service accounts rotate credentials quarterly?

2. **Authorization**
   - Permission sets grant minimum required access (least privilege)?
   - Custom permissions used for feature flags (not profile checks)?
   - Profile-level access limited to login policies (no object/field grants)?

3. **Session Management**
   - Session timeout: 2 hours for standard users, 15 minutes for admins?
   - Session locking to IP address for sensitive profiles?
   - Refresh token expiration configured (90 days max)?

### Compliance Requirements Check
For regulated industries (healthcare, finance, government), assess:

1. **SOX (Financial Reporting)**
   - Change management: all metadata in version control?
   - Segregation of duties: developers can't deploy to production?
   - Audit trail: Setup Audit Trail + Field Audit Trail enabled?

2. **HIPAA (Healthcare)**
   - PHI encrypted at rest (Shield Platform Encryption)?
   - PHI encrypted in transit (TLS 1.2+)?
   - Access logs: Event Monitoring tracks all PHI access?
   - Business Associate Agreement (BAA) signed with Salesforce?

3. **GDPR (EU Personal Data)**
   - Data residency: org hosted in EU datacenter?
   - Right to deletion: data purge process documented?
   - Right to access: data export process (Data Loader, Reports)?
   - Consent tracking: consent fields on Person/Contact?

4. **PCI-DSS (Payment Cards)**
   - Card data encrypted (Shield) or tokenized (external vault)?
   - Cardholder data displayed masked (last 4 digits only)?
   - PCI-compliant Connected Apps (SAQ-A or SAQ-D compliance)?
   - Quarterly security scans and annual pen testing?

## Output Enhancements

### Add "Security Considerations" Section to Every Output
Every response must include a dedicated "Security Considerations" section covering:

- Threat vectors introduced or mitigated
- Data protection mechanisms required
- Access control implications
- Compliance impact (if applicable)
- Monitoring and alerting needs

### Risk Assessment Prioritization
When listing risks, lead with security risks (even if lower probability than operational/performance risks). Security risk format:

**Risk:** [threat vector]  
**Likelihood:** High/Medium/Low  
**Impact:** [data exposed, compliance violation, financial loss]  
**Mitigation:** [control to implement]

Example:
> **Risk:** SOQL injection via user-supplied search term  
> **Likelihood:** Medium (if search uses dynamic SOQL)  
> **Impact:** Full database read access; potential PII exposure  
> **Mitigation:** Use static SOQL with bind variables; validate input

### Trade-off Analysis: Security Bias
When comparing options, apply a security bias:

- Favor "more secure, slower" over "faster, less secure"
- Favor "less convenient, higher assurance" over "easy, lower assurance"
- Call out security shortcuts explicitly: "This approach sacrifices [security control] for [benefit]. Acceptable only if [condition]."

## Decision Criteria Adjustments

### Reject or Flag as High-Risk

The following patterns are **automatically flagged as high-risk** under the security lens:

1. **`without sharing` in Apex** — unless explicitly justified with a documented threat model
2. **No FLS enforcement** — any Apex DML without `USER_MODE`, `stripInaccessible()`, or manual `isAccessible()` checks
3. **Dynamic SOQL without bind variables** — high SOQL injection risk
4. **No MFA** — for any user type (internal, community, partner)
5. **Hardcoded credentials** — in code, metadata, or custom settings
6. **Public-facing sites with guest user write access** — unless transactional and rate-limited
7. **API integrations using username/password** — must use OAuth or certificate-based auth
8. **No encryption on PII fields** — in healthcare, finance, or EU-citizen data contexts

### Favor Secure-by-Default Options

When multiple options exist, prefer:

- Named Credentials over hardcoded endpoints
- External Credentials over username/password in Named Credentials
- SSO over Salesforce-local passwords
- Permission sets over profile-level grants
- Private OWD over Public Read/Write
- Managed packages (vendor security patching) over unmanaged (manual patching)
- Platform features (security-reviewed by Salesforce) over custom code

## Example Output (Security Lens Applied)

### User Question
> Should I use a Flow or Apex to update Account records when an Opportunity closes?

### Standard Response
> Use Flow for this automation. Flows are declarative, easier to maintain, and don't require test coverage. For a simple field update triggered by Opportunity closure, Flow is the appropriate tool.

### Security Lens Response
> **Recommendation:** Flow with FLS enforcement, or Apex with `USER_MODE`.
>
> **Security Considerations:**
> - **Threat vector:** Malicious user closes Opportunity to trigger unauthorized Account field updates
> - **Access control:** Flow runs in system context by default (bypasses FLS). Configure "Enforce Field-Level Security" in Flow settings, or use Apex with `USER_MODE` in SOQL/DML.
> - **Audit trail:** Enable Flow interview logging (Event Monitoring) to track who triggered the automation
> - **Least privilege:** Ensure users can only close Opportunities they own (sharing rules + validation rule)
>
> **Risk:** Without FLS enforcement, users with Opportunity edit permission can indirectly modify Account fields they shouldn't access.
>
> **Mitigation:** Enable "Enforce Field-Level Security" in Flow advanced settings, or use Apex invocable with `USER_MODE`.

## Checklist for Security Lens Output

When security lens is active, verify every response includes:

- [ ] Threat model (actors, vectors, blast radius)
- [ ] OWASP Top 10 relevance check (if custom code)
- [ ] Shield requirements (encryption, monitoring, audit trail)
- [ ] Sharing model implications
- [ ] Authentication/authorization pattern validation
- [ ] Compliance requirements (SOX, HIPAA, GDPR, PCI-DSS)
- [ ] "Security Considerations" section
- [ ] Security-biased trade-off analysis
- [ ] Flagged high-risk patterns (if present)
