# Security Architecture for AI Immigration Form Assistant

## Executive Summary

The AI Immigration Form Assistant handles highly sensitive Personally Identifiable Information (PII) including SSNs, passport numbers, visa status, and immigration records. This document prescribes a security-first architecture designed to meet SOC2 Type II compliance, protect against data exfiltration, and manage third-party LLM API risks.

**Recommendation: Hybrid Zero-Knowledge + Encryption-at-Rest model** with strict API isolation and client-side processing for sensitive data.

---

## 1. Data Classification & Threat Model

### 1.1 Data Sensitivity Tiers

**Tier 1: Sensitive PII (Highest Risk)**
- Social Security Numbers (SSN)
- Passport Numbers
- Visa/Immigration Status details
- Date of Birth
- Address information
- Contact Information (in context)

**Tier 2: Form Data (High Risk)**
- Employment history
- Financial information
- Relationship status
- Criminal history disclosures
- Educational background

**Tier 3: Metadata (Medium Risk)**
- User email
- Session IDs
- IP addresses
- Timestamps
- Form completion progress

### 1.2 Attack Vectors

1. **Data Breach in Transit** - Interception of PII during API calls
2. **LLM API Exposure** - Sending raw PII to third-party LLM providers
3. **Database Compromise** - Unauthorized access to encrypted or unencrypted data at rest
4. **Insider Threat** - Malicious employee/contractor access
5. **Compliance Violation** - Failure to meet SOC2/privacy regulations
6. **Inference Attacks** - Reverse-engineering user data from model responses

---

## 2. Encryption Strategy

### 2.1 Data at Rest

**Requirement: AES-256-GCM encryption for all Tier 1 and Tier 2 data**

```
┌─────────────────────────────────────────┐
│  User Device (Client-Side Encryption)   │
│                                         │
│  Raw PII → AES-256-GCM Encrypt        │
│  (per-user key derived from password) │
└──────────────────────────────────────────┘
                    ↓ (encrypted blob)
┌──────────────────────────────────────────┐
│  Application Server (Encrypted Storage)  │
│                                         │
│  Encrypted Blob → Database (PostgreSQL) │
│  + Application-level encryption wrapper │
└──────────────────────────────────────────┘
```

**Implementation Details:**

1. **Client-Side Encryption (Primary)**
   - Use TweetNaCl.js or libsodium.js for browser-based encryption
   - Derive encryption key from user password (PBKDF2-SHA256, 600K iterations)
   - User never transmits plaintext PII over the network
   - Encryption happens before form submission

2. **Server-Side Encryption (Defense in Depth)**
   - All Tier 1/2 data re-encrypted at application layer
   - Master key stored in AWS Secrets Manager / HashiCorp Vault
   - Key rotation policy: quarterly (automatic, transparent to users)
   - Database-level encryption (PostgreSQL pgcrypto or transparent encryption)

**Key Management:**
```
User Password (input)
  ↓
PBKDF2-SHA256 (600K iterations + per-user salt)
  ↓
User Encryption Key (stored in browser memory only)
  ↓
AES-256-GCM Encrypt PII
  ↓
Ciphertext + metadata → sent to server

Server receives ciphertext
  ↓
Master Key (AWS Secrets Manager)
  ↓
Re-encrypt or store as-is (client key derivation makes it unreadable to server)
  ↓
PostgreSQL with transparent encryption
```

### 2.2 Data in Transit

**Requirement: TLS 1.3 with HSTS and certificate pinning**

1. **TLS Configuration**
   - Minimum TLS 1.3 (TLS 1.2 with strong ciphers acceptable for legacy clients)
   - ECDHE key exchange (prevents replay attacks)
   - AEAD ciphers only (AES-256-GCM, ChaCha20-Poly1305)
   - HSTS: `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`

2. **Certificate Management**
   - Use Let's Encrypt with auto-renewal
   - Implement certificate pinning (public key pinning) for native mobile app
   - Monitor certificate expiry (automated alerts at 30, 14, 7 days)

3. **API Communication**
   - All API endpoints HTTPS-only
   - API tokens transmitted in Authorization header (not query params)
   - Request signing for sensitive operations (HMAC-SHA256)

