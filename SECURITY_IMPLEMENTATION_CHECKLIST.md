# Security Implementation Checklist - Immigration Form AI

## Quick Reference: Security Decisions Made

| Decision | Recommendation | Rationale |
|----------|---|----------|
| **Encryption at Rest** | AES-256-GCM (client + server) | Defense in depth; server can't decrypt user data |
| **Encryption in Transit** | TLS 1.3 mandatory | Prevents man-in-the-middle; certificate pinning for apps |
| **Zero-Knowledge Use** | Yes, for validation proofs | Verify data without decrypting on server |
| **LLM API Pattern** | Redaction proxy (isolated service) | Never send raw PII to third-party APIs |
| **Key Management** | AWS Secrets Manager + KMS | Industry standard, automatic rotation, audit logging |
| **Database Strategy** | PostgreSQL + pgcrypto + RLS | Application-layer encryption + database-level controls |
| **Access Control** | MFA + VPN + audit logging | Zero-trust for all internal access |
| **Compliance Target** | SOC2 Type II | Meets regulatory needs (GDPR/CCPA secondary) |
| **Monitoring** | Real-time alerts + ELK stack | Detect breaches immediately |
| **Backup Strategy** | Encrypted geo-redundant daily | RTO < 1 hour, encrypted in transit & rest |

---

## Pre-Build Decisions (Before Writing Code)

### Legal & Compliance Review

- [ ] **Consult immigration law expert**
  - Confirm: Is this practicing immigration law? (likely NO if AI-assisted form filling only)
  - Verify: Can we include a liability waiver? (yes, required)
  - Risk: Clients suing if form rejected or incorrect

- [ ] **Privacy/Data Protection Review**
  - Consult: Privacy lawyer or compliance firm
  - Confirm GDPR applicability if serving EU users
  - Confirm CCPA applicability if serving California users
  - Cost: $2-5K (one-time)

- [ ] **HIPAA Review** (if handling medical exam records)
  - Immigration medical exams may trigger HIPAA requirements
  - If yes: Implement BAA, audit controls, minimum necessary principle
  - Cost: $5-10K (implementation + ongoing)

- [ ] **Insurance**
  - Professional liability ($1-2M coverage): $3-5K/year
  - Cyber liability ($2M coverage): $2-4K/year
  - E&O insurance (errors & omissions)

### Threat Modeling Session

- [ ] **Assemble threat model team**
  - Security engineer (or external contractor)
  - Product manager (understand use cases)
  - Backend engineer (understand data flow)
  - Time: 4-6 hours

- [ ] **Document attack vectors**
  - Data exfiltration by admin/employee
  - LLM API compromise (Claude/OpenAI breach)
  - Database breach (ransomware, SQL injection)
  - User device compromise (malware on client)
  - Third-party vendor compromise (hosting provider, email)

- [ ] **Risk rating & mitigation**
  - High-risk items: require mitigation before launch
  - Medium-risk: mitigate in Phase 2
  - Low-risk: accept or monitor

---

## Phase 1: Foundation (Weeks 1-4)

### 1.1 Infrastructure & Cloud Setup

- [ ] **AWS Account Setup**
  - [ ] Enable CloudTrail (audit logging)
  - [ ] Enable AWS Config (compliance monitoring)
  - [ ] Set up AWS Secrets Manager
  - [ ] Create KMS master key for encryption
  - [ ] Enable bucket encryption (S3)
  - [ ] Enable RDS encryption (database)
  - Estimate cost: $200-500/month (development tier)

- [ ] **VPC & Network Security**
  - [ ] Create VPC with public/private subnets
  - [ ] RDS in private subnet (no direct internet)
  - [ ] Application servers in private subnet
  - [ ] NAT Gateway for outbound traffic
  - [ ] Security groups: restrict by port & protocol
  - [ ] NACLs: additional layer of filtering

- [ ] **DNS & HTTPS**
  - [ ] Register domain (e.g., immigrationforms.app)
  - [ ] Enable DNSSEC
  - [ ] Setup Let's Encrypt with auto-renewal
  - [ ] HSTS header configuration (max-age=63072000)
  - [ ] Test HTTPS with SSL Labs (target: A+ grade)

