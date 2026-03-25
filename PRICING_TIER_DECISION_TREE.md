# Pricing Tier Decision Tree
## Quick Reference for Feature Justification & Pricing

---

## QUICK DECISION FRAMEWORK

**Does this feature justify a price increase? Answer these 3 questions:**

### Question 1: Customer Value
**How much does this feature save the customer?**

- **$500+** (replaces lawyer cost) → Justify $25-30 premium
- **$200-500** (saves major time/stress) → Justify $15-20 premium
- **$50-200** (convenience feature) → Justify $5-10 premium
- **<$50** → Include free with tier (not premium)

### Question 2: Implementation Cost
**How much engineering effort does this require?**

- **<2 weeks** (low effort) → Can be Standard or add-on
- **2-4 weeks** (medium effort) → Premium tier feature
- **4+ weeks** (high effort) → Elite tier feature or outsource

### Question 3: Repeat Revenue
**Will customers purchase again or renew?**

- **Yes, 2-3 times per customer** (high LTV) → Justify premium pricing
- **Yes, 1-2 times per customer** (medium LTV) → Justify modest premium
- **One-time purchase** (low LTV) → Keep price low or bundle

---

## FEATURE PRIORITY MATRIX

```
        HIGH VALUE ($500+)
             |
        _____|_____
       /  |  |  |  \
      /   |  |  |   \
     | Elite Tier    |  (RFE Helper, Lawyer Consult)
     | Features      |
     |   |  |  |   |
     ----+--+--+-----
         | Standard  |  (Case Tracking, Expert Review)
         | Features  |
         |   |  |   |
    _____+---+--+-----
   /         |       \
  Standard   Premium  Elite
  ($29)      ($59)    ($99)

      ↓
  LOW VALUE (<$50)
```

---

## FEATURE JUSTIFICATION TABLE (FOR QUICK REFERENCE)

| Feature | Customer Value | Implementation | Repeat Value | Justifies Tier | Recommended Price |
|---------|---|---|---|---|---|
| **Case Status Tracking** | $200-300 (saves lawyer followup) | 4 weeks | Yes (track 2-3 cases/lifetime) | Premium | $15 add-on or in $59 |
| **Document Checklist** | $100-150 (prevents RFE) | 2 weeks | Yes (for each form) | Standard/Premium | $10 add-on |
| **RFE Response Helper** | $500-1,000 (replaces lawyer review) | 6 weeks | Maybe (only if RFE) | Elite | $25-30 premium |
| **Interview Prep** | $200-300 (confidence + tips) | 5 weeks | No (one-time interview) | Premium add-on | $15 add-on |
| **Renewal Reminders** | $50-100 (prevents missed deadlines) | 2 weeks | Yes (annual, recurring) | Standard or Premium | $10/year add-on |
| **Priority Support** | $300-500 (replaces lawyer retainer) | 8 weeks | Yes (ongoing) | Elite | $20 premium |
| **Lawyer Consultation** | $300-500 per referral | 6 weeks | Yes (commission model) | Elite | 20-30% commission |
| **Multi-Form Bundle** | $100+ savings (I-130 + I-485 + N-400) | 3 weeks | Yes (family cases) | All tiers | 15-20% discount |

---

## PRICING DECISION TREE

```
START: New Feature Request
│
├─ Does it replace lawyer cost ($200+)?
│  │
│  ├─ YES → Goes to Premium ($59) or Elite ($99) only
│  │  │
│  │  └─ How complex is the feature?
│  │     ├─ <4 weeks → Premium ($59, add $10-15)
│  │     └─ 4+ weeks → Elite ($99, add $25-30)
│  │
│  └─ NO → Goes to Standard ($29) or add-on ($+5-10)
│
├─ Will customers purchase multiple times (repeat value)?
│  │
│  ├─ YES (2-3x per customer lifetime) → Justify premium pricing
│  │
│  └─ NO (one-time purchase) → Keep low price or bundle
│
├─ Can we build this in 4 weeks or less?
│  │
│  ├─ YES → Launch as add-on or Standard feature
│  │
│  └─ NO → Phase into Premium/Elite tiers OR outsource
│
└─ DECISION
   ├─ Standard tier (+$0, all users)
   ├─ Premium tier (+$10-15, 20% of users)
   ├─ Elite tier (+$25-30, 10% of users)
   ├─ Add-on feature (+$5-30, optional)
   └─ Bundle feature (10-20% discount for 3+ forms)
```

---

## FEATURE LAUNCH ROADMAP (BY TIER)

### STANDARD TIER ($29) - Must Have by V1 Launch
- [x] Interactive Q&A engine
- [x] PDF auto-fill + export
- [x] Consistency checker (40+ rules)
- [x] Document checklist
- [x] Renewal reminders
- [x] Email support (24-48hr)
- [x] Offline access (PWA)

**Don't add anything else to Standard after V1; keep it simple and cheap.**

### PREMIUM TIER ($59) - Launch Weeks 1-4 Post-V1
- [ ] Case status tracking (1 case)
- [ ] Document upload + OCR (5 docs)
- [ ] Priority support (4-6hr response)
- [ ] 1 expert form review included
- [ ] Access to 3-form bundle ($69, save $18)
- [ ] Interview prep (practice questions)

**Features to add if time permits:**
- [ ] Advanced interview coaching (audio feedback)
- [ ] Multi-case tracking (up to 3 cases)

### ELITE TIER ($99) - Launch Weeks 5-10 Post-V1
- [ ] Everything in Premium +
- [ ] RFE response helper (unlimited)
- [ ] Priority support (2hr chat + phone)
- [ ] Unlimited expert reviews
- [ ] Lawyer consultation booking (1 free)
- [ ] Case strategy calls (quarterly)
- [ ] 5-form bundle discount
- [ ] Early access to new features