**Example Request:**
```
POST /api/v1/form/auto-fill HTTP/1.1
Host: immigrationforms.app
Authorization: Bearer <JWT>
Content-Type: application/json
X-Request-Signature: HMAC-SHA256(request_body + timestamp)
X-Request-Timestamp: 1711382400

{
  "encrypted_pii": "<AES-256-GCM ciphertext>",
  "form_type": "I-130"
}
```

---

## 3. Zero-Knowledge Architecture

### 3.1 When to Use Zero-Knowledge (ZK)

**YES - Apply Zero-Knowledge:**
1. Storing form responses on server
2. Generating completion reports
3. Risk assessment/validation
4. Multi-form state management

**NO - Don't Apply Zero-Knowledge:**
1. PDF auto-fill (must decrypt locally to populate fields)
2. Real-time form validation (requires decryption on server)
3. User authentication (requires plaintext comparison)

### 3.2 Zero-Knowledge Implementation

**Approach: Client-Side Decryption + Server-Side ZK Proofs**

```
┌──────────────────────────────────────────┐
│  Client (Browser)                        │
│                                         │
│  1. Decrypt PII using user key         │
│  2. Validate form locally              │
│  3. Encrypt again for server storage   │
│  4. Send encrypted + ZK commitment     │
└──────────────────────────────────────────┘
        ↓ (never sees plaintext on server)
┌──────────────────────────────────────────┐
│  Server                                  │
│                                         │
│  1. Verify ZK proof (validity, ranges) │
│  2. Store encrypted blob (no decrypt)  │
│  3. Generate risk warnings from proof  │
│  4. Never decrypt or log plaintext     │
└──────────────────────────────────────────┘
```

**Specific Implementation:**

1. **Form Validation Commitment**
   - Client computes SHA-256(plaintext_field) for each sensitive field
   - Sends hash commitment + encrypted data + ZK range proof
   - Server verifies proof (e.g., "SSN is 9 digits") without decryption

2. **Risk Assessment ZK**
   - Use homomorphic encryption for cross-field validation
   - Example: Verify "visa expiry > today" without decrypting date
   - Library: HElib (IBM), SEAL (Microsoft)

3. **Multi-Form Consistency**
   - Create cryptographic commitments to prevent tampering
   - Merkle tree of form submissions for audit trail

### 3.3 Practical ZK Example: SSN Validation

```typescript
// Client-side
const ssn = "123-45-6789";
const userKey = derive_key(password);

// Create ZK commitment (prove SSN is valid format without revealing)
const commitment = {
  ssn_hash: SHA256(ssn),
  ssn_length_proof: ZK.prove_range(ssn.length, 9, 9), // exactly 9 digits
  encrypted_ssn: AES256_GCM(ssn, userKey)
};

// Send to server
POST /api/form/validate
{
  encrypted_ssn: commitment.encrypted_ssn,
  ssn_hash: commitment.ssn_hash,
  range_proof: commitment.ssn_length_proof
}

// Server-side
const isValid = ZK.verify_range_proof(range_proof, 9, 9);
if (isValid) {
  store_encrypted(encrypted_ssn); // Never decrypt
}
```

---

## 4. LLM API Security

### 4.1 Architecture: Proxy Pattern with Redaction

**Problem:** OpenAI/Claude APIs have data retention policies; sending raw PII violates security requirements.

**Solution:** API proxy with automatic redaction + role-based processing

```
┌─────────────────────────────────┐
│  User Encrypted Data            │
│  {ssn: "XXX-XX-6789", ...}     │
└─────────────────────────────────┘
           ↓
┌─────────────────────────────────┐
│  Internal Redaction Layer       │
│  (Your Server)                  │
│                                 │
│  SSN → REDACTED_SSN_12345      │
│  Passport → REDACTED_PASS_XXXX │
│  Address → [REDACTED]           │
└─────────────────────────────────┘
           ↓
┌─────────────────────────────────┐
│  LLM API Call (Claude/OpenAI)   │
│  Pseudonymized data only        │
│  {form_type: "I-130",           │
│   ssn_validity: true,           │
│   has_visa: true,               │
│   inconsistencies: [...]}       │
└─────────────────────────────────┘
```

