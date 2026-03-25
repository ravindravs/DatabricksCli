# Immigration Form Assistant - MVP Quick Reference

## One-Pager Summary

### The Product
**AI assistant that guides users through I-130 form filling with automated Q&A, PDF generation, and error checking.**

### The Model
- **V1 Launch:** 1 form (I-130 only)
- **Price:** $29 one-time
- **Development Time:** 6 weeks
- **Target Year 1 MRR:** $8-35K

---

## Why This Works

| Factor | Why It Wins |
|---|---|
| **Search Volume** | I-130 = 80K+ monthly searches ("How to fill I-130") |
| **Willingness to Pay** | Immigrants pay $300+/hr for lawyers; $29 is instant buy |
| **No Marketing Needed** | People Google at 2am in panic mode—organic search captures 100% of demand |
| **Competitive Gap** | SimpleCitizen costs $500+; no AI-native tool under $40 exists |
| **Build Time** | 6 weeks is achievable; PDF + LLM are solved problems |

---

## V1 Feature Set (Must-Have)

1. **Interactive Q&A** (60 questions, branching logic, AI explanations)
2. **PDF Auto-Fill** (generates completed form ready to mail)
3. **Consistency Checker** (flags 40+ common RFE risks)
4. **Summary + Checklist** (document list, filing instructions, timeline)
5. **Offline Support** (PWA for spotty internet areas)

**Total development: 5-6 weeks**

---

## Why 1 Form, Not 10

| Metric | 1 Form | 10 Forms |
|---|---|---|
| Time to revenue | 6 weeks | 16+ weeks |
| Quality | Excellent | Poor |
| SEO depth | Deep (100 pages/form) | Shallow |
| Competitive moat | High | None |
| Expansion cost | +2 weeks/form | - |

**Decision:** Ship 1 form, expand 1 per month. By month 4, you have 3 forms (80% of market).

---

## Form Expansion Roadmap

| Month | Form | Rationale |
|---|---|---|
| 1 | I-130 (Immediate Relative) | 80K+ searches, highest emotional urgency |
| 2 | I-485 (Green Card Application) | Natural follow-on, 35K+ searches |
| 3 | N-400 (Citizenship) | Third most popular, 20K+ searches |
| 4+ | DS-160 or I-131/I-765 | Based on user demand data |

---

## Pricing Strategy

### V1.0 (Weeks 1-6)
- **$29 one-time** for I-130 form completion

### V1.1+ (Week 7 onward)
- **$29/form** (single form)
- **$69 bundle** (I-130 + I-485 + N-400)
- **$99/year** (unlimited forms)

### Why NOT subscription at V1:
- Users file once per 5-10 years
- One-time feels right for one-time use
- Subscription = predatory perception

---

## The $29 Scope

**Users get:**
- ✓ 60-question Q&A through I-130
- ✓ Filled PDF ready to mail to USCIS
- ✓ One-page summary of all answers
- ✓ Document checklist
- ✓ Filing instructions + timeline
- ✓ Risk flagging (40+ error checks)

**Users DON'T get (post-launch):**
- ✗ Live lawyer review
- ✗ Document uploads
- ✗ Multiple forms (V1.1 adds these)
- ✗ Multi-language (V1.1 adds Spanish)

---

## Weekly Breakdown (6 Weeks to Launch)

| Week | What | Deliverable |
|---|---|---|
| 1 | Foundation + form analysis | Field map + question taxonomy in JSON |
| 2 | Q&A engine core | 60-question flow with AI guidance |
| 3 | PDF generation | Download filled PDF from Q&A answers |
| 4 | Validation rules | Consistency checker with 40+ error rules |
| 5 | Summary + checklist | One-pager + filing instructions |
| 6 | Offline + deploy | Live product, PWA support, Stripe integration |

---

## Launch Checklist (Week 6)

Must have:
- [ ] Working PDF export
- [ ] Consistency checker (flags mistakes)
- [ ] Legal disclaimer
- [ ] Stripe payments
- [ ] SSL certificate

Nice to have:
- [ ] 5 beta testimonials
- [ ] 10 SEO blog posts
- [ ] Product Hunt post
- [ ] Analytics setup

---

## Go-to-Market (Weeks 1-8)

### Primary Channel: Organic Search
**Keywords (40-80K monthly searches each):**
- "How to fill out I-130"
- "I-130 instructions"
- "I-130 form step by step"
- "I-130 common mistakes"

**Content plan:**
- Week 7-8: 10 blog posts (2,000 words each, SEO-optimized)
- Each blog post links to product
- Build authority on "I-130 form guide"