**Never add:**
- [ ] Live lawyer on retainer (too expensive)
- [ ] Notary services (legal liability)
- [ ] Payment processing for USCIS fees (too complex)
- [ ] Complete document preparation service (hire law firm instead)

---

## REVENUE IMPACT BY FEATURE

### Quick Revenue Estimates (at 1,000 customers)

| Feature | % Adoption | Avg Price | Monthly Revenue |
|---------|-----------|-----------|---|
| Case Status Tracking | 20% of Premium (68 customers) | $15 | $1,020 |
| Document Checklist/Upload | 30% of all | $10 | $1,000 |
| RFE Response Helper | 8% of Elite (27 customers) | $25 | $675 |
| Interview Prep | 15% of Premium (102 customers) | $15 | $1,530 |
| Renewal Reminders | Free for Premium, $10/year for Standard | - | $100 |
| Priority Support | 20% + 10% (Elite) | $20 | $2,000 |
| Lawyer Referrals | 3% of all customers | $100 commission | $3,000 |
| 3-Form Bundle | 40% of new customers | $69 (vs $87) | $2,300 |
| **TOTAL ADD-ON REVENUE** | | | **$11,625/mo** |

**Base tier revenue (all customers):**
- 700 Standard × $29 = $20,300
- 200 Premium × $59 = $11,800
- 100 Elite × $99 = $9,900
- **Base total: $42,000/mo**

**Total revenue: $42,000 + $11,625 = $53,625/mo = $643K ARR**

(vs. V1 baseline of $103K ARR @ $29 blended = $2,950/mo = $35,400 annual)

---

## PRICING COMPARISON: CURRENT VS. RECOMMENDED

### Current V1 Pricing
```
$29/form (one-time) or $39/month
Average customer revenue: $29-39
Year 1 ARR: $103K
```

### Recommended Multi-Tier Pricing
```
Standard:  $29
Premium:   $59 (+103% vs. Standard)
Elite:     $99 (+241% vs. Standard)

Average customer revenue: $49-55
Year 1 ARR: $135K-165K (+31-60% growth)
```

### Why Customers Accept Price Increase

| Tier Upgrade | Justification | Value Delivered |
|---|---|---|
| Standard → Premium (+$30) | Prevents $2,000 RFE legal fees | Case tracking + expert review + priority support |
| Premium → Elite (+$40) | Replaces $500-1,000 lawyer consultation | RFE specialist + lawyer referral + 2hr support |
| Any tier → Add-ons (+$15-30) | Solves specific pain point | Interview prep, multi-case tracking, etc. |

---

## OBJECTION HANDLING

**"Why would customers pay more for Premium?"**

Answer: They won't, unless Premium solves a real problem.

**Real objections to overcome:**
- "I just want a cheap form filler" → Keep Standard at $29
- "I can't afford a lawyer review" → Premium ($59) is still cheaper than 1hr lawyer ($300+)
- "I'll never need these features" → Premium is optional; Standard works fine

**Solution:** Only show Premium/Elite to customers with complex cases or sign-up after receiving an RFE.

---

## GO/NO-GO DECISION CHECKLIST

### Before launching Premium Tier ($59)

- [ ] Have at least 100+ Standard customers with positive feedback
- [ ] Case tracking feature is built, tested, and working
- [ ] At least 1 paralegals or legal expert hired for reviews
- [ ] Pricing page A/B tested (Standard vs. Premium messaging)
- [ ] NPS score >40 for Standard tier
- [ ] Support load is manageable (<10 hours/week)

### Before launching Elite Tier ($99)

- [ ] Premium tier has 20%+ adoption (15+ Premium customers)
- [ ] RFE response helper feature is built and tested
- [ ] Lawyer referral network has 10+ signed partners
- [ ] Expert review process is documented + trained
- [ ] Support is staffed (1 FTE paralegal minimum)
- [ ] LTV/CAC ratio is still healthy (>50x)

### Before launching Add-ons

- [ ] Base tier revenue is >$5,000/month
- [ ] Customer churn is <5% monthly
- [ ] Tier adoption is stabilized (70% Standard, 20% Premium, 10% Elite)
- [ ] Feature is proven popular in user surveys

---

## RED FLAGS (Don't Launch This Feature)

- ❌ Feature takes >8 weeks to build (unless it's Elite tier defining feature)
- ❌ Feature has <100 customers requesting it
- ❌ Feature requires hiring full-time staff (unless commission model like lawyer referral)
- ❌ Feature creates legal liability without clear mitigation
- ❌ Feature is "nice to have" but not "must have"
- ❌ Feature cannibalize other features (e.g., lawyer referral cannibalizes RFE helper)
- ❌ Feature requires USCIS API that isn't documented/stable

---

## FINAL RECOMMENDATION

**Launch sequence:**

**Phase 1 (Week 1-4):** Premium tier with case tracking + expert review ($59)
- Quick win (4-week build, immediate $1,500+/mo revenue)
- Establishes premium pricing anchor
- Gathers customer feedback for next features

**Phase 2 (Week 5-10):** Elite tier with RFE helper + priority support ($99)
- Complex feature (6-week build, $2,500+/mo revenue)
- Captures high-value customer segment
- Requires hiring paralegals/ops team

**Phase 3 (Week 11-16):** Multi-form bundles + annual plan
- Quick feature (3-week build, $5,000+/mo revenue)
- Increases ASP, reduces churn
- Leverages existing infrastructure

**Don't launch until:**
1. Standard tier has 50+ paying customers
2. NPS is >40
3. Support load is manageable
4. You have clear product-market fit feedback

**Success = 30%+ of new customers choose Premium/Elite tiers within 3 months of launch**
