# AI Immigration Form Tool - Conversion Funnel Model & Optimization Strategy

**Date:** March 2026
**Product:** AI Immigration Form Assistant ($29 one-time per form)
**Objective:** Design the complete visitor-to-paying-customer funnel with realistic conversion rates and optimization tactics.

---

## SECTION 1: FUNNEL ARCHITECTURE & CONVERSION RATES

### The Funnel Stages

```
1. AWARENESS (Visitor Lands)
   ↓
2. TRIAL/FREE ACCESS (Explores)
   ↓
3. CONVERSION (Pays $29)
   ↓
4. RETENTION & UPSELL (Repeat purchase)
```

---

## Stage 1: AWARENESS - Traffic Sources (Top of Funnel)

### Where Visitors Come From (Monthly Target)

| Traffic Source | Monthly Visitors | % of Total | Comment |
|---|---|---|---|
| **Organic Search** | 4,000 | 80% | "How to fill I-130" + 50 related keywords |
| **Direct + Referral** | 500 | 10% | Word-of-mouth, Reddit, Facebook groups |
| **Product Hunt / Launch** | 300 | 6% | Day-of-launch traffic spike |
| **Social Media** | 200 | 4% | Twitter, LinkedIn, Instagram organic |
| **TOTAL VISITORS** | **5,000** | **100%** | Baseline for Month 3+ (organic steady state) |

**Key Insight:** 80% of traffic comes from organic search (people Googling in panic mode at 2am). This is *not* paid acquisition—it's free traffic. Therefore, **CAC = $0-$2** (just infrastructure costs).

---

## Stage 2: TRIAL / FREE ACCESS - What's Free?

### Defining the Free Tier

**Question:** "What should be free to maximize conversion to paid?"

**Answer:** First 3 questions + landing page demo (free sample, limited scope)

#### Free Tier Offering (V1)

| Element | Free | Paid ($29) |
|---|---|---|
| **Form field explanations** | First 3 questions only | All 60 questions |
| **PDF preview** | Show what the form looks like | Full PDF auto-fill + download |
| **Consistency checker** | Not included | Full 40+ validation rules |
| **Landing page demo** | 60-second video showing the tool | Access to full interactive experience |
| **Account creation** | Not required | Optional (for email export) |

#### Free Tier Engagement Flow

```
Landing Page (2 min read)
   ↓
Watch 60-second demo video ("See how it works")
   ↓
Try 3 free questions (relationship type, petitioner name, beneficiary name)
   ↓
See AI-powered explanation for each field
   ↓
[Paywall] "Answer all 60 questions for $29"
   ↓
Suggested upsells shown (PDF, consistency check, document checklist)
```

#### Landing Page Demo Video Content

**Duration:** 60 seconds
**Script:**

```
[0-5s] "Filling out the I-130 form is confusing and risky."
[5-15s] "Show person struggling with form, crossing things out"
[15-30s] "Our AI walks you through step-by-step with plain English explanations"
[30-45s] "Your form is auto-filled, checked for errors, then ready to mail"
[45-60s] "Get started with 3 free questions. Just $29 for the full form."
[CTA] "Try 3 Free Questions" button (above fold)
```

### Free Trial Conversion Rates

| Metric | Conservative | Realistic | Optimistic |
|---|---|---|---|
| **Visitors → Start Free Trial** | 20% | 30% | 45% |
| **Start Trial → Complete 3 Questions** | 60% | 75% | 85% |
| **Complete 3 Qs → Hit Paywall** | 100% | 100% | 100% |
| **Hit Paywall → See Conversion Page** | 85% | 90% | 95% |
| **Blended Free Trial Engagement** | 10.2% | 20.3% | 36.2% |

**Interpretation:**
- Out of 5,000 monthly visitors, 1,010-1,810 will try the free 3 questions
- Of those, ~85-90% will encounter the paywall and see the $29 offer

---

## Stage 3: CONVERSION - Free to Paid

### Paywall Conversion Rates

**The Critical Moment:** After completing 3 free questions, user sees paywall offering $29 option.

