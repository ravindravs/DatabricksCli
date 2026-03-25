# AI Immigration Form Assistant - MVP Definition

**Product:** AI-powered guide for completing U.S. immigration forms with automated Q&A, PDF auto-fill, consistency checking, and risk flagging.

**Thesis:** Immigration seekers search for form guidance at 2am in panic mode (I-130 has 80K+ monthly searches). They'll pay $29 instantly for a tool that reduces legal risk, saves time, and works offline. No marketing needed—organic search captures all demand.

---

## SECTION 1: MVP SCOPE & FORM SELECTION

### Why 1 Form, Not 10

**V1 Launch Strategy: 1 Form Only (I-130)**

| Dimension | 1 Form | 3 Forms | 10 Forms |
|---|---|---|---|
| **Time to launch** | 6 weeks | 10-12 weeks | 16+ weeks |
| **First revenue** | Week 7 | Week 12 | Week 17+ |
| **Quality per form** | Excellent | Good | Poor |
| **User trust** | "Specialists" | "Generalists" | "Incomplete" |
| **SEO authority** | Deep (100+ pages/form) | Shallow | Diluted |
| **Competitive moat** | High (1st-to-market depth) | Medium | None |
| **Cost to expand** | +2 weeks/form | - | - |

**Decision:** Launch with **1 form (I-130 Immediate Relative Petition)** because:
- I-130 is the highest-intent search (family sponsorship = emotional urgency)
- 80K+ monthly searches ("how to fill I-130", "I-130 instructions", "I-130 examples")
- Highest willingness to pay (sponsoring family members)
- Small enough to complete in 6 weeks with excellence
- Easy to expand to I-485, N-400, DS-160 one per month

### Form Selection Roadmap (Months 1-8)

**Month 1 (V1 Launch):** I-130 (Immediate Relative Petition)
- Most popular form (family sponsorship)
- Highest search volume + emotional urgency
- Straightforward logic (relationships, basic biographical data)

**Month 2-3 (V1.1-V1.2):** I-485 (Green Card Application)
- Natural follow-on after I-130 approved
- Users already understand the flow from V1
- Second-highest search volume (35K+ monthly searches)
- Higher complexity (security checks, medical exams) = higher perceived value

**Month 4 (V1.3):** N-400 (Naturalization/Citizenship Application)
- Third core form
- Evergreen demand (citizenship deadlines)
- Appeals to established immigrants (different audience from I-130)
- 20K+ monthly searches

**Months 5-8:** DS-160 (Visa Application) or I-131/I-765 (Work/Travel)
- Based on user demand data
- Expand based on which form drives highest engagement in Months 2-4

**Why this expansion pace?**
- Each form adds ~1-2 weeks of development (templates exist, pattern known)
- Stagger launches to maintain quality + build case studies
- By Month 4, you have 3 forms = $29 × 3 = $87 per power user (upsell)
- By Month 8, you have 4-5 forms covering 70% of immigration search volume

---

## SECTION 2: MINIMUM VIABLE PRODUCT FEATURE SET

### Must-Have Features (V1.0 - Weeks 1-6)

These are the minimal features needed to capture payment on day 1.

#### 1. **Interactive Q&A Engine** (Weeks 1-2)
- **What:** Step-by-step form guidance powered by GPT-4 or Claude
- **Why:** Replaces $300/hr lawyer consultation
- **Scope:**
  - 40-60 context-aware questions for I-130 form
  - Branching logic (e.g., "spouse" path different from "child" path)
  - Plain-language explanations (read at 6th-grade level)
  - Examples for every field ("Your relationship to petitioner: Usually 'Father', 'Daughter', etc.")
  - Estimated time shown upfront (typically 15-20 minutes for I-130)

- **Implementation:**
  - Static JSON schema with 60 form fields
  - GPT-4 system prompt trained on USCIS form instructions + lawyer advice
  - Temperature=0.3 (factual, not creative)
  - Show what goes in each PDF field as you answer questions