- [ ] **CDN & WAF**
  - [ ] CloudFlare account (free tier minimum)
  - [ ] WAF rules (OWASP Top 10)
  - [ ] DDoS protection enabled
  - [ ] Rate limiting (100 requests/minute per IP)
  - [ ] Geographic restrictions (block high-risk countries if needed)

### 1.2 Backend Security Framework

- [ ] **Authentication & Authorization**
  - [ ] Implement OAuth2 (Google/Microsoft login)
  - [ ] Hash passwords with bcrypt (cost: 12)
  - [ ] JWT tokens for API access
  - [ ] Token expiry: 15 minutes access + 7-day refresh
  - [ ] Logout: invalidate refresh token on server
  - [ ] Session timeout: 30 minutes of inactivity

- [ ] **Secrets Management**
  - [ ] Create Secrets Manager secrets for:
    - [ ] Database password
    - [ ] OpenAI API key
    - [ ] Claude API key (if used)
    - [ ] Master encryption key
    - [ ] LLM proxy API key (internal)
  - [ ] Set up automatic rotation (quarterly)
  - [ ] Create Lambda function to handle rotation
  - [ ] Test secret rotation (quarterly)

- [ ] **Database Security**
  - [ ] PostgreSQL 14+ installed
  - [ ] Enable pgcrypto extension
  - [ ] Create roles:
    - [ ] admin (full access, MFA required)
    - [ ] app_user (limited to API schema)
    - [ ] read_only (for reporting, no write)
  - [ ] Row-level security:
    - [ ] Users see only own form data
    - [ ] Admins see anonymized reports
  - [ ] Enable SSL connections only (sslmode=require)

- [ ] **API Security Headers**
  - [ ] `Content-Security-Policy: default-src 'self'`
  - [ ] `X-Frame-Options: DENY`
  - [ ] `X-Content-Type-Options: nosniff`
  - [ ] `Strict-Transport-Security: max-age=63072000`
  - [ ] `Referrer-Policy: strict-origin-when-cross-origin`

### 1.3 Encryption Implementation

- [ ] **Client-Side Encryption**
  - [ ] Integrate libsodium.js or TweetNaCl.js
  - [ ] Key derivation (PBKDF2-SHA256, 600K iterations)
  - [ ] AES-256-GCM for encryption
  - [ ] Tests: encrypt/decrypt roundtrip
  - [ ] Tests: performance on slow devices (<500ms)
  - [ ] No plaintext PII in localStorage
  - [ ] Encrypted IndexedDB for draft forms

- [ ] **Server-Side Encryption**
  - [ ] Application-layer encryption wrapper
  - [ ] Master key in Secrets Manager
  - [ ] Encrypt before storing in database
  - [ ] Decrypt only when necessary (not in logs)
  - [ ] Tests: encryption/decryption correctness
  - [ ] Tests: key rotation without data loss

- [ ] **Data in Transit**
  - [ ] All endpoints HTTPS-only (no HTTP)
  - [ ] API endpoints verify TLS certificate
  - [ ] Reject weak ciphers (test with testssl.sh)
  - [ ] Certificate pinning for mobile apps
  - [ ] CORS: restrict to own domain only

### 1.4 Logging & Monitoring (Safe Logging)

- [ ] **Structured Logging Setup**
  - [ ] JSON log format (not plain text)
  - [ ] Include: timestamp, level, service, request_id, user_id_hash, action, result
  - [ ] EXCLUDE: SSN, passwords, plaintext PII, API keys
  - [ ] Log to CloudWatch + ELK stack (backup)
  - [ ] Retention: 7 years (encrypted)

- [ ] **Alerts & Monitoring**
  - [ ] Alert on failed decryption attempts (>5/min)
  - [ ] Alert on unauthorized database access
  - [ ] Alert on API key usage anomaly
  - [ ] Alert on SSL certificate expiry (<30 days)
  - [ ] Dashboard: request latency, error rate, encryption performance
  - [ ] PagerDuty integration for critical alerts