| Scenario | Paywall CR | Visitors → Trial | Trial → Paid | Cumulative Funnel |
|---|---|---|---|---|
| **Conservative** | 8% | 20% | 8% | **1.6%** |
| **Realistic** | 12% | 30% | 12% | **3.6%** |
| **Optimistic** | 18% | 45% | 18% | **8.1%** |

### Detailed Paywall Conversion Flow (1,000 visitors baseline)

#### Conservative Scenario
```
5,000 visitors/month
  → 1,000 land on page (20% entrance rate)
    → 600 try 3 free questions (60% engagement)
      → 510 hit paywall (85% completion)
        → 40 convert to $29 (8% CR)
          = 40 customers × $29 = $1,160/month (single form)
```

#### Realistic Scenario
```
5,000 visitors/month
  → 1,500 land on page (30% entrance rate)
    → 1,125 try 3 free questions (75% engagement)
      → 1,012 hit paywall (90% completion)
        → 121 convert to $29 (12% CR)
          = 121 customers × $29 = $3,509/month (single form)
```

#### Optimistic Scenario
```
5,000 visitors/month
  → 2,250 land on page (45% entrance rate)
    → 1,912 try 3 free questions (85% engagement)
      → 1,817 hit paywall (95% completion)
        → 327 convert to $29 (18% CR)
          = 327 customers × $29 = $9,483/month (single form)
```

### Why These Conversion Rates?

**Realistic paywall CR (12%) is justified by:**

1. **Low friction:** No email required to start free trial
2. **Immediate value:** Users see AI explanations they couldn't get elsewhere
3. **Pain point:** Immigration forms are genuinely confusing; $29 is cheap insurance
4. **Urgency:** People searching at 2am are filing *today*, willing to impulse-buy
5. **Comparison:** $29 vs. $300/hr lawyer = obvious value prop
6. **Trust signals:** USCIS official form shown, warning messages, expert explanations

**Comparison to SaaS benchmarks:**
- Typical SaaS paywall CR (email → paid): 2-5%
- Immigration form tool paywall CR: 12% (2.4x higher because of pain-driven behavior)

---

## Stage 4: FULL FUNNEL MODEL - 1,000 Visitors Scenario

### Baseline: 1,000 Monthly Visitors

**Starting from:** 1,000 organic visitors/month (Month 3+ steady state)

| Stage | Metric | Conservative | Realistic | Optimistic |
|---|---|---|---|---|
| **Awareness** | Visitors | 1,000 | 1,000 | 1,000 |
| **Trial** | Free trial starts (20-45%) | 200 | 300 | 450 |
| **Trial Completion** | Completes 3 Qs (60-85%) | 120 | 225 | 382 |
| **Paywall** | Sees paywall (85-95%) | 102 | 202 | 363 |
| **Conversion** | Converts to $29 (8-18%) | **8** | **24** | **65** |
| **Funnel CR** | Visitor → Paid | **0.8%** | **2.4%** | **6.5%** |

### Scaled: 5,000 Monthly Visitors (Month 4+)

| Stage | Conservative | Realistic | Optimistic |
|---|---|---|---|
| **Visitors** | 5,000 | 5,000 | 5,000 |
| **Free Trial Starts** | 1,000 | 1,500 | 2,250 |
| **Completes 3 Qs** | 600 | 1,125 | 1,912 |
| **Sees Paywall** | 510 | 1,012 | 1,817 |
| **Converts to Paid** | **40** | **121** | **327** |
| **Monthly Revenue** | **$1,160** | **$3,509** | **$9,483** |

---

## SECTION 2: LANDING PAGE OPTIMIZATION

### Should the Landing Page Show a Demo?

**Answer: YES, absolutely.**

#### Demo Impact on Conversion (A/B Test Hypothesis)

| Landing Page Version | Trial Engagement | Paywall CR | Revenue |
|---|---|---|---|
| **No Demo (text only)** | 15% | 8% | $348/1000 visitors |
| **Demo (60-sec video)** | 30% | 12% | $1,044/1000 visitors |
| **Demo + Live Tool Preview** | 35% | 14% | $1,470/1000 visitors |
| **Lift from Demo** | **+100%** | **+50%** | **+200%** |

