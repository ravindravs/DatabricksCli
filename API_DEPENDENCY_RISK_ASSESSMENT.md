# API Dependency Risk Assessment & Mitigation Strategy
## AI Immigration Form Assistant

**Date:** March 2026
**Product:** AI Immigration Form Assistant (I-130, I-485, N-400 forms)
**Critical APIs:** OpenAI/Anthropic Claude (Q&A engine), Stripe (payments), SendGrid (email)

---

## EXECUTIVE SUMMARY

### Current Dependency Profile

The AI Immigration Form Assistant has **moderate-to-high dependency on third-party APIs**:

| API | Purpose | Risk Level | Annual Cost | Criticality |
|-----|---------|-----------|-------------|-------------|
| **OpenAI/Claude** | Q&A guidance, form validation | HIGH | $15K-50K | Critical |
| **Stripe** | Payment processing | MEDIUM | $0.02-$3K | Critical |
| **SendGrid** | Email export/notifications | LOW | $0-500 | Medium |
| **AWS (infrastructure)** | Hosting, secrets, KMS | MEDIUM | $3K-10K | Critical |

### Key Risks Identified

1. **Price Risk:** 10x increase in LLM API costs would increase COGS from ~15% to 150%+
2. **Content Risk:** Anthropic/OpenAI immigration-related restrictions could block functionality
3. **Outage Risk:** API downtime during critical filing deadlines could cause legal/customer damage
4. **Regulatory Risk:** Data handling restrictions could force architecture changes
5. **Dependency Lock-in:** Switching providers takes 4-8 weeks engineering effort

---

## SECTION 1: PRICE INCREASE SCENARIO (10x Cost Shock)

### Current Economics

**Baseline (Realistic Year 1):**
- Revenue: ~$105K ARR (340 customers)
- LLM API costs: ~$5-8K/year (at current rates)
- Gross margin: 85%
- Net margin: 75%

**Per-Customer Costs:**
- Average customer LTV: $241
- LLM cost per customer: $15-24 (API calls for Q&A, validation, risk assessment)
- Infrastructure cost per customer: $18-25
- **Total COGS per customer: ~$33-49 (13-20% of LTV)**

### 10x Price Increase Impact

**Scenario: OpenAI/Claude prices increase 10x**

```
Current pricing: $0.01-0.10 per 1K tokens
New pricing (10x): $0.10-1.00 per 1K tokens

Per-customer LLM cost would increase from $20 → $200
New COGS per customer: $33-49 → $213-229 (exceeds LTV of $241)
```

**Financial Impact:**

| Metric | Before | After (10x) | Change |
|--------|--------|------------|--------|
| **LLM cost per customer** | $20 | $200 | +900% |
| **Total COGS/customer** | $45 | $225 | +400% |
| **Gross margin** | 85% | -6% | **Negative margin** |
| **Unit economics** | Viable | Unsustainable | **Business breaks** |

**Timeline to insolvency:** 2-3 months (existing customer base becomes loss-making)

---

## SECTION 2: CONTENT RESTRICTION SCENARIO (Immigration Data Blocks)

### Regulatory/Policy Risk

Both OpenAI and Anthropic have content policies that could restrict immigration services:

1. **Anthropic Content Policy (Current)**
   - Restricts "helping users evade laws"
   - Could be interpreted to restrict immigration form guidance
   - No explicit immigration carve-out in ToS

2. **OpenAI Usage Policy (Current)**
   - Restricts "illegal activities" (vague)
   - Does NOT explicitly permit immigration assistance (grey zone)
   - Could restrict if users claim reliance on AI for legal decisions

3. **Future Political Risk**
   - If U.S. immigration policy tightens, APIs might add restrictions
   - Example: Could restrict guidance on forms for certain visa categories
   - Could require user geolocation verification

### Specific Failure Scenarios

**Scenario A: Immigration-Specific Block**
- Claude/OpenAI adds policy: "Cannot provide guidance on immigration forms"
- Impact: Q&A engine becomes unusable (core product feature)
- Recovery time: 4-8 weeks to replace with open-source model

**Scenario B: Vague Legal Risk**
- Provider flags immigration assistance as "potentially unlicensed legal advice"
- Threatens account suspension unless ToS updated with legal disclaimers
- Requires legal review, increases liability exposure
- May force shutdown until legal compliance confirmed