- [ ] **Backup & Recovery**
  - [ ] Daily automated backups (RDS automated backups)
  - [ ] Point-in-time recovery (35 days)
  - [ ] Cross-region backup replication
  - [ ] Test restore (monthly)
  - [ ] RTO < 1 hour, RPO < 15 minutes

### 1.5 LLM API Isolation

- [ ] **Redaction Service Setup**
  - [ ] Create separate microservice (AWS Lambda or ECS)
  - [ ] Input: encrypted form data (from main service)
  - [ ] Redaction rules for all Tier 1 data
  - [ ] Output: redacted JSON (no PII)
  - [ ] Logging: redacted data only (no plaintext)
  - [ ] Tests: verify SSN/passport never in logs

- [ ] **API Key Management**
  - [ ] Store OpenAI/Claude keys in Secrets Manager
  - [ ] Create separate API key per environment (dev/staging/prod)
  - [ ] Track usage per endpoint (cost monitoring)
  - [ ] Set usage limits (quota on keys if supported)
  - [ ] Rotate keys quarterly
  - [ ] Disable unused keys immediately

- [ ] **LLM Request/Response Handling**
  - [ ] Request: send only redacted fields
  - [ ] Response: extract insights (risk level, inconsistencies)
  - [ ] Never store raw LLM response if it contains inferred PII
  - [ ] Hash the response for deduplication (without logging)
  - [ ] Tests: LLM output doesn't contain reconstructed PII

### 1.6 Security Testing

- [ ] **Unit Tests (Encryption)**
  - [ ] Test AES-256-GCM with test vectors
  - [ ] Test PBKDF2 key derivation
  - [ ] Test PII redaction rules
  - [ ] Test ZK proof generation/verification

- [ ] **Integration Tests**
  - [ ] End-to-end form submission (encrypted)
  - [ ] Verify server can't decrypt without key
  - [ ] Verify database stores ciphertext
  - [ ] Verify API response is encrypted

- [ ] **Manual Testing**
  - [ ] Test HTTPS with curl: `curl -vI https://app.com`
  - [ ] Test SSL Labs: `https://www.ssllabs.com/ssltest/`
  - [ ] Test HSTS: check for `Strict-Transport-Security` header
  - [ ] Test CSP: check `Content-Security-Policy` header
  - [ ] Test API key exposure: scan code with truffleHog

### 1.7 Documentation

- [ ] **Security Architecture Doc** (COMPLETED - see SECURITY_ARCHITECTURE.md)

- [ ] **Operational Runbooks**
  - [ ] Key rotation procedure
  - [ ] Incident response playbook
  - [ ] Disaster recovery procedure
  - [ ] Data breach notification process

- [ ] **API Documentation**
  - [ ] Document all endpoints with security requirements
  - [ ] Rate limits per endpoint
  - [ ] Required headers (Authorization, X-Request-Signature)
  - [ ] Error codes & handling

---

## Phase 2: Hardening (Weeks 5-8)

### 2.1 Zero-Knowledge Proofs

- [ ] **Research & Library Selection**
  - [ ] Evaluate libzk (JavaScript)
  - [ ] Evaluate SEAL (C++, via WASM)
  - [ ] Decide: ZK for client validation OR just server re-validation

- [ ] **Implementation**
  - [ ] ZK commitment for SSN (9-digit range proof)
  - [ ] ZK commitment for date fields (range + format)
  - [ ] ZK consistency proof (passport expiry > today)
  - [ ] Tests: proof generation + verification
  - [ ] Performance tests (target: <100ms per proof)

- [ ] **Integration**
  - [ ] Client generates ZK proofs on form validation
  - [ ] Server verifies proofs without decryption
  - [ ] Flag inconsistencies based on failed proofs
  - [ ] Log proof verification results (safe, no PII)

### 2.2 Advanced Access Control

- [ ] **Multi-Factor Authentication (MFA)**
  - [ ] Require MFA for employee accounts
  - [ ] Support TOTP (Google Authenticator) + hardware keys
  - [ ] MFA for database access (via Teleport or Bastion)
  - [ ] MFA for AWS Console access