**Why demo is critical:**
1. **Builds trust:** See the tool works before paying
2. **Reduces friction:** Shows it's not a complicated wizard
3. **Answers objections:** "What exactly do I get for $29?"
4. **Mobile-friendly:** Short video (60s) loads fast, works offline-ish
5. **Social proof:** Shows actual form + AI explanations (not just marketing copy)

#### Recommended Landing Page Structure

```
Above Fold (0-3 seconds, no scroll):
  [Logo] Immigration Form AI
  [Headline] "Fill out your I-130 form correctly in 15 minutes"
  [Subheadline] "AI-powered guidance, auto-filled PDF, error checking—$29"
  [CTA Button] "Try 3 Free Questions"
  [Demo Video Thumbnail] "See how it works (60 sec)"

Mid-Scroll (3-10 seconds):
  [Play Button Demo Video]
  [Text] "What's Included"
  - 60-question guided Q&A
  - Auto-filled PDF ready to mail
  - 40+ error checks (catches RFE risks)
  - Document checklist
  - Filing instructions

Lower-Scroll (10-20 seconds):
  [Social Proof]
  - "Used by 2,000+ filers"
  - "5-star reviews on Reddit"
  - Testimonials (short text quotes)

Footer:
  [CTA] "Get Started with 3 Free Questions"
  [Trust Signals] "Secure payment" + Stripe logo
  [FAQ] "What do I get?" "How long?" "Refund policy?"
```

#### Landing Page Copy Optimization

**Headline A/B Test Ideas:**

| Version | Focus | Expected Lift |
|---|---|---|
| "Fill out your I-130 form correctly in 15 minutes" | Speed + accuracy | Baseline |
| "AI immigration lawyer for $29" | Comparison to lawyer | +15% |
| "Avoid the $2,000 RFE mistake" | Pain point | +25% |
| "Get your I-130 approved faster" | Outcome | +10% |

**Recommended:** Lead with pain ("Avoid the $2,000 RFE mistake"), then pivot to solution ("AI-powered I-130 guide, $29").

---

## SECTION 3: FREE TIER STRATEGY - Should First 3 Questions Be Free?

### The Strategic Question

**"Should we give away the first 3 questions for free, or charge $29 upfront?"**

#### Option A: 3 Free Questions (Recommended)

**Pros:**
- Reduces friction (no payment before trial)
- Builds trust (user sees AI quality before paying)
- Increases paywall CR (from 4% to 12%)
- Standard SaaS freemium model

**Cons:**
- ~5% of users will stop after free questions
- Takes more infrastructure (scaling free usage)

**Paywall Economics:**
```
100 free trials
  → 85 hit paywall (15% dropout)
    → 10 convert (12% CR on paywall)
    = 10% funnel conversion
```

#### Option B: $29 Upfront (No Free Tier)

**Pros:**
- Simpler infrastructure (no freemium scaling)
- Only serious buyers purchase

**Cons:**
- Conversion rate drops to 2-3% (no trial)
- Cold traffic struggles (unproven tool)
- 50x fewer conversion events (less learning)

**Paywall Economics:**
```
100 visitors
  → 2-3 convert (2-3% CR)
    = 2-3% funnel conversion
```

#### Decision Matrix

| Factor | 3 Free Qs | $29 Upfront |
|---|---|---|
| **Funnel conversion** | 2.4% | 0.5% |
| **Monthly revenue (5K visitors)** | $3,509 | $726 |
| **User trust** | High | Low |
| **Learning velocity** | Fast | Slow |
| **CAC recovery** | <1 month | 2-3 months |
| **Recommended?** | ✓ YES | ✗ No |

**Recommendation: Launch with 3 free questions.** This maximizes conversion while building trust.

---

## SECTION 4: CONVERSION RATE OPTIMIZATION TACTICS

### Pre-Launch Optimization (Weeks 1-6)