**Scenario C: Geopolitical Restrictions**
- If U.S. adds sanctions on certain countries, providers might auto-block form access
- Example: Cannot help users from embargoed nations file immigration forms
- Impact: Customer base restricted to whitelisted countries only

---

## SECTION 3: OUTAGE RISK DURING CRITICAL DEADLINES

### Scenario: API Outage During Immigration Filing Deadline

**Context:** Immigration forms have hard deadlines:
- Receipt dates for visa categories
- Processing times (e.g., I-130 processing: 12-36 months)
- Users often file 1-2 days before deadlines

**Outage Impact:**

| Duration | Impact | Customer Damage |
|----------|--------|-----------------|
| **1 hour** | Users reroute to manual form filling | Low - minor inconvenience |
| **4-8 hours** | Some users miss filing deadlines | **CRITICAL** - legal consequences |
| **24+ hours** | Hundreds of customers miss deadlines | **Catastrophic** - class action risk |

**Legal Exposure:**
- Customer files I-130 2 days before deadline
- API outage prevents form submission
- Deadline missed → visa petition denied → customer sues for damages
- Potential damages: $500-5,000 per customer (legal fees, visa denial consequences)

**Year 1 Exposure:** 340 customers × 10% hit by outage × $2,000 avg damage = **$680K liability**

---

## SECTION 4: MULTI-PROVIDER STRATEGY

### RECOMMENDED APPROACH: Hybrid Redundancy Model

Build system that can **automatically failover between providers** without user-facing disruption.

#### Architecture: Provider Abstraction Layer

```
┌─────────────────────────────────────────────┐
│  User Q&A Form Engine (Frontend)            │
└────────────────────┬────────────────────────┘
                     ↓
┌─────────────────────────────────────────────┐
│  LLM Provider Abstraction (Backend)         │
│  ├─ Check provider health                  │
│  ├─ Route to primary provider              │
│  ├─ If primary fails, route to secondary   │
│  ├─ If both fail, use fallback logic       │
│  └─ Log provider performance               │
└────────────────────┬────────────────────────┘
         ┌───────────┼───────────┬─────────┐
         ↓           ↓           ↓         ↓
    ┌────────┐  ┌────────┐  ┌────────┐  ┌──────────┐
    │ Claude │  │ OpenAI │  │Llama2/ │  │ Fallback │
    │(Primary)│  │(Secondary)│ Mistral │  │ (Rules) │
    │        │  │(tier 2)  │(Local) │  │(Cached) │
    └────────┘  └────────┘  └────────┘  └──────────┘
    $0.01-0.10/   $0.015/    Free (self-  Zero cost
    1K tokens    1K tokens   hosted)      (templates)
```

#### Provider Options (Ranked by Cost)

| Provider | Cost/1K Tokens | Outage History | Immigration-Friendly | Setup Time |
|----------|----------------|----------------|----------------------|------------|
| **Claude (Anthropic)** | $0.01-0.10 | <0.1% downtime | Yes (current) | N/A |
| **OpenAI GPT-4** | $0.015-0.06 | <0.1% downtime | Unclear | 1 week |
| **Llama 2 (Meta)** | Free (self-hosted) | Depends on hosting | Yes (neutral) | 3-4 weeks |
| **Mistral 7B** | Free (self-hosted) | Depends on hosting | Yes (neutral) | 3-4 weeks |
| **Google Gemini** | $0.01-0.08 | <0.1% downtime | Unclear | 2 weeks |
| **Cached Templates** | $0 | N/A | N/A | Built-in |

---

## SECTION 5: MITIGATION PLAN (12-Month Roadmap)

### Phase 1: Cost Insurance (Months 1-3)

**Goal:** Lock in pricing, reduce price shock risk

**Actions:**
1. **Negotiate volume commitment with Claude/OpenAI**
   - Commit to $50K/year volume → negotiate 30-40% discount
   - Lock in pricing for 24 months
   - Add price cap clause: "Price increases capped at 5% annually"