### 4.2 Specific Redaction Rules

**Tier 1 Data - Complete Redaction:**
```javascript
function redactPII(formData) {
  return {
    // Never send these to LLM API
    ssn: formData.ssn ? "REDACTED" : undefined,
    passport_number: formData.passport ? "REDACTED" : undefined,
    date_of_birth: formData.dob ? "[DATE_REDACTED]" : undefined,

    // Send metadata only
    ssn_valid: isValidSSN(formData.ssn),
    passport_valid: isValidPassport(formData.passport),
    age_over_18: calculateAge(formData.dob) > 18,

    // Send non-identifying context
    form_type: formData.form_type,
    field_count: Object.keys(formData).length,
    visa_category: formData.visa_category,
    inconsistencies_detected: detectInconsistencies(formData)
  };
}
```

### 4.3 API Call Isolation

**Requirement: LLM API calls must NOT receive original encrypted data**

1. **Create an Internal Service Layer:**
   - Separate microservice for LLM interactions
   - Receives only redacted data
   - No access to user encryption keys
   - Logs: timestamp, redacted inputs, LLM response (no PII)

2. **Data Flow Isolation:**
   ```
   API Gateway (auth, rate limit)
      ↓
   Data Service (handles encryption/decryption)
      ↓
   LLM Proxy Service (redacts, calls API, logs safely)
      ↓ (returns)
   Data Service (re-encrypts response)
      ↓
   Client
   ```

3. **API Key Management for LLM Providers:**
   - Store in HashiCorp Vault (not environment variables)
   - Rotate quarterly
   - Use API key scoping (if provider supports):
     - Read-only scopes
     - IP whitelist your servers only
     - Time-limited tokens for batch jobs

4. **Example Request to Internal LLM Proxy:**
   ```json
   {
     "request_id": "uuid-4",
     "form_type": "I-130",
     "redacted_data": {
       "ssn_valid": true,
       "passport_valid": true,
       "visa_status": "F-1",
       "employment_duration_years": 2,
       "inconsistencies": ["employment_dates_overlap"]
     },
     "analysis_type": "risk_assessment"
   }
   ```

### 4.4 LLM Response Handling

**Never log or store raw LLM responses if they contain reconstructed PII**

```typescript
// WRONG - stores PII in response
const response = await openai.chat.completions.create({
  messages: [{ role: "user", content: redacted_prompt }]
});
logger.info("LLM response:", response); // Risk: may contain inferred PII

// RIGHT - extract only actionable insights
const response = await openai.chat.completions.create({...});
const safeInsights = {
  risk_level: extractRiskLevel(response),
  inconsistency_flags: extractFlags(response),
  missing_required_fields: extractMissingFields(response),
  // Do NOT log the raw response
};
logger.info("Form analysis:", safeInsights);
storeEncrypted(safeInsights, userKey);
```

---

## 5. SOC2 Type II Compliance

### 5.1 Trust Service Criteria (TSC)

**CC: Common Criteria (Security)**
1. **CC6.1: Information and Assets**
   - Maintain encrypted inventory of all data stores
   - Encryption key audit log
   - Data retention policy (delete after 1 year if user requests)

2. **CC6.2: Configuration Change Management**
   - All changes to encryption/network config require review
   - Automated rollback for failed deployments
   - Change log retention (7 years)

3. **CC7.2: System Monitoring**
   - Real-time alerts for unauthorized decryption attempts
   - Database access logs (all queries)
   - API rate limit monitoring

**A&A: Availability & Confidentiality**
1. **Encryption keys**
   - Multi-region backup (AWS Secrets Manager replicated)
   - Key recovery procedures documented
   - Never log encryption keys

2. **Access Control**
   - MFA required for all employees
   - Role-based access (no "god" accounts)
   - VPN required for database access

**PI: Processing Integrity**
1. **Data Validation**
   - Client + server-side validation
   - Reject malformed encryption
   - Integrity checks on encrypted data

### 5.2 Required Policies