- [ ] **Role-Based Access Control (RBAC)**
  - [ ] Define roles: Admin, Support, Analyst, Read-Only
  - [ ] Admin: requires 2 MFA methods + manager approval
  - [ ] Support: can access user forms (encrypted view only)
  - [ ] Analyst: can access anonymized reports (no PII)
  - [ ] Implement in code + database RLS

- [ ] **VPN Requirement**
  - [ ] All database access via VPN only
  - [ ] All secrets retrieval via VPN only
  - [ ] Document: employees required to use corporate VPN
  - [ ] Test: verify direct database access blocked

### 2.3 Audit Logging

- [ ] **Comprehensive Audit Trail**
  - [ ] Log all database queries (pg_stat_statements)
  - [ ] Log all API access (request + response metadata)
  - [ ] Log all secrets access (Secrets Manager CloudTrail)
  - [ ] Log all encryption key access
  - [ ] User_id: always hash (never plaintext)

- [ ] **Audit Review Process**
  - [ ] Weekly: review failed login attempts
  - [ ] Monthly: review data access patterns
  - [ ] Quarterly: full security audit
  - [ ] Maintain audit log for 7 years

### 2.4 Rate Limiting & DDoS

- [ ] **Application-Level Rate Limiting**
  - [ ] Rate limit per IP: 100 requests/minute
  - [ ] Rate limit per user: 1000 requests/hour
  - [ ] Rate limit per API endpoint (stricter for sensitive)
  - [ ] Implement token bucket algorithm
  - [ ] Return 429 Too Many Requests

- [ ] **DDoS Protection**
  - [ ] CloudFlare DDoS protection (auto-enabled)
  - [ ] AWS Shield Standard (auto-enabled)
  - [ ] Consider AWS Shield Advanced ($3K/month)
  - [ ] Geo-blocking if needed (US/Canada/etc.)

### 2.5 Data Retention & Deletion

- [ ] **Data Retention Policy**
  - [ ] User forms: 1 year after completion or request
  - [ ] Audit logs: 7 years (legal requirement)
  - [ ] Backups: 90 days (encrypted)
  - [ ] LLM API logs: 30 days (anonymized)
  - [ ] User metadata (email, login): 1 year after account deletion

- [ ] **Automated Deletion**
  - [ ] Cron job: daily check for expired data
  - [ ] Cryptographic erasure: delete encryption key for old data
  - [ ] OR physical deletion: DELETE from database
  - [ ] Verify backup doesn't contain deleted data
  - [ ] Alert on deletion failure

- [ ] **User-Initiated Deletion**
  - [ ] One-click "Delete all my data"
  - [ ] Requires re-authentication (password + MFA)
  - [ ] Cascading delete: forms + audit trail (keep anon)
  - [ ] Confirmation email sent
  - [ ] Completion within 24 hours

### 2.6 Dependency & Vulnerability Management

- [ ] **Dependency Scanning**
  - [ ] Setup: Dependabot or Snyk
  - [ ] Scan npm/pip packages weekly
  - [ ] Auto-patch: minor & patch updates
  - [ ] Manual review: major version upgrades
  - [ ] Remove unused dependencies

- [ ] **Code Security Scanning**
  - [ ] Setup: SAST tool (SonarQube, Semgrep, CodeQL)
  - [ ] Scan on every commit
  - [ ] Block merge if critical issues found
  - [ ] Quarterly: manual code review of crypto code

### 2.7 Incident Response Plan

- [ ] **Create Incident Response Doc**
  - [ ] Define: who to notify, when, how
  - [ ] Data breach: notify users within 24 hours
  - [ ] Ransomware: isolated backup, restore within 1 hour
  - [ ] API key compromise: revoke + rotate within 15 minutes
  - [ ] Escalation path: engineer → manager → CISO

- [ ] **Test Incident Response**
  - [ ] Tabletop exercise: simulated breach
  - [ ] Quarterly: test backup restoration
  - [ ] Document: post-incident review

---

## Phase 3: Compliance (Weeks 9-12)

### 3.1 SOC2 Type II Readiness