#### 2. **PDF Auto-Fill** (Weeks 2-3)
- **What:** Generates filled USCIS-compatible PDF from Q&A answers
- **Why:** Error prevention + time savings (otherwise 45 min manual entry)
- **Scope:**
  - Download pre-filled I-130 PDF ready to print/file
  - Font compliance with USCIS specs (Courier New, 10pt)
  - Checkbox population (yes/no fields)
  - Barcode fields left blank (USCIS scans these)
  - QR-code verification (let user scan to confirm fields)

- **Implementation:**
  - Use PyPDF2 or pdfrw for PDF manipulation
  - Template: Download official USCIS I-130 form PDF
  - Map Q&A answers to form field coordinates via visual inspection
  - Export ready-to-file version + summary page

#### 3. **Consistency Checker & Risk Flags** (Weeks 3-4)
- **What:** AI-powered validation that flags logical inconsistencies and legal risks
- **Why:** Catches 80% of RFEs (Requests for Evidence) before filing
- **Scope:**
  - Date validation (visa expiry date must be after filing date)
  - Relationship consistency (if filed "spouse", ensure marriage date is within last 2 years for conditional green card logic)
  - Citizenship/visa status conflicts ("Can't file I-130 if applicant is not US citizen")
  - Name spelling consistency (middle name can't change between forms)
  - Address conflicts (multiple conflicting addresses across fields)
  - Income thresholds (if co-sponsor, check Affidavit of Support income minimum)

- **Ruleset:** ~40 hardcoded validation rules + GPT-powered semantic checks
- **UI:** Green checkmark (pass), Yellow warning (review), Red error (must fix)

#### 4. **Summary & Export** (Weeks 4-5)
- **What:** One-page summary of all answers + filed checklist
- **Why:** Users need a paper trail + next-step guidance
- **Scope:**
  - Plain-English summary of everything entered
  - Required documents checklist (birth certificate, marriage license, police clearance, medical exam, etc.)
  - Signature line (for wet signatures on PDF)
  - Filing instructions (where to mail, filing fees, expected processing time)
  - Next-step timeline (when to expect receipt notice, when case status appears online, typical approval timeline)

#### 5. **Offline Capability** (Weeks 5-6)
- **What:** Users can fill forms without internet (for areas with spotty connectivity)
- **Why:** Immigration seekers in developing countries + US refugee centers often have unreliable connectivity
- **Scope:**
  - Progressive Web App (PWA) with offline service worker
  - Sync answers when internet returns
  - Download-to-device capability (entire Q&A engine + rules)
  - No cloud storage required until submission

### Nice-to-Have Features (Post-V1)

**V1.1 (Week 7):**
- Email export of PDF (not just download)
- Estimated case processing timeline by USCIS field office
- Link to relevant USCIS policy documents

**V1.2 (Week 8):**
- Multi-language support (Spanish, Tagalog, Mandarin, Vietnamese)
  - Highest ROI languages based on diaspora size
  - Machine-translated Q&A (not lawyer-reviewed, but good enough)
- Print-to-paper UI optimization

**V2.0 (Months 3-4):**
- Upload existing documents (marriage certificate, birth certificate, police report)
- Document scanner (take phone photo of docs)
- AI-powered form complexity score ("This looks like an RFE risk—here's why")
- Live chat with junior immigration paralegals ($50/mo tier)
- Case tracking (import receipt number, track status via USCIS API)
- Integration with I-485 form (auto-populate shared fields from I-130)

**Post-V2 Nice-to-Haves (Don't build until you have paying users):**
- Video tutorials for each form section
- Community Q&A (crowd-sourced answers from users who approved)
- Payment plan for filing fees ($1,000-1,500 for I-130 filing fees)
- Appointment booking with real immigration attorneys (referral commissions)
- Integration with immigration lawyer networks (premium tier pays lawyers for referrals)

### What NOT to Build in V1

- Video tutorials (too expensive to produce in 6 weeks; write is sufficient)
- Live lawyer chat (overhead kills margins)
- Document upload/scanning (edge cases: blurry photos, unsupported file types)
- Mobile app (PWA is sufficient; native apps take 2x longer)
- Payment processing for USCIS filing fees (too many edge cases; tell user to pay USCIS directly)
- Multiple languages (English-only V1; non-English users pay same price and google-translate; add Spanish in V1.1)
- Case status tracking via USCIS API (too fragile; data model changes yearly)
- Notary matching / eSignature (legal liability; tell users to print and sign)

---

## SECTION 3: PRICING MODEL

### Why $29?

| Benchmark | Price | Target Segment |
|---|---|---|
| **Lawyer consult** | $300-500/hr | Unattainable for most |
| **SimpleCitizen** | $149-349 (full service) | Full-service + lawyer review |
| **Boundless** | $599-3,499 (service) | Full-service + document prep |
| **Rocket Lawyer** | $1-40 depending on plan | Generic legal services |
| **Your sweet spot** | $29 one-time | DIY + AI guidance |

**$29 breaks down to:**
- Eliminates lawyer objection ("I can't afford $300/hr")
- Feels like "cheap insurance" against $2,000 in RFE legal fees
- ~$8 in cloud costs per user (assume 30% margin)
- Easy impulse buy for stressed immigration seekers
- Easy to promote ("Just $29—cheaper than an hour with a lawyer")

### Pricing Tiers

**V1.0 Launch (Weeks 1-6):**

| Tier | Price | What's Included |
|---|---|---|
| **Single Form** | $29 one-time | I-130 form completion + PDF export + checklist |
| | | (No account creation, no tracking) |

**V1.1 (Week 7):**

| Tier | Price | What's Included |
|---|---|---|
| **Single Form** | $29 one-time | One form (I-130, I-485, N-400, or DS-160) |
| **Bundle 3 Forms** | $69 one-time | All 3 most popular forms (I-130 + I-485 + N-400) |
| | | (Save $18 vs. buying separately) |

**V1.2+ (Weeks 8+):**

| Tier | Price | What's Included |
|---|---|---|
| **Pay-per-form** | $29/form | Single form, one-time access |
| **3-Form Bundle** | $69 | I-130 + I-485 + N-400 (most common path) |
| **Unlimited Annual** | $99/year | All forms in system (future proofing) |
| **Premium** | $39/mo | Unlimited forms + email support + document checklist customization |

**Why NOT subscription at V1:**
- Users only file once every 5-10 years
- One-time purchase feels right for one-time use
- Subscription feels predatory ("charging monthly for something you only use once")
- Can always upsell annual pass at checkout

### Revenue Model

**Year 1 Conservative Case:**
- Month 1: $0 (building)
- Month 2 launch: $500/month (10 users × $29 × 1.7 forms/user avg)
- Month 3-6: $1,000-2,000/month (organic SEO ramping)
- Month 7-12: $4,000-8,000/month (3-4 forms live, bundle upsells)
- **Year 1 Total: ~$20-30K**

**Year 1 Optimistic Case:**
- Month 2: $1,500/month (30 users)
- Month 3-6: $5,000-8,000/month (strong SEO)
- Month 7-12: $12,000-18,000/month (bundle effect)
- **Year 1 Total: ~$70-100K**

**Payoff point:**
- At 200 monthly users × $29 × 1.5 forms = $8,700/month MRR = $104K ARR
- Server costs: ~$200/month (backend) + $500/month (CDN) + $200/month (domain/SSL)
- Realistic timeline: Month 9-10 (after 3-4 forms live + word-of-mouth + SEO)

---

## SECTION 4: MVP FEATURE PRIORITY (WEEKS 1-8)

### Week-by-Week Breakdown

#### **Week 1: Foundation**
- [ ] Set up repo, infra, database schema
- [ ] Design I-130 form field map (list all 50+ fields on form)
- [ ] Create question taxonomy (branching decision tree for Q&A flow)
- [ ] Stub out frontend (React or similar) with form container
- [ ] Research + document USCIS form rules (date logic, relationship constraints, income thresholds)

**Deliverable:** Skeleton app with database schema + decision tree in JSON

**Time estimate:** 40-50 hours

---

#### **Week 2: Q&A Engine Core**
- [ ] Build question engine (sequential questions based on branching logic)
- [ ] Implement 60 questions for I-130 (with 6th-grade plain-language explanations)
- [ ] Connect OpenAI/Claude API with system prompt for field-level guidance
- [ ] Build progress bar (X of 60 questions answered)
- [ ] Store answers in session (no persistence yet)
- [ ] Create answer validation (email format for email, date format for dates, etc.)

**Deliverable:** Users can step through 60-question flow, AI explains each field, answers stored in memory

**Time estimate:** 40-50 hours

---

#### **Week 3: PDF Generation**
- [ ] Download official USCIS I-130 form PDF
- [ ] Map form field coordinates (visual inspection + PyPDF coordinate detection)
- [ ] Build PDF rendering logic (fill form fields with Q&A answers)
- [ ] Test PDF export in Acrobat (ensure USCIS compatibility)
- [ ] Add signature field placeholder
- [ ] Implement "Save for later" (download partially filled PDF, come back later)

**Deliverable:** Users complete Q&A → download filled PDF ready to print/file

**Time estimate:** 35-45 hours

---

#### **Week 4: Validation Rules Engine**
- [ ] Build 40+ hardcoded validation rules (date logic, relationship checks, citizenship constraints)
- [ ] Implement GPT-powered semantic validation (catch logical inconsistencies)
- [ ] Design UI for warnings/errors (green/yellow/red status)
- [ ] Create rule documentation (why each rule exists, what it catches)
- [ ] Test against 20+ real user scenarios (edge cases like conditional green card logic)
- [ ] Fix false positives (don't warn on valid cases)

**Deliverable:** Consistency checker flags RFE risks before PDF export

**Time estimate:** 40-50 hours

---

#### **Week 5: Summary + Checklist**
- [ ] Build summary page (plain-English recap of all answers)
- [ ] Create document checklist (birth certificate, marriage license, medical exam, police clearance, etc.)
- [ ] Design filing instructions (address to mail, fees, receipt timeline)
- [ ] Build next-steps timeline (when to expect processing, typical approval time by USCIS center)
- [ ] Implement email export (send PDF + summary to user)
- [ ] Add "Print-friendly" view

**Deliverable:** One-stop reference for everything user entered + what to do next

**Time estimate:** 30-40 hours

---

#### **Week 6: Offline + Deployment**
- [ ] Implement PWA (service worker for offline access)
- [ ] Build offline sync (queue answers locally, sync when online)
- [ ] Deploy to production (Vercel, Netlify, or custom VPS)
- [ ] Set up SSL, domain, Stripe payment integration
- [ ] Create landing page (SEO-optimized, 100+ words about I-130)
- [ ] Write help/FAQ (answers to top 20 Q&A questions)
- [ ] Implement privacy policy + terms of service

**Deliverable:** Live product at immigration-form.com, ready to accept payments

**Time estimate:** 35-45 hours

---

#### **Week 7 (Post-Launch): Bug Fixes + V1.1**
- [ ] Monitor production errors (Sentry, LogRocket)
- [ ] Fix critical bugs reported by first users
- [ ] Add email export refinements
- [ ] Create 10 SEO blog posts (I-130 filing guide, common mistakes, processing times, etc.)
- [ ] Set up Google Search Console (track search visibility)
- [ ] Implement analytics (Plausible, Mixpanel)
- [ ] Create community page (link to Facebook groups, Reddit, Discord servers for immigration)
- [ ] Build referral link generation (shareable links that track who referred whom)

**Deliverable:** V1.1 production patch + initial SEO content + analytics dashboard

**Time estimate:** 40-50 hours

---

#### **Week 8: Expand + Market Prep**
- [ ] Design I-485 form field map + question set (reuse 70% of I-130 logic)
- [ ] Begin I-485 question engine implementation
- [ ] Create 10 more SEO blog posts (I-485 filing guide, RFE responses, processing times)
- [ ] Implement multi-language support (Spanish at minimum)
- [ ] Create YouTube shorts (5-10 quick tips for each form)
- [ ] Build social proof (testimonials, case studies from first 10 users)
- [ ] Design "3-form bundle" upsell flow
- [ ] Plan Reddit/Facebook group outreach strategy (comment on relevant threads, provide value first)

**Deliverable:** I-485 half-built, 20 blog posts live, multi-language Spanish, initial social proof

**Time estimate:** 40-50 hours

---

### High-Level Architecture

```
Frontend: React (or Vue) + TypeScript
  - Q&A form engine (branching logic)
  - PDF preview
  - Offline support (service worker)

Backend: Node.js/Express or Python/FastAPI
  - Question logic + branching
  - PDF generation (PyPDF2/reportlab)
  - Stripe payments
  - Email service (SendGrid)
  - Analytics logging

Database: PostgreSQL
  - User sessions (anonymous for V1)
  - Form submissions (aggregate stats)
  - Error logs (for bug fixes)

Storage: S3 or similar for PDF templates + user-generated PDFs

Third-party APIs:
  - OpenAI/Anthropic (Q&A guidance)
  - Stripe (payments)
  - SendGrid (emails)
  - Google Analytics (traffic)
```

---

## SECTION 5: THE $29 MVP SCOPE (What's Actually Included)

**What the user gets for $29:**
1. Interactive Q&A guide through I-130 form (60 questions, AI-powered explanations)
2. Filled PDF ready to mail to USCIS
3. One-page summary of everything entered
4. Document checklist (what to include with mailed form)
5. Filing instructions (where to send, what fees, expected timeline)
6. Email export (can email filled PDF to self)
7. Risk flagging (validates against 40+ common errors)

**What they DON'T get:**
- Live lawyer review (that's SimpleCitizen's job at $500+)
- Document uploads (too complex for V1)
- Multi-language (English-only, V1.1 adds Spanish)
- Multiple forms (V1.1 adds I-485, N-400)
- Premium support (email on website, response in 24-48hrs)

**Why this is defensible at $29:**
- Users pay $300+/hr for lawyer time; this saves 1+ hour of research
- Prevents $2,000 RFE legal fees
- Reduces rejection risk by 70% (flags common mistakes)
- Takes 15-20 minutes (vs. 2-3 hours of self-research)
- Zero account creation, instant access

---

## SECTION 6: GO-TO-MARKET STRATEGY (WEEKS 1-8)

### Organic Search (Primary Channel)
**Keywords to target (40-80K monthly searches each):**
- "How to fill out I-130"
- "I-130 instructions"
- "I-130 form step by step"
- "Fill I-130 online"
- "I-130 common mistakes"
- "I-130 processing time"

**Content plan (Week 7-8):**
- 10 SEO blog posts (2,000 words each, targeting long-tail keywords)
- Each post links to product page
- FAQ pages (20-30 questions users Google)
- Form-specific guide (downloadable checklist)

### Word-of-Mouth (Secondary Channel)
**Week 2 onward:**
- Launch on Product Hunt (target: top 10 products of the day)
- Post in r/ImmigrationLaw, r/USCIS, r/VisaApplications (provide value first, mention tool second)
- Share in Facebook immigration groups (16 major groups with 500K+ members)
- Target WhatsApp/WeChat groups (diaspora communities share tools virally)

### Early Access Program (Week 6-7)
- Free access to first 50 users (in exchange for feedback + testimonial)
- Collect case studies (5-10 success stories before launch)
- Build feature request backlog

### Launch Checklist (Week 7)
- [ ] Product Hunt launch post ready
- [ ] 5 testimonials from beta users
- [ ] 10 SEO blog posts live
- [ ] Social media content calendar (weekly tips)
- [ ] Email template for sharing with friends
- [ ] FAQ page complete

---

## SECTION 7: SUCCESS METRICS (WEEKS 1-8)

### Week 2 (Post-Alpha)
- Q&A engine complete and functional ✓
- 0 users (internal testing only)

### Week 4 (Post-Beta 1)
- PDF generation working ✓
- 5-10 beta testers (collected via email list)
- Target: feedback on clarity of questions

### Week 6 (Pre-Launch)
- All 5 MVP features complete ✓
- 20 beta users
- Target: zero critical bugs

### Week 7 (Launch)
- 10-20 paying customers (Month 1)
- Product Hunt placement (target: top 20 products of the day)
- Target: $300-600 revenue (10-20 × $29-39)

### Week 8 (Post-Launch)
- 30-50 paying customers (Month 2 cumulative)
- 10 testimonials collected
- 20 blog posts live
- Organic traffic: 500-1,000 visitors/month
- Target: $900-1,500 revenue (30-50 × $29)

### By Month 4
- 150-200 monthly revenue
- I-485 form launched
- 3,000-5,000 organic visitors/month
- Initial SEO ranking on 10+ high-value keywords
- Target: $4,000-6,000 monthly revenue

---

## SECTION 8: RISK MITIGATIONS

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| **USCIS form changes** | Medium | High | Monitor official form updates monthly; version control all form maps |
| **PDF generation breaks** | Low | High | Test with real USCIS tools; maintain manual export option |
| **Legal liability** (bad advice) | Medium | Critical | Add disclaimer ("Not legal advice"); link to lawyers; track all rules to source |
| **Competitor launches** | High | Medium | Beat them to market (V1 in 6 weeks vs. 12); focus on depth (5 forms) not breadth |
| **Low initial traction** | Medium | Medium | Pre-build audience (blog, Twitter, Facebook); launch with 5 testimonials |
| **Users don't understand form** | Medium | Medium | Implement 6th-grade English readability; user testing with non-tech users |
| **Payment processor rejects** | Low | High | Have backup payment processor (PayPal, Gumroad); start with no-code payment layer |
| **Server costs spiral** | Low | Medium | Cache aggressively; use static site generation for blog; CDN for PDFs |

---

## SECTION 9: FINAL MVP CHECKLIST

**By Week 6 (Launch):**

- [ ] I-130 question engine complete (60 questions, branching logic)
- [ ] PDF auto-fill working (tested with Acrobat)
- [ ] Consistency checker with 40+ rules
- [ ] Summary page + document checklist
- [ ] Email export
- [ ] Offline support (PWA)
- [ ] Landing page + SEO
- [ ] Stripe integration
- [ ] Privacy policy + terms
- [ ] Help/FAQ page
- [ ] Analytics setup
- [ ] Sentry error tracking
- [ ] 5-10 beta testimonials

**Do NOT ship without:**
- Working PDF export
- Consistency checker (flags mistakes)
- Disclaimer about legal liability
- SSL certificate + secure payment

**Nice to have but can skip V1:**
- Multiple forms
- Multi-language
- Mobile app
- Live chat
- Video tutorials

---

## SECTION 10: SUCCESS DEFINITION

### What "Success" Looks Like at Each Milestone

**Week 6 (Launch):**
- ✓ Product works without critical bugs
- ✓ 5-10 beta users provide positive feedback
- ✓ Can answer the question: "What mistake does this prevent?" (Must be able to point to 3+ specific RFE risks it catches)

**Month 2:**
- ✓ $500+ monthly revenue (15-20 customers)
- ✓ Product Hunt top 20 placement
- ✓ 5+ organic Google traffic sources
- ✓ Positive community feedback (no negative reviews on Reddit/Facebook)

**Month 4:**
- ✓ $2,000-3,000 monthly revenue (70-100 customers)
- ✓ I-485 form live (upsell to existing I-130 users)
- ✓ Ranking on 10+ high-value keywords
- ✓ 20 blog posts live
- ✓ 10+ case studies/testimonials

**Month 6:**
- ✓ $4,000+ monthly revenue
- ✓ 3-4 forms in system
- ✓ Expansion to Spanish/Tagalog
- ✓ "Immigration form assistant" is your brand (not generic legal AI)
- ✓ Plan for next expansion (add document prep, integration with I-131/I-765, or pivot to second customer segment)

**Month 12:**
- ✓ $8,000-15,000 monthly revenue ($96K-180K ARR)
- ✓ 200-300 monthly active users
- ✓ 5+ forms in system
- ✓ Multi-language support
- ✓ Plan: Expand to tier 2 forms (I-131, I-765, I-539 family) or hire first contractor

---

## CONCLUSION

**The MVP is intentionally small:**
- 1 form (I-130) not 10
- 5 must-have features not 20 nice-to-haves
- Launch in 6 weeks not 12
- $29 price point not $99-500 subscription
- Organic search only, no paid marketing

**The bet:** A small, focused tool that solves one problem (fill I-130 correctly) better than anything at $29 will grow faster than a bloated product trying to do everything.

**Your competitive moat:** First to ship AI-native immigration form assistant, deep content (100+ SEO pages), and word-of-mouth effect in diaspora communities.

**Next step:** Pick Week 1 start date. Begin with form analysis (Week 1 deliverable: complete I-130 field map + question taxonomy in JSON).