**1. Data Retention & Deletion**
```
- User forms: 1 year after completion or deletion request
- Audit logs: 7 years (legal requirement)
- Backups: 90 days (encrypted)
- Unencrypted PII: never stored
- Deletion: cryptographic erasure (key destruction)
```

**2. Access Control Policy**
```
- Admin access: requires 2 MFA methods + manager approval
- Database access: restricted to Data Service layer only
- API key access: Vault + MFA
- Contractor access: automatically revoked after 30 days inactivity
- Code review: required for all encryption/security changes
```

**3. Incident Response**
```
- Data breach: notify users within 24 hours (if PII accessed)
- Ransomware: isolated backup + restore within 1 hour
- API compromise: revoke keys, rotate secrets within 15 minutes
- Documentation: incident report filed within 48 hours
```

**4. Penetration Testing**
```
- Annual third-party pen test
- Quarterly internal security audit
- OWASP Top 10 verification
- Encryption strength validation
```

### 5.3 Documentation Requirements

For SOC2 Type II audit, maintain:
- Risk assessment (quarterly updated)
- Security incident log
- Access control matrix
- Data flow diagrams (with encryption points marked)
- Disaster recovery plan + testing results
- Employee training records (security awareness)
- Third-party vendor security assessments

---

## 6. Technical Implementation Stack

### 6.1 Recommended Architecture

**Frontend (Client-Side Encryption)**
```
Browser/App
├── libsodium.js (encryption/decryption)
├── TweetNaCl.js (key derivation)
├── HTML5 Local Storage + IndexedDB (encrypted cache only)
└── Form validation + error handling

Encryption happens BEFORE data leaves device
```

**Backend (API Server)**
```
Node.js/Python/Go + Express/FastAPI/Gin
├── TLS termination (nginx reverse proxy)
├── Rate limiting + DDoS protection (Cloudflare)
├── Authentication (OAuth2 + MFA)
├── Data validation + redaction layer
├── Database abstraction layer (never direct DB queries)
└── Structured logging (no PII logged)

Database: PostgreSQL 14+
├── pgcrypto extension (server-side encryption)
├── Row-level security (users see only own data)
├── Encrypted backups
└── Automated VACUUM + ANALYZE

Secrets Management: AWS Secrets Manager
├── Master encryption key (AWS KMS)
├── API keys for LLM providers
├── Database credentials
└── Automatic rotation
```

**LLM Proxy Service (Isolated)**
```
Separate microservice
├── Receives redacted data only
├── Calls OpenAI/Claude APIs
├── Extracts safe insights
├── Logs only non-sensitive metrics
└── No database access (stateless)

Deployment: AWS Lambda or separate container
Scaling: auto-scale based on queue depth
```

### 6.2 Deployment & Infrastructure

**Cloud Infrastructure (AWS Example)**
```
┌─────────────────────────────────────┐
│ CDN (CloudFront)                    │
│ ├─ HTTPS/TLS termination           │
│ └─ Cache static assets only        │
└──────────────┬──────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ WAF (Web Application Firewall)       │
│ ├─ OWASP Top 10 protection          │
│ ├─ Rate limiting                    │
│ └─ SQL injection prevention         │
└──────────────┬──────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ API Gateway (ALB/NLB)                │
│ ├─ Auth validation                  │
│ ├─ Request signing verification     │
│ └─ CORS + security headers          │
└──────────────┬──────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ Application Servers (ECS Fargate)    │
│ ├─ VPC + private subnets            │
│ ├─ Auto-scaling (CPU/memory)        │
│ └─ No outbound internet access      │
└──────┬──────────────────────────────┘
   ┌───┴─────────────────┬──────────┐
   ↓                     ↓          ↓
┌─────────────┐  ┌──────────────┐  ┌────────────┐
│ RDS Aurora  │  │ Vault/Secrets│  │ Lambda LLM │
│ (encrypted) │  │ Manager      │  │ Proxy      │
│ (backup)    │  │ (encrypted)  │  │ (isolated) │
└─────────────┘  └──────────────┘  └────────────┘
```

---

## 7. Compliance Checklist

### 7.1 Legal/Regulatory