2. **Implement usage optimization**
   - Cache common Q&A responses (reduce API calls 40-50%)
   - Pre-compute validation rules (offline, no API needed)
   - Batch API calls (reduce token overhead)
   - **Expected savings:** 40% reduction in API spend

3. **Set spending budget + alerts**
   - Monthly cap: $500 for Q&A (halt new customers if exceeded)
   - Quarterly audit of cost per customer
   - Implement rate limiting (max 100 API calls per customer)

**Timeline:** Weeks 1-4
**Owner:** Technical Lead + Finance
**Cost:** $0 (renegotiation only)

---

### Phase 2: Fallback Infrastructure (Months 2-4)

**Goal:** Build open-source LLM backup to survive API outages

**Actions:**
1. **Deploy Llama 2 7B locally**
   - Self-host on AWS EC2 (or Modal Labs, Replicate)
   - Total setup cost: $2K infrastructure + 2 weeks engineering
   - Response quality: 80-85% of Claude (acceptable for backup)
   - Latency: 2-3 seconds vs. 0.5s Claude (acceptable for async)

2. **Implement provider routing logic**
   - Primary: Claude
   - Secondary (timeout >2s): Switch to Llama 2
   - Tertiary (error): Cache + return template response
   - Monitor provider health every 60 seconds

3. **Optimize Llama 2 for immigration forms**
   - Fine-tune on 100+ real I-130/I-485 form examples
   - Add LoRA adapter for form-specific language
   - Test against 20 edge cases (names with accents, date formats, etc.)
   - Expected accuracy: 80-90% (vs. 95%+ for Claude)

**Timeline:** Weeks 3-8
**Owner:** ML Engineer (1 FTE)
**Cost:** $2K infrastructure + labor

---

### Phase 3: Content Policy Protection (Months 3-6)

**Goal:** Insulate from provider policy changes

**Actions:**
1. **Add legal disclaimers to product**
   - "This tool provides guidance only, not legal advice"
   - "Consult immigration attorney before filing"
   - "Verify all information with USCIS official instructions"
   - Add these to every page + PDF export

2. **Implement audit trail**
   - Log all form data (encrypted)
   - Document which provider generated each response
   - Enable customer support to trace errors to specific provider
   - Defend against "AI gave bad advice" claims

3. **Diversify Q&A engine**
   - Don't rely on LLM for validation (use hardcoded rules instead)
   - Use LLM only for explanations (low-risk application)
   - Example: Validation = 100% rules-based, explanations = LLM-powered
   - Result: Can survive provider restrictions on sensitive operations

4. **Draft contingency ToS**
   - Include clause: "If core LLM provider becomes unavailable, we may switch providers without notice"
   - Protect against customer claims of service disruption
   - Document data handling compliance with each provider

**Timeline:** Weeks 2-8
**Owner:** Legal + Product
**Cost:** $3-5K legal review

---

### Phase 4: API Outage Resilience (Months 4-8)

**Goal:** Survive 24+ hour API outages without customer impact

**Actions:**
1. **Implement response caching**
   - Cache 1,000 most common Q&A pairs locally
   - Update cache daily (batch pre-compute)
   - Cost: $500/month extra storage
   - Benefit: 70-80% of customer queries hit cache, no API call needed

2. **Build offline-first PWA**
   - All Q&A engine logic runs client-side (already in design)
   - Forms can be completed offline, sync when online
   - Validation rules baked into frontend
   - Result: Product works even if all APIs down

3. **Implement graceful degradation**
   - If LLM unavailable: Show template explanations (pre-written fallbacks)
   - If Stripe unavailable: Show manual payment instructions
   - If SendGrid unavailable: Generate PDF in-browser (no email needed)
   - Never show "Service unavailable" error

4. **Set up monitoring + alerts**
   - Monitor provider API latency (target: <1 second)
   - Alert if any provider response time >5 seconds
   - Auto-failover to backup if latency spike detected
   - PagerDuty integration (wake on-call engineer immediately)

**Timeline:** Weeks 4-12
**Owner:** Infrastructure Lead
**Cost:** $2K/month (AWS Lambda, monitoring services)

---

### Phase 5: Financial Hedge (Months 6-12)

**Goal:** Protect business model from price increases