- [ ] **SOC2 Control Implementation**
  - [ ] CC: Common Criteria (Security)
    - [ ] CC6.1: Information and Assets (inventory + classification)
    - [ ] CC6.2: Configuration Management (change controls)
    - [ ] CC7.2: System Monitoring (logging + alerting)
  - [ ] A&A: Availability & Confidentiality
    - [ ] Encryption keys backed up (multi-region)
    - [ ] Key recovery documented
    - [ ] Never log encryption keys
  - [ ] PI: Processing Integrity
    - [ ] Input validation (client + server)
    - [ ] Data integrity checks
    - [ ] Error handling & logging

- [ ] **SOC2 Documentation**
  - [ ] Risk assessment (annual)
  - [ ] Data flow diagrams (with encryption points)
  - [ ] Security incident log (template)
  - [ ] Access control matrix (roles + permissions)
  - [ ] Disaster recovery plan + test results
  - [ ] Employee security training records
  - [ ] Penetration test results (annual)

- [ ] **SOC2 Audit Preparation**
  - [ ] Select auditor (Big 4 accounting firm recommended)
  - [ ] Type II requires: minimum 6 months of control evidence
  - [ ] Start collecting evidence: logs, access records, training docs
  - [ ] Cost: $15-30K (one-time), $10-15K (annual)

### 3.2 GDPR Compliance (if serving EU)

- [ ] **Data Protection Impact Assessment (DPIA)**
  - [ ] Identify high-risk processing (yes, this is high-risk)
  - [ ] Privacy risks: data breach, unauthorized access
  - [ ] Mitigations: encryption, access controls
  - [ ] Document: approved by legal/privacy officer

- [ ] **Data Processing Agreement (DPA)**
  - [ ] Create DPA with AWS (data processor)
  - [ ] Create DPA with OpenAI/Claude (if US-based, consider data transfer)
  - [ ] Document: data categories, purpose, duration
  - [ ] Require: processor audit rights, sub-processor approval

- [ ] **Data Rights**
  - [ ] Implement: right to access (download data as JSON)
  - [ ] Implement: right to erasure (delete all data)
  - [ ] Implement: data portability (export format)
  - [ ] Implement: right to object (opt-out collection)
  - [ ] Response time: 30 days

- [ ] **GDPR Compliance Doc**
  - [ ] Privacy policy (GDPR-specific sections)
  - [ ] Legal basis: legitimate interest OR consent (decide)
  - [ ] Data retention: policy document
  - [ ] Subprocessors: list + approval mechanism

### 3.3 CCPA Compliance (if serving California)

- [ ] **Consumer Privacy Notice**
  - [ ] Categories of personal information collected
  - [ ] Purpose of collection
  - [ ] Source of information
  - [ ] Right to delete / right to know / right to opt-out

- [ ] **Consumer Rights Implementation**
  - [ ] Right to know: provide data copy within 45 days
  - [ ] Right to delete: delete (with exceptions)
  - [ ] Right to opt-out: stop selling (if applicable)
  - [ ] Do not sell: make it easy to opt-out

### 3.4 HIPAA (if handling medical records)

- [ ] **Determine HIPAA Applicability**
  - [ ] Are you collecting medical exam records? (yes/no)
  - [ ] If yes: covered entity or business associate?
  - [ ] If business associate: need BAA with healthcare clients

- [ ] **HIPAA Controls (if applicable)**
  - [ ] Access controls (minimum necessary)
  - [ ] Audit controls (logging)
  - [ ] Encryption & decryption (AES-256)
  - [ ] Integrity controls (tamper detection)
  - [ ] Transmission security (TLS 1.3)

### 3.5 Third-Party Security Assessments

- [ ] **AWS Security Assessment**
  - [ ] Review: Shared Responsibility Model
  - [ ] Verify: AWS handles infrastructure security
  - [ ] Your responsibility: application + data encryption
  - [ ] Cost: part of AWS compliance (included)

- [ ] **OpenAI/Claude Security Assessment**
  - [ ] Request: security whitepaper
  - [ ] Question: data retention (verify: no retention for free tier)
  - [ ] Question: encryption in transit
  - [ ] Question: data centers (US only or global?)
  - [ ] Document: response in security file

- [ ] **Vendor Risk Assessment**
  - [ ] Email hosting provider: encryption, access controls
  - [ ] Monitoring service (Datadog, etc.): data residency, access
  - [ ] Create matrix: vendor name, data access, security controls