#### Tactic 1: Landing Page Copy Testing
- **What:** Test 3-5 headline variations using A/B testing
- **Expected lift:** +15-25% conversion
- **Timeline:** Implement after first 100 visitors
- **Tool:** Google Optimize, VWO, or Unbounce

**Headline Variations to Test:**
1. "Fill I-130 correctly in 15 minutes" (speed)
2. "Avoid the $2,000 RFE mistake" (fear)
3. "AI immigration lawyer for $29" (comparison)
4. "Get your I-130 approved 2x faster" (outcome)
5. "USCIS form guidance + auto-filled PDF" (feature)

#### Tactic 2: Demo Video Quality
- **What:** Invest in 60-90 second screenshare demo showing real Q&A
- **Expected lift:** +50-100% trial engagement
- **Timeline:** Before launch week
- **Cost:** $500-1,000 (Fiverr freelancer) or free (record yourself with ScreenFlow)

**Demo Script (60 seconds):**
```
[0-5s] Problem: "I'm filing I-130 but the form is confusing..."
[5-15s] Show form PDF + person crossing things out
[15-30s] Show AI tool answering "Who are you filing for?" with explanation
[30-40s] Show PDF auto-filling with answers
[40-50s] Show consistency checker flagging potential error
[50-60s] Show completed PDF ready to download
[60s] CTA: "Try it free for the first 3 questions"
```

#### Tactic 3: Trust Signals on Landing Page
- **What:** Add social proof, security badges, testimonials early
- **Expected lift:** +10-20% conversion
- **Timeline:** Before launch
- **What to show:**
  - "Used by 2,000+ immigrants" (or real number)
  - Stripe/PayPal security badges
  - 5-star average rating (collect from beta users)
  - Quote from lawyer: "Finally, an affordable way to check your form"
  - USCIS official form screenshot (shows legitimacy)

#### Tactic 4: Mobile Optimization
- **What:** Ensure landing page + free tool work flawlessly on mobile
- **Expected lift:** +15-30% (60% of traffic is mobile)
- **Timeline:** Before launch
- **Key elements:**
  - CTA button must be thumb-friendly (large, above fold on mobile)
  - Form inputs must be mobile-friendly (auto-focus, auto-fill)
  - Video must auto-play with sound-off option
  - No horizontal scroll

#### Tactic 5: FAQ Section (Conversion Enabler)
- **What:** Address top 10 objections on landing page
- **Expected lift:** +5-10% conversion (reduces "abandonment due to questions")
- **Top FAQ questions:**
  - "Is this legal advice?" (No, but it catches common mistakes)
  - "What if I have a unique situation?" (You can export and get lawyer review)
  - "Can I get a refund?" (Yes, within 30 days)
  - "How long does it take?" (15-20 minutes)
  - "Will this get my form approved?" (Not guaranteed, but prevents RFEs)
  - "Is my data secure?" (Yes, SSL, no storage by default)

---

### Post-Launch Optimization (Weeks 7-12)

#### Tactic 6: Exit-Intent Popup
- **What:** When user about to leave without converting, offer discount
- **Expected lift:** +3-5% recovery
- **Timeline:** Week 2 post-launch
- **Copy:** "Wait! Complete your I-130 for just $24.50 today" (10% discount)
- **Tool:** Sumo.com, ConvertKit, or custom JS

#### Tactic 7: Email Capture for Retargeting
- **What:** Collect email from free trial users who don't convert
- **Expected lift:** +10-20% (retargeting via email)
- **Timeline:** Week 1 post-launch
- **Sequence:**
  - Email 1 (immediate): "You abandoned your I-130—complete it for $29"
  - Email 2 (day 2): "This catches the #1 RFE mistake"
  - Email 3 (day 4): "See how it works (testimonial + video)"
  - Email 4 (day 7): "Last chance—$24 this week only"

#### Tactic 8: Progressive Disclosure
- **What:** Show more features/benefits as user scrolls landing page
- **Expected lift:** +5-15% (keeps attention, builds case)
- **Timeline:** Week 1 post-launch
- **Structure:**
  - Above fold: Problem + demo + CTA
  - Scroll 1: What's included (list of features)
  - Scroll 2: Testimonials + social proof
  - Scroll 3: How it works (step-by-step screenshots)
  - Scroll 4: FAQ section
  - Scroll 5: Final CTA + guarantee

