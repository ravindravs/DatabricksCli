# Metrics Dashboard Template - Quick Implementation

**For:** Immediate setup (copy/paste into Metabase, Google Sheets, or custom tool)

---

## GOOGLE SHEETS IMPLEMENTATION (Fastest)

### Sheet 1: Daily Metrics (Auto-update from Stripe API)

```
Date       | Daily Revenue | Sessions | Conversions | Conv Rate | MRR | ARR
-----------|---------------|----------|-------------|-----------|-----|-----
2026-03-24 | $145          | 52       | 3           | 5.8%      | $1280 | $15,360
2026-03-23 | $87           | 38       | 2           | 5.3%      | $1270 | $15,240
2026-03-22 | $116          | 45       | 2           | 4.4%      | $1248 | $14,976

Formulas:
- Daily Revenue = SUMIF(Stripe_Transactions, DATE = TODAY())
- Sessions = QUERY(Analytics, "SELECT COUNT(*) WHERE DATE = TODAY()")
- Conversions = COUNTIF(Payments, DATE = TODAY())
- Conv Rate = Conversions / Sessions
- MRR = SUM(Active Subscriptions × Monthly Price) + Projected PPOF
- ARR = MRR × 12
```

### Sheet 2: Monthly KPI Summary

```
┌─────────────────────────────────────────────────┐
│  MARCH 2026 - MONTHLY KPI SUMMARY               │
└─────────────────────────────────────────────────┘

REVENUE METRICS
├── MRR: $1,280
├── MRR Growth (vs Feb): +18%
├── ARR (annualized): $15,360
├── ARPU: $49
├── Gross Margin: 87.6%
└── Net Margin (est): 45%

CUSTOMER METRICS
├── Active Customers: 31
├── New Customers (month): 8
├── Churn Rate: 7.1%
├── Net Retention Rate: 110%
└── LTV/CAC Ratio: 91x

ACQUISITION
├── Organic Traffic: 523 visitors
├── Traffic Growth: +156%
├── Form Starts: 87
├── Form Completions: 38
├── Conversion Rate: 5.9%
└── Keywords Ranked (Top 20): 12

PRODUCT QUALITY
├── Form Completion Time: 18-28 min
├── Error Rate: 3.2%
├── NPS: 59
├── Support Tickets: 4
├── Uptime: 99.87%
└── Customer Satisfaction: 4.7/5

FINANCIAL HEALTH
├── Payback Period: 4.2 weeks ✓
├── 12-Month Runway: Infinite ✓
├── Unit Economics: Exceptional ✓
└── Status: Sustainable ✓
```

---

## METABASE SQL QUERIES (Professional)

### Query 1: Daily Revenue Tracking

```sql
SELECT
  DATE(payment_date) as date,
  SUM(amount) as daily_revenue,
  COUNT(DISTINCT customer_id) as daily_customers,
  SUM(CASE WHEN payment_type = 'subscription' THEN amount ELSE 0 END) as sub_revenue,
  SUM(CASE WHEN payment_type = 'ppof' THEN amount ELSE 0 END) as ppof_revenue,
  COUNT(*) as transactions
FROM payments
WHERE payment_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(payment_date)
ORDER BY date DESC
```

### Query 2: Monthly Cohort Retention

```sql
SELECT
  DATE_TRUNC(first_purchase_date, MONTH) as cohort_month,
  DATE_TRUNC(payment_date, MONTH) as payment_month,
  COUNT(DISTINCT customer_id) as retained_customers
FROM payments p
JOIN customers c ON p.customer_id = c.id
GROUP BY cohort_month, payment_month
ORDER BY cohort_month, payment_month
```

### Query 3: Conversion Rate by Form Type

```sql
SELECT
  f.form_type,
  COUNT(DISTINCT f.session_id) as form_starts,
  COUNT(DISTINCT p.customer_id) as conversions,
  ROUND(100.0 * COUNT(DISTINCT p.customer_id) / COUNT(DISTINCT f.session_id), 2) as conversion_rate,
  SUM(p.amount) as revenue
FROM form_sessions f
LEFT JOIN payments p ON f.customer_id = p.customer_id
  AND DATE(f.session_date) = DATE(p.payment_date)
WHERE f.session_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY f.form_type
ORDER BY conversion_rate DESC
```