**Actions:**
1. **Implement cost monitoring + alerts**
   - Track LLM cost per customer monthly
   - Alert if cost increases >5% MoM
   - Budget: "If LLM costs exceed 25% of COGS, trigger mitigation"
   - Current COGS: ~20%, Cost limit: 25%

2. **Build pricing flexibility into product**
   - Ability to tier features by API cost
   - Premium users (higher price) get full LLM features
   - Basic users get template-only explanations
   - Example: $29 (templates) vs. $49 (AI-powered)

3. **Diversify customer base**
   - Don't become over-reliant on single form (I-130)
   - Different forms have different API usage patterns
   - Example: I-485 requires more validation, higher API cost
   - I-130 is simpler, can use cached responses

4. **Plan for margin compression**
   - If API costs double, accept 50% margin instead of 75%
   - Still profitable at $241 LTV if COGS rises to 50%
   - Prepare pricing increase: $29 → $39-49 per form
   - Communicate to customers: "AI technology costs increased"

**Timeline:** Months 6-12
**Owner:** Finance + Product
**Cost:** $0 (planning + monitoring)

---

## SECTION 6: SPECIFIC RISK MITIGATION BY SCENARIO

### Risk 1: Anthropic Prices 10x

**Probability:** Low (3-5% annually)
**Impact:** Business-breaking ($200+ COGS per customer)
**Mitigation:**
1. **Month 1:** Negotiate 24-month pricing lock with 5% increase cap
2. **Month 2:** Implement usage optimization (cache + rate limiting)
3. **Month 4:** Deploy Llama 2 fallback (reduces dependency from 100% to 20%)
4. **If increase happens:**
   - Activate Llama 2 for 80% of queries
   - Keep Claude for high-priority edge cases
   - Reduce API spend by 85%
   - Accept 5-10% quality degradation

**Cost to mitigate:** $5K upfront + $2K/month ongoing

---

### Risk 2: Immigration Content Restrictions

**Probability:** Medium (20-30% over 3 years)
**Impact:** Product core functionality disabled
**Mitigation:**
1. **Month 1:** Add comprehensive legal disclaimers to product
2. **Month 2:** Shift to rules-based validation (reduce LLM risk surface)
3. **Month 4:** Deploy open-source LLM fallback
4. **If restriction happens:**
   - Switch to Llama 2 (neutral provider, no immigration policy)
   - Fallback to rules-only mode (no LLM explanations)
   - Send customer notification: "Using local AI instead of Claude"
   - No product outage, minor quality reduction (80% → 70%)

**Cost to mitigate:** $2K upfront + $1K/month ongoing

---

### Risk 3: API Outage During Deadline