#### Tactic 9: First-Time Buyer Offer
- **What:** $5 discount for first purchase ($24 instead of $29)
- **Expected lift:** +8-15% conversion (removes price objection)
- **Timeline:** Weeks 1-8 only (launch period)
- **Messaging:** "Launch offer: $24 this week only"
- **Profit impact:** -$5 per unit, but 10x unit volume increase = net positive

#### Tactic 10: Testimonial Page
- **What:** Dedicated page with 10-20 user testimonials + case studies
- **Expected lift:** +10-20% (builds trust for skeptics)
- **Timeline:** Week 2 post-launch
- **What to collect:**
  - Short quote (1-2 sentences)
  - Full name + profession (credibility)
  - Photo (optional, adds trust)
  - How many questions answered, how many minutes took

**Sample Testimonial:**
```
"I was terrified of filing I-130 wrong. This tool walked me through
every field with plain English. My form was approved in 8 months."
— Maria Rodriguez, Nurse, Texas
```

---

## SECTION 5: FUNNEL MODEL - EXPECTED MONTHLY RESULTS

### Scenario 1: Conservative Case (Month 3)

```
Monthly Organic Visitors: 1,000
  ↓
Try Free Tier (20% → 200 visitors)
  ↓
Complete 3 Questions (60% → 120)
  ↓
See Paywall (85% → 102)
  ↓
Convert to Paid (8% → 8 customers)

Monthly Revenue: 8 × $29 = $232
Monthly Costs: $300 (hosting + APIs)
Monthly Profit: -$68 (not profitable yet)
```

### Scenario 2: Realistic Case (Month 4, After Optimization)

```
Monthly Organic Visitors: 3,000 (growing 20% MoM)
  ↓
Try Free Tier (30% → 900 visitors)
  ↓
Complete 3 Questions (75% → 675)
  ↓
See Paywall (90% → 607)
  ↓
Convert to Paid (12% → 73 customers)

Monthly Revenue: 73 × $29 = $2,117
Monthly Costs: $400 (hosting + APIs)
Monthly Profit: $1,717
```

### Scenario 3: Optimistic Case (Month 6, Multiple Forms Live)

```
Monthly Organic Visitors: 5,000 (strong SEO)
  ↓
Try Free Tier (45% → 2,250 visitors)
  ↓
Complete 3 Questions (85% → 1,912)
  ↓
See Paywall (95% → 1,817)
  ↓
Convert to Paid (18% → 327 customers)

Monthly Revenue: 327 × $29 = $9,483
[Plus upsells: 30% upgrade to I-485 ($29 more)]
Total with Upsells: 327 × $42 = $13,734
Monthly Costs: $600 (hosting + APIs for multiple forms)
Monthly Profit: $13,134
```

---

## SECTION 6: UPSELL & BUNDLE STRATEGY

### Upsell Funnel (Converting $29 → Higher Revenue)

#### After I-130 Purchase ($29)

**Immediate upsell (shown after payment):**
- "88% of I-130 filers also need I-485 (Green Card Application) → $29 extra"
- Expected uptake: 25-35%
- Revenue per customer: $29 + $8 (avg upsell) = $37

**Week-based upsell (email sequence):**
- Week 1: "Your I-130 was approved? Here's the next step (I-485 guide)"
- Week 4: "Many clients use our 3-form bundle (save $18)"

#### Bundle Strategy

**Current pricing (V1.1+):**
| Tier | Price | Discount | Forms |
|---|---|---|---|
| Single Form | $29 | - | I-130, I-485, N-400, DS-160 (pick one) |
| 3-Form Bundle | $69 | -21% | I-130 + I-485 + N-400 (most popular) |
| Unlimited Annual | $99 | -66% | All forms current + future |

**Bundle conversion impact:**
```
100 customers buying single form at $29 = $2,900
  → 30% see bundle offer (after first purchase)
    → 50% convert to $69 bundle (+$40 extra per customer)
    = $2,900 + (30 × $40) = $4,100 revenue
    = 41% revenue lift from bundling
```