### 3.6 Penetration Testing

- [ ] **Scope & Planning**
  - [ ] Define: what to test (API, web app, database, cloud infra)
  - [ ] Define: rules of engagement (no DoS attacks, etc.)
  - [ ] Duration: 2-4 weeks
  - [ ] Contractor: reputable firm (Bugcrowd, Intigriti, etc.)
  - [ ] Cost: $5-15K

- [ ] **Test Execution**
  - [ ] Black-box: attacker has no prior knowledge
  - [ ] Gray-box: attacker knows API endpoints (more realistic)
  - [ ] Focus areas: LLM API redaction, encryption, auth
  - [ ] Document: findings + risk rating

- [ ] **Remediation**
  - [ ] Critical: fix within 1 week
  - [ ] High: fix within 2 weeks
  - [ ] Medium: fix within 1 month
  - [ ] Low: fix within quarter
  - [ ] Re-test critical fixes

### 3.7 Bug Bounty Program

- [ ] **Bug Bounty Setup** (Optional, but recommended)
  - [ ] Platform: HackerOne, Bugcrowd, or Intigriti
  - [ ] Scope: all production endpoints
  - [ ] Out of scope: DoS, brute force, social engineering
  - [ ] Rewards: $50-1000 per finding
  - [ ] Process: researcher → report → fix → payment

---

## Phase 4: Ongoing Security (Post-Launch)

### 4.1 Quarterly Security Reviews

- [ ] **Q1, Q2, Q3, Q4: Security Audit**
  - [ ] Review: access logs for anomalies
  - [ ] Review: failed login attempts
  - [ ] Review: encryption key usage
  - [ ] Update: threat model based on new CVEs
  - [ ] Assess: third-party vendor security posture
  - [ ] Document: findings in audit report

### 4.2 Annual Activities

- [ ] **Annual Penetration Test**
  - [ ] Hire external firm
  - [ ] Full black-box assessment
  - [ ] Document findings + remediation

- [ ] **Annual Disaster Recovery Drill**
  - [ ] Simulate: database failure
  - [ ] Restore: from backup
  - [ ] Measure: actual RTO/RPO
  - [ ] Document: results + lessons learned

- [ ] **Annual Security Training**
  - [ ] All employees: security awareness
  - [ ] Topics: password management, phishing, social engineering
  - [ ] Track: completion + pass rate

- [ ] **Annual SOC2 Audit** (if pursuing certification)
  - [ ] Auditor reviews control evidence
  - [ ] Verify: controls operating effectively
  - [ ] Publish: SOC2 Type II report

- [ ] **Annual Privacy Impact Assessment**
  - [ ] Review: data handling practices
  - [ ] Update: DPA with AWS/third parties
  - [ ] Assess: new regulatory requirements
  - [ ] Document: update to privacy policy if needed

### 4.3 Continuous Monitoring (Year-Round)

- [ ] **Weekly**
  - [ ] Monitor: failed login attempts
  - [ ] Monitor: error rate + latency
  - [ ] Monitor: encryption key usage (should be ~0)
  - [ ] Review: any PagerDuty alerts

- [ ] **Monthly**
  - [ ] Review: access logs (check for unusual patterns)
  - [ ] Review: data deletion logs (verify compliance)
  - [ ] Update: dependency security scan results
  - [ ] Test: backup restoration (1st of month)

- [ ] **Quarterly**
  - [ ] Full security audit (access, logs, encryption)
  - [ ] Update: threat model
  - [ ] Review: third-party vendor security
  - [ ] Test: incident response plan
  - [ ] Rotate: encryption keys (automatic, verify)

---

## Critical "Do Not" List

### Never Do This

- [ ] ❌ Log plaintext SSN, passport, or PII
- [ ] ❌ Send unencrypted PII to LLM APIs
- [ ] ❌ Store encryption keys in environment variables
- [ ] ❌ Commit API keys or secrets to Git
- [ ] ❌ Use HTTP (always HTTPS)
- [ ] ❌ Trust client-side validation alone
- [ ] ❌ Store user passwords (hash only)
- [ ] ❌ Grant database access without VPN
- [ ] ❌ Skip security testing before launch
- [ ] ❌ Ignore SOC2 requirements (customers will ask)
- [ ] ❌ Assume "security through obscurity" works
- [ ] ❌ Disable CORS to "allow" access (use proper auth)
- [ ] ❌ Sync encryption keys to version control
- [ ] ❌ Test with real user data in development