**Probability:** Medium (20-40% chance of outage during customer deadline in Year 1)
**Impact:** $500-2,000 per affected customer (legal liability)
**Mitigation:**
1. **Month 1:** Implement caching (70% of queries don't need API)
2. **Month 4:** Deploy offline-first PWA (product works without APIs)
3. **Month 6:** Add monitoring + auto-failover (switches to backup provider)
4. **If outage happens:**
   - 70% of customers unaffected (cached responses)
   - 30% affected see template explanations (not AI-powered, but functional)
   - Offline users complete forms, sync when APIs return
   - Zero customer impact in most scenarios

**Cost to mitigate:** $3K upfront + $1-2K/month ongoing

---

### Risk 4: Regulatory Data Restrictions

**Probability:** Low (5-10% over 3 years)
**Impact:** Product redesign required, 2-4 week delay
**Mitigation:**
1. **Month 1:** Implement encryption + data isolation (security already designed)
2. **Month 3:** Add data residency options (EU data in EU, US in US)
3. **Month 6:** Build customer data export (satisfy GDPR/CCPA requirements)
4. **If restriction happens:**
   - Already encrypted → easy to prove compliance
   - Quick pivot to GDPR-compliant data handling
   - No product shutdown, minor compliance overhead

**Cost to mitigate:** Included in security architecture (already budgeted)

---

## SECTION 7: OPEN-SOURCE ALTERNATIVE ANALYSIS

### Candidates for Full LLM Replacement

| Model | Size | Quality | Cost | Latency | Immigration-Ready |
|-------|------|---------|------|---------|------------------|
| **Llama 2 70B** | 70B | 90% of Claude | $0 self-hosted | 3-5s | Moderate |
| **Mistral 7B** | 7B | 75% of Claude | $0 self-hosted | 1-2s | Good |
| **Llama 2 7B** | 7B | 70% of Claude | $0 self-hosted | 1-2s | Good |
| **Code Llama** | 7-34B | 65% for forms | $0 self-hosted | 2-3s | Poor |

### Implementation Effort: Llama 2 70B Deployment

**Timeline:** 4 weeks
**Effort:** 1 ML engineer + 0.5 DevOps
**Cost:** $3-5K infrastructure setup

**Quality Trade-offs:**
- Claude accuracy: 95%+ for form guidance
- Llama 2 70B accuracy: 85-90% (acceptable)
- Requires fine-tuning on immigration forms: +2 weeks
- Testing against 100+ edge cases: +1 week

**Deployment Options:**
1. **Self-hosted (AWS EC2 GPU):** $2-3K/month, full control
2. **Replicate/Modal Labs:** $1-2K/month, managed service
3. **On-premises (VPS):** $500/month, latency risk

**Go/No-Go Decision:**
- High confidence implementation can succeed
- Quality acceptable for product (backup, not primary)
- Cost manageable as percentage of revenue
- **Recommendation: Build in Phase 2 as insurance**

---

## SECTION 8: FINANCIAL IMPACT SUMMARY

### Investment Required to Mitigate All Risks

| Phase | Cost | Timeline | Risk Covered |
|-------|------|----------|--------------|
| **Cost Insurance (negotiations + optimization)** | $5K | Months 1-3 | 10x price shock |
| **Fallback LLM (Llama 2 deployment)** | $2K setup + $1-2K/mo | Months 2-4 | Outage, content restriction |
| **Legal/Compliance** | $3-5K | Months 3-6 | Legal liability, policy risk |
| **Monitoring + Alerting** | $1-2K/month | Months 4-8 | Outage during deadline |
| **Caching + Offline** | $500/month | Months 4-8 | API failure resilience |
| **TOTAL Year 1** | **~$20-30K** | Months 1-12 | **All major risks** |

### ROI Calculation

**Baseline Year 1 Revenue:** $105K
**Mitigation Investment:** $30K (all-in)
**Risk Exposure Without Mitigation:** $680K (40% chance of $680K outage liability)
**Expected Value of Risk:** $680K × 40% = $272K

**ROI:** $272K risk reduction ÷ $30K investment = **9x return** (net +$242K value protection)

---

## SECTION 9: IMPLEMENTATION ROADMAP (12-Month Timeline)

### Month 1: Assessment & Planning
- [ ] Review all API contracts (clause language, price caps, exit fees)
- [ ] Audit current API usage (tokens, costs, providers)
- [ ] Set up cost monitoring dashboard
- [ ] Begin OpenAI/Anthropic contract negotiation
- **Owner:** Finance Lead + Technical Lead
- **Effort:** 1 week

### Month 2: Quick Wins
- [ ] Implement response caching (70% query reduction target)
- [ ] Deploy Llama 2 7B test environment (non-production)
- [ ] Add legal disclaimers to product
- [ ] Set up API usage alerts
- **Owner:** Backend Engineer + Product
- **Effort:** 3 weeks

### Month 3: Fallback Deployment
- [ ] Fine-tune Llama 2 70B on immigration forms (100 examples)
- [ ] Deploy to staging environment
- [ ] Test failover logic (switch to Llama 2 when Claude times out)
- [ ] Document contingency procedures
- **Owner:** ML Engineer + DevOps
- **Effort:** 4 weeks

### Month 4: Offline Resilience
- [ ] Implement PWA offline support (already in design)
- [ ] Build template fallback system
- [ ] Test graceful degradation (LLM unavailable scenarios)
- [ ] Set up monitoring dashboards
- **Owner:** Frontend + Infrastructure
- **Effort:** 3 weeks

### Months 5-8: Hardening
- [ ] Deploy production Llama 2 infrastructure
- [ ] Set up auto-failover (Claude → Llama 2 → templates)
- [ ] Implement response caching at CDN level
- [ ] Test disaster recovery procedures
- **Owner:** Infrastructure + QA
- **Effort:** 8 weeks

### Months 9-12: Ongoing Monitoring
- [ ] Monthly cost audits
- [ ] Quarterly risk assessments
- [ ] Annual penetration testing (security)
- [ ] Review and update contingency plans
- **Owner:** Finance + Technical Lead
- **Effort:** Ongoing, 1 week/quarter

---

## SECTION 10: GO/NO-GO CHECKLIST

### Can This Product Survive 10x API Price Increase?

- [x] Yes, with mitigation:
  1. Negotiate pricing lock (reduces shock to ~5% annually)
  2. Implement caching (reduce usage 40-50%)
  3. Deploy Llama 2 fallback (reduce API dependency 80%)
  4. **Combined: Costs rise from $20 → ~$35-40 per customer, still profitable**

### Can This Product Survive Content Restrictions?

- [x] Yes, with mitigation:
  1. Add legal disclaimers (reduce liability)
  2. Shift to rules-based validation (reduce LLM dependency)
  3. Deploy open-source LLM backup (switch providers instantly)
  4. **Combined: Product continues with 80-90% quality, zero downtime**

### Can This Product Survive API Outages?

- [x] Yes, with mitigation:
  1. Implement response caching (70% queries work offline)
  2. Deploy PWA offline support (forms complete without internet)
  3. Add monitoring + failover (switches to backup provider)
  4. **Combined: <5% of customers experience any disruption**

### Can This Product Survive Regulatory Changes?

- [x] Yes, with mitigation:
  1. Implement encryption + data isolation (already designed)
  2. Add data residency options
  3. Build customer data export (GDPR-compliant)
  4. **Combined: Quick compliance update, no product redesign**

### Financial Viability?

- [x] Yes:
  - Investment to mitigate all risks: $30K
  - Revenue at risk from API issues: $272K (expected value)
  - Net protection value: +$242K
  - LTV still positive even with 50% COGS increase ($241 LTV, $120 COGS = profitable)

---

## CONCLUSION & RECOMMENDATIONS

### Strategic Decision: Build with Multi-Provider Architecture (Recommended)

**Why this matters:**
The immigration form market creates acute timing risk. A 24-hour API outage during the peak immigration filing season (spring/fall) could cause 100-200 customers to miss critical deadlines, resulting in:
- Legal damages: $500-5,000 per customer = $50-1M exposure
- Reputation damage: Single negative Reddit post reaches 50K+ immigration-seeking users
- Business viability: Customer churn 50%+ due to trust loss

**This risk is uninsurable** (no provider covers "customer missed immigration deadline due to app failure").

### Investment Plan

**Tier 1 (Essential - Implement immediately):**
- [ ] Cost monitoring + pricing lock negotiation ($0-5K)
- [ ] Response caching to reduce API usage (2 weeks engineering)
- [ ] Legal disclaimers + liability protection (1 week legal)
- **Investment: $5K + 3 weeks**
- **Protects against:** 50% of price shock risk

**Tier 2 (Highly recommended - Implement by Month 4):**
- [ ] Llama 2 70B fallback deployment ($3K + 4 weeks)
- [ ] Offline-first PWA (already designed, needs integration)
- [ ] Monitoring + auto-failover (1 week infrastructure)
- **Investment: $3K + 5 weeks**
- **Protects against:** 100% of outage risk, content restrictions

**Tier 3 (Prudent - Implement ongoing):**
- [ ] Annual contract reviews ($2K legal)
- [ ] Quarterly risk assessments (1 week/quarter)
- [ ] Annual penetration testing ($5-10K/year)
- **Investment: $10K/year**
- **Protects against:** Regulatory changes, emerging risks

### Final Recommendation

**BUILD WITH CONFIDENCE**, but implement Tier 1 + Tier 2 mitigations:
- Core product economics are exceptional (120x LTV/CAC)
- Risks are manageable with $30K and 12-week implementation
- Multi-provider architecture adds 2 weeks to initial development (acceptable)
- Year 1 margin compression (85% → 65%) is acceptable if APIs increase prices
- **Business remains viable even in worst-case scenarios**

The $30K risk investment is 29% of Year 1 revenue, but protects against $272K in expected losses. This is prudent risk management for a capital-efficient business.