---

## SECTION 7: KEY METRICS & DASHBOARDS

### Core Funnel Metrics to Track

| Metric | Current | Target (Month 4) | Optimization Tactic |
|---|---|---|---|
| **Landing Page CR** | 0.5% | 2-3% | Copy testing, demo, trust signals |
| **Free Trial Engagement** | 20% | 35% | Improve UX, mobile optimization |
| **Paywall CR** | 8% | 12-15% | Testimonials, benefit highlights |
| **End-to-End Funnel CR** | 0.8% | 2.4-3.5% | Optimization across all stages |
| **AVG Order Value** | $29 | $42 (with upsells) | Bundle offers, reminder emails |
| **CAC** | $0 | $0-2 | Organic search only |
| **LTV** | $87 (1-time + 12% CR) | $240+ | Repeat purchases, subscriptions |

### Analytics Dashboard (Weekly Review)

**Metrics to monitor daily/weekly:**
1. **Visitors:** Total unique visitors to site
2. **Free trial starts:** # who clicked "Try Free"
3. **Free trial completion:** # who completed 3 questions
4. **Paywall reach:** # who saw $29 offer
5. **Conversions:** # who paid
6. **Revenue:** $ total
7. **Churn/Refunds:** # who requested refund
8. **Time on site:** Avg time (should be 5+ minutes)
9. **Mobile vs. Desktop:** Conversion rate by device
10. **Traffic source:** Which keyword/referrer converts best

---

## SECTION 8: OPTIMIZATION ROADMAP (Months 1-6)

### Week 1-6: Pre-Launch Optimization

- [ ] Design landing page with demo video
- [ ] Write 3-5 headline variations for A/B testing
- [ ] Create 60-sec demo video showing real Q&A
- [ ] Add trust signals (badges, testimonials from beta users)
- [ ] Mobile-optimize landing page + tool
- [ ] Write FAQ addressing top 10 objections
- [ ] Set up analytics (Google Analytics + Mixpanel)
- [ ] Implement Stripe payment flow
- [ ] Test full funnel end-to-end (5+ test purchases)

### Week 7-8: Launch & Quick Wins

- [ ] Launch landing page + product
- [ ] Activate Product Hunt (day 1)
- [ ] Post to Reddit/Facebook groups
- [ ] Start A/B testing headlines (split traffic 50/50)
- [ ] Collect first 10 testimonials (incentivize with discount)
- [ ] Monitor daily funnel metrics
- [ ] Fix any bugs blocking conversion

### Week 9-12: Post-Launch Optimization

- [ ] Analyze which headline wins (→ run winner longer)
- [ ] Implement exit-intent popup with discount offer
- [ ] Set up email retargeting sequence (4 emails)
- [ ] Create testimonial page (10+ quotes)
- [ ] Test landing page copy variations (benefit vs. problem-focused)
- [ ] Optimize mobile CTA button placement
- [ ] Monitor email retargeting performance
- [ ] Prepare I-485 form for Month 2 launch

### Month 4-6: Scale Optimization

- [ ] Expand to 2-3 forms (I-485, N-400)
- [ ] Implement bundle pricing strategy
- [ ] Set up automated upsell sequences
- [ ] Run split test: $29 vs. $24 pricing
- [ ] A/B test demo video length (60s vs. 90s)
- [ ] Optimize for high-intent keywords (long-tail search)
- [ ] Launch multilingual landing page (Spanish)
- [ ] Expand testimonial collection (goal: 20+ quotes)

---

## SECTION 9: REALISTIC EXPECTATIONS BY MONTH

### Funnel Performance Across 6 Months