---

## Cost Estimation

### One-Time Costs (Before Launch)

| Item | Cost | Notes |
|------|------|-------|
| Privacy/Compliance Legal Review | $2-5K | Essential for immigration legal questions |
| Security Architecture Review | $3-5K | Contractor or senior engineer time |
| Penetration Testing | $5-15K | External firm (wait until Phase 3) |
| Insurance Setup | $2-3K | Professional liability + cyber |
| Secrets Manager / KMS Setup | $1K | AWS costs for secure key storage |
| **Total One-Time** | **$13-33K** | |

### Monthly Recurring Costs

| Item | Cost | Notes |
|------|------|-------|
| AWS Infrastructure (compute, DB, storage) | $500-1500 | Scales with users |
| AWS Secrets Manager (KMS) | $50-100 | Encryption key management |
| CloudFlare CDN + WAF | $20-200 | Depends on tier (free→pro) |
| Datadog or CloudWatch monitoring | $100-500 | Log aggregation + alerts |
| Backups (cross-region replication) | $50-200 | Covered by AWS |
| Let's Encrypt SSL (free) + monitoring | $0-50 | Auto-renewal, optional monitoring |
| **Total Monthly** | **$700-2550** | Minimum viable; scales up with growth |

### Annual Costs

| Item | Cost | Notes |
|------|------|-------|
| SOC2 Type II Audit | $15-30K | One-time first year, $10-15K ongoing |
| Annual Pen Test | $5-15K | Recommended for compliance |
| Employee Security Training | $1-3K | Online courses + platform |
| Compliance Software (optional) | $2-5K | Vanta, Drata, etc. |
| **Total Annual** | **$23-53K** | One-time SOC2 is biggest cost |

### Total Cost to Market (Year 1)

- **Development** (4 engineers × $50K/quarter): $200K
- **Infrastructure + Security**: $13K one-time + $8.4K (12 months)
- **Compliance + Audit**: $23K (SOC2 one-time)
- **Insurance**: $5-7K
- **Other** (legal, contractor help): $5K
- **Total**: ~$254-272K for 12 months

**Break-Even Analysis:**
- Target MRR: $8-35K at 12 months (from MICRO_SAAS_IDEAS.md)
- At $20K MRR: 12-14 months to recoup development costs
- Thereafter: margin improves significantly

---

## Implementation Order (Dependency Graph)

```
Week 1-2:
├─ Legal/compliance review
├─ AWS account setup + IAM
└─ Threat modeling

Week 3-4:
├─ Backend skeleton (auth, database)
├─ Client encryption library integration
└─ Secrets Manager setup

Week 5-6:
├─ Encryption implementation (client + server)
├─ LLM redaction proxy
└─ API security headers

Week 7-8:
├─ Rate limiting + DDoS
├─ Audit logging
└─ Backup + recovery testing

Week 9-10:
├─ ZK proofs (if implementing)
├─ MFA for employees
└─ Penetration testing (external)

Week 11-12:
├─ SOC2 documentation
├─ GDPR/CCPA review
└─ Final security audit
```

---

## Success Criteria (Launch Readiness)

Before launching to production, verify:

- [ ] All Phase 1 items complete
- [ ] Encryption working end-to-end (client-server-DB)
- [ ] LLM redaction proxy operational (no PII in logs)
- [ ] Rate limiting + DDoS protection active
- [ ] Audit logging comprehensive (7-year retention)
- [ ] Backup tested (RTO < 1 hour)
- [ ] API security headers present
- [ ] SSL Labs grade A+ (or at least A)
- [ ] No API keys/secrets in codebase (tuffleHog clean)
- [ ] Privacy policy + ToS reviewed by lawyer
- [ ] Insurance policies in place
- [ ] Incident response plan documented + tested
- [ ] Team trained on security practices

**Do NOT launch without these.**