### Secondary Channel: Word-of-Mouth
- Product Hunt launch (target: top 20)
- Reddit (r/ImmigrationLaw, r/USCIS, r/VisaApplications)
- Facebook groups (16 major immigration groups)
- WhatsApp/WeChat diaspora communities

### NO paid marketing (V1)
- Organic search only
- Word-of-mouth only
- Community posts (value first, mention second)

---

## Revenue Projections

### Conservative Year 1
| Month | Users | MRR | Notes |
|---|---|---|---|
| 1 | - | $0 | Building |
| 2 | 10 | $500 | Launch |
| 3-6 | 50-75 | $1.5-2.5K | Organic SEO ramping |
| 7-12 | 100-200 | $4-8K | 3-4 forms live, bundles |
| **Year 1 Total** | - | **$20-30K** | - |

### Optimistic Year 1
| Month | Users | MRR | Notes |
|---|---|---|---|
| 1 | - | $0 | Building |
| 2 | 30 | $1.5K | Strong launch |
| 3-6 | 150-250 | $5-8K | SEO flying |
| 7-12 | 300-500 | $12-18K | Multiple forms, bundle effect |
| **Year 1 Total** | - | **$70-100K** | - |

### Breakeven Timeline
- Server costs: ~$900/month (backend, CDN, domain)
- At 250 monthly users × $29 × 1.5 forms = $10.9K MRR
- **Breakeven: Month 9-10**

---

## Success Metrics

### Week 6 (Launch)
- ✓ Product works without critical bugs
- ✓ 5-10 beta users provide positive feedback
- ✓ Can point to 3+ specific RFE risks it prevents

### Month 2
- ✓ $500+ monthly revenue
- ✓ Product Hunt top 20
- ✓ 5+ organic traffic sources
- ✓ Positive community feedback

### Month 4
- ✓ $2-3K monthly revenue
- ✓ I-485 form live
- ✓ Ranking on 10+ keywords
- ✓ 20 blog posts live
- ✓ 10+ testimonials

### Month 6
- ✓ $4K+ monthly revenue
- ✓ 3-4 forms in system
- ✓ Spanish + Tagalog support
- ✓ "Immigration form assistant" is your brand

### Month 12
- ✓ $8-15K monthly revenue
- ✓ 200-300 monthly active users
- ✓ 5+ forms in system
- ✓ Plan for next expansion

---

## Risks & Mitigations

| Risk | Mitigation |
|---|---|
| USCIS form changes | Monitor monthly; version control all maps |
| PDF generation breaks | Test with real USCIS tools; manual export backup |
| Legal liability | Add disclaimer; link to lawyers; track all rules to source |
| Competitor launches | Ship in 6 weeks (not 12); focus on depth (5 forms) |
| Low initial traction | Pre-build audience with blog; launch with testimonials |
| Users confused by form | Write in 6th-grade English; user test with non-tech users |

---

## NOT Building in V1

- Video tutorials (too expensive)
- Live lawyer chat (overhead kills margins)
- Document upload/scanning (edge cases)
- Native mobile app (PWA sufficient)
- USCIS payment processing (too complex)
- Multiple languages (English only, Spanish in V1.1)
- Case status tracking API (too fragile)
- eSignature/notary matching (legal liability)

---

## Next Step

**Week 1 Tasks:**
1. Download official I-130 form PDF from USCIS.gov
2. Extract all 50+ fields from form
3. Create branching question taxonomy (JSON)
4. Set up database schema
5. Define GPT system prompt for Q&A guidance

**By end of Week 1:** You should have a complete field map + decision tree ready to code against.

---

## The Big Picture

This is a **6-week sprint to revenue** using:
- 1 high-demand form (not 10 mediocre forms)
- 5 must-have features (not 50 nice-to-haves)
- $29 price point (not $99-500 subscription)
- Organic search only (no paid marketing)
- 80% of users discover you through panic Google searches

**If you execute this in 6 weeks, you'll have:**
- A live product generating revenue
- Organic traffic from 80K+ monthly searches
- A moat (you're first-to-market with AI-native tool)
- A clear expansion roadmap (1 form/month)
- A path to $8-35K MRR in Year 1

**The risk is low because:**
- PDF filling + LLM prompting are solved problems
- Immigration forms are unchanging (I-130 has been same for 20 years)
- Demand is guaranteed (people need this, they search for it, they'll pay)
- Competition is either too expensive ($500+) or too generic (not immigration-specific)