| Month | Visitors | Trial Start | Trial Completion | Paywall Reach | Conversions | Revenue | Status |
|---|---|---|---|---|---|---|---|
| **Month 2 (Launch)** | 500 | 75 (15%) | 36 (48%) | 28 (78%) | 2 (7%) | $58 | Testing |
| **Month 3** | 1,000 | 200 (20%) | 120 (60%) | 102 (85%) | 8 (8%) | $232 | Baseline |
| **Month 4** | 2,000 | 500 (25%) | 350 (70%) | 315 (90%) | 38 (12%) | $1,102 | Optimizing |
| **Month 5** | 3,500 | 1,050 (30%) | 788 (75%) | 709 (90%) | 85 (12%) | $2,465 | Scaling |
| **Month 6** | 5,000 | 1,500 (30%) | 1,125 (75%) | 1,012 (90%) | 121 (12%) | $3,509 | Mature |

**Key observations:**
- Month 2-3: Funnel under-performs (low traffic, optimization needed)
- Month 4-5: Optimization pays off (copy tests, video, testimonials)
- Month 6+: Steady-state with 2.4% funnel conversion

---

## SECTION 10: FINAL RECOMMENDATIONS

### Summary: What to Optimize First?

**Rank these optimizations by ROI (Effort vs. Impact):**

| Priority | Tactic | Effort | Impact | ROI | When |
|---|---|---|---|---|---|
| **1** | Demo video on landing page | Low | High (2x trial engagement) | 10x | Week 6 |
| **2** | Landing page copy A/B test | Low | Medium (+15-25% CR) | 8x | Week 7 |
| **3** | Trust signals (testimonials, badges) | Medium | Medium (+10-20% CR) | 5x | Week 8 |
| **4** | Mobile optimization | Medium | Medium (+15-30% CR) | 6x | Week 6 |
| **5** | Email retargeting sequence | Medium | Low (+5-10% recovery) | 3x | Week 9 |
| **6** | Exit-intent popup + discount | Low | Low (+3-5% recovery) | 2x | Week 9 |
| **7** | FAQ section expansion | Low | Low (+5-10% CR) | 2x | Week 7 |
| **8** | Testimonial page | Medium | Low (+5-10% CR) | 2x | Week 10 |

**Quick-win timeline:**
- Week 6: Record demo video
- Week 7: A/B test 3 headlines
- Week 7-8: Add testimonials + social proof
- Week 8+: Monitor results, iterate based on data

### Launch with Confidence

**The realistic funnel for Month 4+ is:**
```
1,000 visitors/month
  → 30% try free (300)
    → 75% complete 3 Qs (225)
      → 90% see paywall (202)
        → 12% convert ($29) (24 customers)
          = 2.4% end-to-end conversion
          = $696/month from organic traffic
```

**This scales to:**
```
5,000 organic visitors/month (by Month 6)
  → 2.4% conversion
    = 120 customers × $29 = $3,480/month
    + 30% upsell to second form ($42 avg) = $5,040/month
    - $400 costs = $4,640/month profit
```

**Conclusion:** The funnel works. Start with the demo video and copy tests (highest ROI). Track metrics daily. Iterate based on data.

---

## APPENDIX: A/B Testing Roadmap

### Month 1-2: Critical Tests (Run in Parallel)

**Test 1: Demo Video**
- Control: No demo (text only)
- Variant: 60-sec demo video
- Metric: Trial engagement rate
- Expected result: +50-100% improvement
- Duration: Run for first 500 visitors

**Test 2: Headline Copy**
- Control: "Fill I-130 form correctly in 15 minutes"
- Variant A: "Avoid the $2,000 RFE mistake"
- Variant B: "AI immigration lawyer for $29"
- Metric: Landing page CR to trial
- Expected result: +15-25% improvement
- Duration: Run full month

**Test 3: CTA Button Color**
- Control: Blue button "Try Free Questions"
- Variant: Green button "Start Now"
- Metric: CTR on button
- Expected result: +5-10% improvement
- Duration: Run 2 weeks

### Month 3+: Ongoing Optimization

- Run continuous headline tests (update winner monthly)
- Test discount offers (10% off, time-limited)
- Test email sequences (1 email vs. 4-email sequence)
- Test demo length (60s vs. 90s vs. 2m)

---

## END DOCUMENT

**Next Step:** Build landing page with demo video first. That's your highest-impact optimization. Launch week 6, then optimize based on real user data.