### Query 4: LTV Calculation (Segment)

```sql
WITH customer_lifetime AS (
  SELECT
    c.id as customer_id,
    SUM(p.amount) as total_revenue,
    MAX(p.payment_date) as last_purchase,
    DATE_DIFF(MAX(p.payment_date), MIN(p.payment_date), DAY) / 30.0 as months_active,
    COUNT(*) as purchase_count
  FROM customers c
  LEFT JOIN payments p ON c.id = p.customer_id
  GROUP BY c.id
)
SELECT
  AVG(total_revenue) as avg_ltv,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total_revenue) as median_ltv,
  MAX(total_revenue) as max_ltv,
  COUNT(*) as total_customers
FROM customer_lifetime
```

### Query 5: Churn Rate Calculation

```sql
WITH monthly_subscriptions AS (
  SELECT
    DATE_TRUNC(payment_date, MONTH) as month,
    COUNT(DISTINCT customer_id) as active_subscribers
  FROM payments
  WHERE payment_type = 'subscription'
  GROUP BY DATE_TRUNC(payment_date, MONTH)
)
SELECT
  month,
  active_subscribers,
  LAG(active_subscribers) OVER (ORDER BY month) as prev_month_subscribers,
  ROUND(100.0 * (LAG(active_subscribers) OVER (ORDER BY month) - active_subscribers)
    / LAG(active_subscribers) OVER (ORDER BY month), 2) as churn_rate
FROM monthly_subscriptions
ORDER BY month DESC
```

---

## STRIPE DASHBOARD CONFIGURATION

### Dashboards to Create:

**1. Revenue Dashboard:**
```
Metric                          | Data Source
--------------------------------|----------
Monthly Recurring Revenue (MRR) | Subscriptions → Sum of amounts
Annual Recurring Revenue (ARR)  | Formula: MRR × 12
New Revenue (Month)             | New subscriptions + new PPOF
Churn Revenue (Month)           | Canceled subscriptions
Refund Rate (%)                 | Refunds / Total transactions
```

**2. Customer Dashboard:**
```
Active Subscribers              | Stripe subscriptions (non-canceled)
New Customers (Month)           | New payment methods created
Churn Customers (Month)         | Canceled subscriptions
Customer Retention (%)          | (Remaining / Starting) × 100
Avg Revenue Per User            | MRR / Active customers
```

**3. Payment Metrics:**
```
Successful Transactions         | Status = succeeded
Failed Transactions             | Status = failed
Payment Success Rate (%)        | Succeeded / (Succeeded + Failed)
Average Transaction Value       | Sum of amounts / Count of transactions
Refund Count                    | Status = refunded
```

---

## ANALYTICS.JS IMPLEMENTATION (Product Analytics)

### Events to Instrument:

```javascript
// Page views
analytics.track('page_viewed', {
  page: 'form_start',
  form_type: 'i_130',
  user_id: 'anon_12345'
});

// Form interactions
analytics.track('form_started', {
  form_type: 'i_130',
  referrer: 'google',
  device: 'desktop'
});

analytics.track('form_question_answered', {
  form_type: 'i_130',
  question_id: 'q_5',
  answer: 'spouse'
});

analytics.track('form_completed', {
  form_type: 'i_130',
  time_to_complete: 1250,  // seconds
  errors_encountered: 2,
  validation_issues: []
});

analytics.track('form_abandoned', {
  form_type: 'i_130',
  question_id: 'q_15',  // where they dropped
  time_spent: 420,
  reason: 'unclear_question'
});

// Payments
analytics.track('payment_initiated', {
  amount: 29,
  form_type: 'i_130',
  payment_method: 'credit_card'
});

analytics.track('payment_completed', {
  amount: 29,
  revenue: 29,
  form_type: 'i_130',
  customer_id: 'cust_abc123'
});

analytics.track('payment_failed', {
  amount: 29,
  error: 'card_declined'
});

// Referrals
analytics.track('referral_link_generated', {
  referrer_id: 'cust_abc123',
  link_id: 'ref_xyz789'
});

analytics.track('referral_converted', {
  referrer_id: 'cust_abc123',
  referred_customer_id: 'cust_def456',
  revenue: 29
});

// Support
analytics.track('support_ticket_created', {
  issue: 'pdf_download_failed',
  form_type: 'i_130'
});

// Errors
analytics.track('validation_error', {
  error_type: 'date_format',
  form_type: 'i_130',
  question_id: 'q_8'
});
```