- [ ] **Privacy Policy** - Explicit consent for data usage
- [ ] **Terms of Service** - Data processing, liability limitations
- [ ] **GDPR Compliance** (if serving EU users)
  - [ ] DPIA (Data Protection Impact Assessment)
  - [ ] DPA with third-party vendors
  - [ ] Right to deletion implemented
  - [ ] Data portability (export user data)
- [ ] **CCPA Compliance** (if serving California users)
  - [ ] Consumer privacy notice
  - [ ] Right to delete
  - [ ] Opt-out mechanisms
- [ ] **HIPAA** (if handling medical history, immigration medical exams)
  - [ ] Business Associate Agreements
  - [ ] Minimum necessary principle
  - [ ] Audit controls
- [ ] **Immigration Law Disclaimer**
  - [ ] NOT providing legal advice
  - [ ] Recommend consulting immigration attorney
  - [ ] Liability waiver in ToS

### 7.2 Technical Controls

- [ ] **Encryption at Rest:** AES-256-GCM for Tier 1 data
- [ ] **Encryption in Transit:** TLS 1.3, HSTS enabled
- [ ] **Key Management:** Vault/Secrets Manager, quarterly rotation
- [ ] **API Security:** Redaction proxy, rate limiting, request signing
- [ ] **Database Security:** Row-level security, encrypted backups
- [ ] **Access Control:** MFA, VPN requirement, audit logging
- [ ] **Zero-Knowledge:** Commitment proofs for validation
- [ ] **Logging:** Structured logs, no PII logged, 7-year retention
- [ ] **Monitoring:** Real-time alerts for unauthorized access
- [ ] **Secrets Management:** No hardcoded credentials, auto-rotation

### 7.3 Operational Controls

- [ ] **Incident Response Plan** - documented, tested quarterly
- [ ] **Disaster Recovery Plan** - RTO < 1 hour, RPO < 15 minutes
- [ ] **Backups:** Daily, encrypted, geo-redundant, tested monthly
- [ ] **Pen Testing:** Annual (external), quarterly (internal)
- [ ] **Security Training:** All employees, annual refresh
- [ ] **Change Management:** Code review, automated tests
- [ ] **Vendor Security:** Assessments for LLM providers, hosting
- [ ] **Data Retention:** Policy defined, automated deletion jobs

---

## 8. Risk Mitigation & Residual Risks

### 8.1 Mitigated Risks

| Risk | Mitigation | Residual Risk |
|------|-----------|---------------|
| LLM API data leakage | Redaction proxy + no storage | Low - inference attacks possible |
| Database breach | AES-256-GCM encryption | Low - encryption key required |
| Man-in-the-middle | TLS 1.3 + certificate pinning | Very Low - modern protocols |
| Insider threat | MFA + audit logging + VPN | Medium - determined attacker |
| Ransomware | Encrypted geo-redundant backups | Low - RTO < 1 hour |
| Key compromise | Vault + MFA + rotation | Low - quarterly rotation |
| Form validation bypass | ZK proofs + server validation | Very Low - dual validation |

### 8.2 Acceptable Residual Risks

1. **LLM Inference Attacks**
   - Risk: Claude/OpenAI engineers could reconstruct SSN from response patterns
   - Acceptance: Acceptable (very low probability, high effort required)
   - Mitigation: Use trusted providers with security commitments