---

## WEEKLY MONITORING CHECKLIST

### Monday Morning (8 AM) - Check These:

```
☐ Last 7-day MRR vs previous week
☐ Last 7-day conversion rate
☐ Last 7-day organic traffic
☐ Form abandonment rate (by form)
☐ Support tickets (any patterns?)
☐ NPS score (if surveys done)
☐ Error rate (check Sentry)
☐ Uptime (check CloudWatch)

Action if any metric down >20%:
- Investigate root cause immediately
- Check analytics for traffic source changes
- Review support tickets for user feedback
- Test product locally for bugs
```

### End of Month (28th) - Full Review:

```
REVENUE CHECKPOINT:
☐ YTD ARR vs projections
☐ MRR growth rate % (vs previous month)
☐ Average Customer Lifetime Value
☐ LTV/CAC ratio (should be >50x)

CUSTOMER CHECKPOINT:
☐ Total customers (cumulative)
☐ New customers (month)
☐ Churn rate (% lost)
☐ NPS score (survey if <12 responses)

ACQUISITION CHECKPOINT:
☐ Organic traffic growth %
☐ Keywords ranked (top 10, top 20)
☐ Form completion rate by type
☐ Traffic sources (breakdown %)

PRODUCT CHECKPOINT:
☐ Form completion time (by form)
☐ Error rate (validation errors)
☐ Support tickets (count + themes)
☐ Uptime % + critical incidents

FINANCIAL CHECKPOINT:
☐ Gross margin %
☐ Payback period (weeks)
☐ Runway (months if bootstrapped)
☐ Profitability status
```

---

## ALERT THRESHOLDS (Automated)

### Critical (Immediate Slack Alert):

```
IF daily_revenue < ($300 = 1/10 of expected MRR):
  → "⚠️ CRITICAL: Daily revenue below $300"

IF form_completion_rate < 20%:
  → "⚠️ CRITICAL: Form completion dropped below 20%"

IF conversion_rate < 2%:
  → "⚠️ CRITICAL: Conversion rate dropped below 2%"

IF uptime < 99%:
  → "🚨 CRITICAL: Service uptime dropped below 99%"

IF error_rate > 5%:
  → "⚠️ CRITICAL: Error rate exceeded 5%"
```

### Warning (Daily Digest):

```
IF daily_revenue between $200-300:
  → Include in daily digest

IF form_completion between 25-30%:
  → Include in daily digest

IF conversion_rate between 2-3%:
  → Include in daily digest

IF support_tickets > 3 in one day:
  → Include in daily digest
```

### Informational (Weekly Summary):

```
Weekly MRR summary
Weekly traffic growth
Weekly cohort retention
Weekly NPS responses
Weekly feature releases
```

---

## DASHBOARD TOOL COMPARISON

| Tool | Cost | Setup Time | Best For |
|------|------|-----------|----------|
| **Google Sheets** | $0 | 1 hour | Bootstrapped founder (MVP) |
| **Metabase** | $0 (self-hosted) | 4 hours | Full SQL power + custom charts |
| **Tableau** | $70/mo | 8 hours | Beautiful, professional dashboards |
| **Amplitude** | $0 (free tier) | 2 hours | Event tracking + funnels |
| **Plausible** | $9/mo | 1 hour | Privacy-focused analytics |
| **Mixpanel** | $0 (free tier) | 2 hours | User retention + cohorts |

**Recommended Stack for Bootstrap:**
```
Phase 1 (MVP): Google Sheets + Stripe dashboard
Phase 2 (Traction): + Plausible Analytics
Phase 3 (Growth): + Metabase + Mixpanel
```

---

## MONTHLY METRIC CHECKLIST