2. **Client-Side Malware**
   - Risk: User device compromised, encryption key stolen
   - Acceptance: Acceptable (outside app's control, user responsibility)
   - Mitigation: Security awareness, HTTPS-only, no cached plaintext

3. **Social Engineering**
   - Risk: Attacker tricks employee into revealing secrets
   - Acceptance: Acceptable with mitigations
   - Mitigation: MFA, security training, need-to-know access

---

## 9. Security Operations (SecOps)

### 9.1 Continuous Monitoring

**Real-Time Alerts (PagerDuty/OpsGenie)**
```
- Database access from unexpected IP
- Failed decryption attempts (>5/minute)
- API key usage anomalies
- Encryption key access outside rotation schedule
- SSL certificate expiry warning (30 days)
- Rate limit breaches (>100 req/min from single IP)
```

**Daily Automated Checks**
```
- Certificate validity
- Encryption key rotation status
- Backup completion + integrity
- Secret rotation schedule adherence
- Dependency vulnerability scan
```

### 9.2 Log Aggregation & Analysis

**ELK Stack / Datadog / CloudWatch**
```
Structure:
{
  timestamp: ISO-8601,
  level: DEBUG/INFO/WARN/ERROR,
  service: "api" | "llm-proxy" | "auth",
  event_type: "encryption_key_access" | "api_call" | "error",
  user_id: HASH(user_id), // NOT plaintext
  request_id: uuid,
  action: "decrypt_attempt" | "api_call",
  result: "success" | "failure",
  error_code: "E001",
  duration_ms: 45,
  // NEVER include: SSN, password, plaintext PII
}
```

Retention: 7 years (legal), encrypted storage

### 9.3 Quarterly Security Reviews

- Review access logs for anomalies
- Test disaster recovery procedures
- Validate encryption key availability
- Update threat model based on new CVEs
- Review third-party vendor security posture

---

## 10. Implementation Roadmap

### Phase 1: MVP (Weeks 1-4)
- [ ] Client-side encryption (libsodium.js)
- [ ] Server-side HTTPS + TLS 1.3
- [ ] Database encryption at rest
- [ ] Basic authentication (OAuth2)
- [ ] LLM API redaction proxy
- [ ] Privacy policy + ToS

### Phase 2: Hardening (Weeks 5-8)
- [ ] Zero-knowledge proofs for validation
- [ ] Audit logging (no PII)
- [ ] Rate limiting + DDoS protection
- [ ] MFA for employees
- [ ] Secrets Manager integration
- [ ] Disaster recovery plan

### Phase 3: Compliance (Weeks 9-12)
- [ ] SOC2 readiness audit (internal)
- [ ] Penetration testing
- [ ] GDPR/CCPA compliance review
- [ ] Incident response plan (tested)
- [ ] Data retention automation
- [ ] Security awareness training

### Phase 4: Certification (Weeks 13+)
- [ ] SOC2 Type II audit
- [ ] Bug bounty program
- [ ] Annual penetration test
- [ ] Continuous security improvements

---

## 11. Conclusion & Key Recommendations

### Core Principles

1. **Never Trust the Network** - Encrypt client-side, assume interception
2. **Defense in Depth** - Multiple layers (client encryption, TLS, database encryption, access control)
3. **Zero-Trust for Employees** - MFA, VPN, audit all access
4. **Isolate LLM APIs** - Redaction proxy, never send raw PII
5. **Log Safely** - Structured logs, no PII, long retention

### Go/No-Go Decision

**This product is buildable with strong security, BUT requires:**
- 3-4 months development (security properly implemented)
- $20-30K infrastructure + services (AWS KMS, Vault, Datadog, etc.)
- Quarterly SOC2 audits ($5-10K each)
- Dedicated security engineer (or contractor review)

**Recommended Stack:**
- **Frontend:** React + libsodium.js + TweetNaCl.js
- **Backend:** Node.js + Express (or Python/FastAPI)
- **Database:** PostgreSQL 14+ with pgcrypto
- **Secrets:** AWS Secrets Manager + KMS
- **API Gateway:** CloudFlare (WAF + DDoS) + AWS ALB
- **Monitoring:** Datadog or CloudWatch + custom alerts
- **LLM Proxy:** AWS Lambda or ECS Fargate (isolated)

**Expected Compliance Path:**
1. Week 4: Privacy/Security fundamentals
2. Week 8: SOC2-ready (internal validation)
3. Month 4: SOC2 Type II audit (pass)
4. Ongoing: Quarterly assessments, annual pen test

---

## Appendix A: Code Examples

### A.1 Client-Side Encryption (JavaScript)

```javascript
import nacl from 'tweetnacl';
import { secretbox, randomBytes, secretbox as box } from 'tweetnacl';

// Key derivation from password
async function deriveKeyFromPassword(password, salt) {
  const encoder = new TextEncoder();
  const data = encoder.encode(password);

  const hash = await crypto.subtle.importKey(
    'raw',
    data,
    { name: 'PBKDF2' },
    false,
    ['deriveKey']
  );

  const key = await crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      hash: 'SHA-256',
      salt: salt,
      iterations: 600000
    },
    hash,
    { name: 'AES-GCM', length: 256 },
    true,
    ['encrypt', 'decrypt']
  );

  return key;
}

// Encrypt sensitive field before sending
async function encryptPII(plaintext, userPassword, salt) {
  const key = await deriveKeyFromPassword(userPassword, salt);
  const encoder = new TextEncoder();
  const data = encoder.encode(plaintext);
  const iv = crypto.getRandomValues(new Uint8Array(12));

  const ciphertext = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv: iv },
    key,
    data
  );

  return {
    ciphertext: Array.from(new Uint8Array(ciphertext)),
    iv: Array.from(iv),
    salt: Array.from(salt)
  };
}

// Usage
const ssn = "123-45-6789";
const password = userPassword;
const salt = crypto.getRandomValues(new Uint8Array(16));

const encrypted = await encryptPII(ssn, password, salt);

// Send to server
fetch('/api/form/encrypt-field', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    field: 'ssn',
    encrypted_value: encrypted,
    form_id: formId
  })
});
```

### A.2 Server-Side Redaction (Node.js)

```javascript
const crypto = require('crypto');

function redactPII(formData) {
  const redacted = {};

  const sensitiveFields = {
    'ssn': data => 'XXX-XX-' + data.slice(-4),
    'passport_number': data => 'REDACTED',
    'date_of_birth': data => '[DATE_REDACTED]',
    'drivers_license': data => 'REDACTED',
    'visa_number': data => 'REDACTED'
  };

  for (const [field, value] of Object.entries(formData)) {
    if (sensitiveFields[field]) {
      redacted[field] = sensitiveFields[field](value);
    } else {
      redacted[field] = value;
    }
  }

  // Add metadata for LLM analysis
  return {
    ...redacted,
    ssn_valid: /^\d{3}-\d{2}-\d{4}$/.test(formData.ssn),
    passport_valid: formData.passport_number ? true : false,
    has_visa: formData.visa_number ? true : false,
    analysis_timestamp: new Date().toISOString()
  };
}

app.post('/api/form/analyze', authenticate, async (req, res) => {
  const { encrypted_form, user_id } = req.body;

  // Decrypt only if allowed
  // (most operations should work on redacted data)

  const redacted = redactPII(decryptedForm);

  // Call LLM with redacted data ONLY
  const llmResponse = await callLLMProxy({
    form_type: redacted.form_type,
    redacted_data: redacted
  });

  // Encrypt response before returning to client
  const encrypted = await encryptResponse(llmResponse, user_id);

  res.json({ success: true, analysis: encrypted });
});
```

### A.3 Secrets Manager Integration (AWS)

```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager({ region: 'us-east-1' });

async function getSecret(secretName) {
  try {
    const data = await secretsManager.getSecretValue({ SecretId: secretName }).promise();

    if ('SecretString' in data) {
      return JSON.parse(data.SecretString);
    } else {
      // Binary secret
      const buff = Buffer.from(data.SecretBinary, 'base64');
      return buff.toString('ascii');
    }
  } catch (error) {
    console.error('Failed to retrieve secret:', secretName);
    throw error;
  }
}

// Usage
const dbPassword = await getSecret('prod/rds/password');
const openaiKey = await getSecret('prod/openai/api-key');
const encryptionKey = await getSecret('prod/encryption/master-key');

// NEVER hardcode, NEVER log secrets
```

---

## References

- NIST SP 800-175B: Guidelines for Media Sanitization
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- SOC2 Trust Service Criteria: https://www.aicpa.org/soc2
- TLS 1.3 RFC: https://tools.ietf.org/html/rfc8446
- GDPR Article 28 (DPA requirements): https://gdpr-info.eu/art-28-gdpr/
- CCPA Consumer Privacy Act: https://oag.ca.gov/privacy/ccpa
- Zero-Knowledge Proofs: https://en.wikipedia.org/wiki/Zero-knowledge_proof