### Start of Month:
```
☐ Create new row in historical tracker
☐ Set targets for the month
☐ Review last month's root causes (what changed?)
☐ Plan 3 improvements based on metrics
```

### Mid-Month:
```
☐ Check if on pace for monthly targets
☐ Identify any metric deviating >20% from target
☐ Take corrective action if needed
☐ Update dashboard
```

### End of Month:
```
☐ Final month close (ensure all transactions captured)
☐ Calculate all KPIs
☐ Compare to benchmarks
☐ Write 1-page summary (wins, losses, next actions)
☐ Share with stakeholders (if any)
☐ Archive dashboard snapshot (for historical tracking)
```

---

## DECISION TREE

```
START: Review monthly metrics

  ↓

Is MRR > target?
├── YES → Is conversion rate > target?
│   ├── YES → SCALE: Keep doing what works
│   └── NO → OPTIMIZE: Fix funnel leaks
│
└── NO → Is organic traffic > target?
    ├── YES → PIVOT PRODUCT: UX issue
    ├── NO → PIVOT MARKETING: SEO not working
    └── < 50% of target → REASSESS: Market doesn't exist?

Decision Point: Are 70%+ of KPIs on target?
├── YES (8+ metrics) → Continue strategy, minor tweaks
├── SOME (4-7 metrics) → 1 major change + continue
├── FEW (< 4 metrics) → Major pivot or shutdown
```

---

## TEMPLATE: MONTHLY METRICS REPORT

```
# MARCH 2026 - MONTHLY METRICS REPORT

## Summary
- **Status:** ✓ On Track
- **MRR:** $1,280 (+18% vs Feb)
- **Revenue:** $3,847 (month)
- **New Customers:** 8
- **Churn Rate:** 7.1%

## Highlights
✓ Form completion rate improved to 44%
✓ Organic traffic up 156% YoY
✓ Conversion rate up 84% vs launch
✓ NPS score 59 (excellent)

## Concerns
⚠️ DS-160 form underperforming (2.1% conv)
⚠️ Mobile conversion 1.9x lower than desktop
⚠️ "Co-sponsor income" question has high abandonment

## Actions for April
1. Redesign DS-160 form (simplify UX)
2. A/B test mobile experience
3. Improve "Co-sponsor income" explanation
4. Launch Spanish language support
5. Test bundle pricing (3-form package)

## Next Month Targets
- MRR: $1,500 (+17%)
- ARR: $18,000+
- Conversion Rate: 6-7%
- Organic Traffic: 800+
```

---

## FINAL SETUP CHECKLIST (Week 1)

```
Week 1: Get metrics running
☐ Set up Stripe dashboard
☐ Create Google Sheets tracker
☐ Install Plausible Analytics
☐ Add tracking events to website
☐ Create Metabase dashboard (optional)
☐ Set Slack alerts
☐ Create weekly review calendar event

Week 2: First data review
☐ Collect first week of metrics
☐ Compare to expectations
☐ Document baseline metrics
☐ Identify 3 metrics to optimize

Week 3: Optimization sprint
☐ Implement 1-2 improvements (highest impact)
☐ Re-measure impact
☐ Document what changed

Week 4: Month-end close
☐ Full metrics review
☐ Write monthly report
☐ Share with stakeholders
☐ Plan next month
```

---

## REFERENCE: Expected Ranges by Month

| Month | MRR | Customers | Conversion | Organic Traffic | ARR |
|-------|-----|-----------|------------|-----------------|-----|
| **Launch (M2)** | $300-500 | 10-15 | 3-5% | 100-300/mo | $3.6K-6K |
| **M3** | $1,200-2,000 | 30-50 | 4-6% | 500-1,200/mo | $14-24K |
| **M4** | $1,800-3,000 | 45-75 | 5-7% | 1,000-2,000/mo | $22-36K |
| **M6** | $2,500-4,500 | 60-90 | 5-8% | 2,000-3,500/mo | $30-54K |
| **M12** | $6,000-10,000 | 150-200 | 5-7% | 5,000-8,000/mo | $72-120K |

---

**Start today:** Set up Google Sheets + Stripe dashboard (1 hour). Add Plausible next week. You'll have full visibility by launch.
