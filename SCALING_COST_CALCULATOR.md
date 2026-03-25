# AI Immigration Form Tool - Cost Calculator & Deployment Checklist

---

## COST CALCULATOR

### Interactive Cost Model

**Instructions:** Fill in your expected daily form volume to see total costs.

```
Daily Forms: _______
Expected conversion from your traffic volume

Monthly Forms = Daily Forms × 30
```

---

## SMALL SCALE COST CALCULATOR (100-200 Forms/Day)

### Platform: Vercel/Railway

```
Input Variables:
- Forms per day: 100
- Forms per month: 3,000
- Average price per form: $44
- LLM tokens per form: 1,000
- PDF generation per form: 3 seconds
```

### Cost Breakdown

| Service | Unit Cost | Monthly Volume | Monthly Cost |
|---------|-----------|---|---|
| **COMPUTE** | | | |
| Vercel Hosting | Included in free | N/A | $0 |
| Node.js serverless | $0.50 per 1GB-hour | 75 GB-hours | $37 |
| **STORAGE & DATABASE** | | | |
| Railway PostgreSQL | Free (5GB tier) | 50 MB | $0 |
| Railway Redis | $0.10/GB | 0.5GB | $5 |
| Vercel Blob (PDFs) | $0.50/GB | 5GB | $2.50 |
| **LLM APIS** | | | |
| OpenAI GPT-4 | $0.50/form | 3,000 forms | $1,500 |
| **PAYMENTS** | | | |
| Stripe | 2.9% + $0.30 | $132,000 revenue | $3,828 |
| **MONITORING & TOOLS** | | | |
| Sentry Error Tracking | Free | N/A | $0 |
| Plausible Analytics | $9/month | N/A | $9 |
| **EMAIL & SERVICES** | | | |
| SendGrid | Free (100/day) | N/A | $0 |
| Domain & SSL | $1/month | N/A | $1 |
| **TOTAL MONTHLY COST** | | | **$6,382.50** |

### Revenue at Small Scale

```
Monthly Forms: 3,000
Price per form: $44
Gross Revenue: $132,000
Stripe Fees: -$3,828
Net Revenue: $128,172

Operating Costs: -$6,382.50
NET PROFIT: $121,789.50
Profit Margin: 92.2%
```

### Small Scale Summary

| Metric | Value |
|--------|-------|
| **Infrastructure Cost** | $150/month |
| **LLM Cost** | $1,500/month |
| **Total Operating Cost** | $6,382/month |
| **Break-even Forms/Day** | ~10 |
| **Expected Profit at 100 forms/day** | $121K/month |
| **When to Scale** | > 150 forms/day |
| **Team Needed** | 1 founder (part-time) |
| **Ops Overhead** | < 5 hours/week |

---

## MEDIUM SCALE COST CALCULATOR (500-2,000 Forms/Day)

### Platform: AWS

```
Input Variables:
- Forms per day: 1,000
- Forms per month: 30,000
- Average price per form: $44
- LLM tokens per form: 1,000
- PDF generation per form: 3 seconds
- Concurrent users: ~100
```

### Cost Breakdown

| Service | Unit Cost | Monthly Volume | Monthly Cost |
|---------|-----------|---|---|
| **COMPUTE** | | | |
| ECS Fargate (3 AZ) | $14.4/vCPU-month | 9 vCPU | $130 |
| ECS memory (2GB each) | $1.54/GB-month | 18GB | $28 |
| Application Load Balancer | $22.50 + $0.006/LCU | 100 LCU | $22.50 + $0.60 = $23 |
| **STORAGE & DATABASE** | | | |
| RDS Aurora (db.t4g.small) | $0.084/hour | 730 hours | $61 |
| RDS Read Replicas (2x) | $0.042/hour x 2 | 1,460 hours | $61 |
| RDS Backup Storage | $0.10/GB | 50GB | $5 |
| ElastiCache Redis | $0.017/hour | 730 hours | $12 |
| S3 Storage (PDFs) | $0.023/GB | 50GB | $1 |
| S3 Data Transfer (out) | $0.09/GB | 50GB | $4.50 |
| **LLM APIS** | | | |
| OpenAI GPT-4 (real-time 30%) | $0.50/form | 9,000 forms | $4,500 |
| Claude Batch (70%) | $0.03/form | 21,000 forms | $630 |
| **COMPUTE FOR PDF** | | | |
| Lambda for PDF generation | $0.0000002/req | 30,000 reqs | $6 |
| Lambda compute (1GB, 3s) | $0.0000166/sec | 90,000 seconds | $1.50 |
| **NETWORKING** | | | |
| CloudFront (CDN) | $0.085/GB | 30GB | $2.55 |
| NAT Gateway | $0.045/hour | 730 hours | $33 |
| Route53 | $0.50 | - | $0.50 |
| **PAYMENTS & SERVICES** | | | |
| Stripe | 2.9% + $0.30 | $1.32M revenue | $38,280 |
| **MONITORING & LOGGING** | | | |
| CloudWatch Logs | $0.50/GB ingested | 5GB | $2.50 |
| CloudWatch Alarms | $0.10/alarm | 10 alarms | $1 |
| Datadog APM | $31/host/month | 3 hosts | $93 |
| **SECURITY & COMPLIANCE** | | | |
| AWS Secrets Manager | $0.40 + $0.05/API | 30 retrievals | $1.90 |
| VPC Security Groups | Free | - | $0 |
| **TOTAL MONTHLY COST** | | | **$43,882.45** |

### Cost Optimization at Medium Scale

**By switching to batch APIs:**
- Replace 70% of OpenAI with Claude batch (50% cheaper)
- Reduces LLM cost from $15,000 → $5,130
- **Total cost drops to $33,765**

### Revenue at Medium Scale

```
Monthly Forms: 30,000
Price per form: $44
Gross Revenue: $1,320,000
Stripe Fees: -$38,280
Net Revenue: $1,281,720

Operating Costs: -$33,765 (with batch optimization)
NET PROFIT: $1,247,955
Profit Margin: 97.4%
```

### Medium Scale Summary

| Metric | Value |
|--------|-------|
| **Infrastructure Cost** | $2,100/month |
| **LLM Cost (with batch)** | $5,130/month |
| **Total Operating Cost** | $33,765/month |
| **Break-even Forms/Day** | ~45 |
| **Expected Profit at 1K forms/day** | $1.25M/month |
| **Cost per Form** | $1.13 |
| **When to Scale** | > 2,000 forms/day |
| **Team Needed** | 1 founder + 1-2 ops |
| **Ops Overhead** | 15-20 hours/week |

---

## LARGE SCALE COST CALCULATOR (5,000-15,000 Forms/Day)

### Platform: AWS Multi-Region

```
Input Variables:
- Forms per day: 10,000
- Forms per month: 300,000
- Average price per form: $44
- LLM tokens per form: 1,000
- PDF generation per form: 3 seconds
- Concurrent users: ~1,000 (globally)
```

### Cost Breakdown

| Service | Unit Cost | Monthly Volume | Monthly Cost |
|---------|-----------|---|---|
| **COMPUTE (US-East)** | | | |
| ECS Fargate Tasks (20) | $14.40/vCPU | 40 vCPU | $576 |
| ECS Memory (4GB each) | $1.54/GB | 80GB | $123 |
| ALB + NLB | $22.50 + $0.006/LCU | 500 LCU | $25 |
| **COMPUTE (EU-West)** | | | |
| ECS Fargate Tasks (10) | $14.40/vCPU | 20 vCPU | $288 |
| ECS Memory (4GB each) | $1.54/GB | 40GB | $62 |
| ALB | $22.50 + $0.006/LCU | 200 LCU | $24 |
| **DATABASE** | | | |
| Aurora PostgreSQL (db.r6g.2xlarge) | $2.95/hour | 730 hours | $2,154 |
| Aurora Read Replicas (3x) | $1.475/hour x 3 | 2,190 hours | $3,231 |
| Aurora Global DB Replication | $1.00/hour | 730 hours | $730 |
| Enhanced Backup (14 days) | $0.095/GB | 500GB | $47.50 |
| **CACHE** | | | |
| ElastiCache Redis US (r6g.xlarge) | $0.48/hour | 730 hours | $350 |
| ElastiCache Redis EU (r6g.large) | $0.24/hour | 730 hours | $175 |
| **STORAGE** | | | |
| S3 Standard (PDFs) | $0.023/GB | 500GB | $11.50 |
| S3 Cross-Region Replication | $0.02/GB | 500GB | $10 |
| S3 Data Transfer (out) | $0.09/GB | 300GB | $27 |
| Glacier Archive (old PDFs) | $0.004/GB | 1000GB | $4 |
| **LLM APIS** | | | |
| Claude Batch (85%) | $0.03/form | 255,000 forms | $7,650 |
| Claude 3.5 Sonnet Real-time (15%) | $0.15/form | 45,000 forms | $6,750 |
| **AWS Bedrock (alternative)** | Provisioned | 300M tokens | $3,000 |
| **LAMBDA (PDF Gen & Batch)** | | | |
| Lambda Requests | $0.20/1M | 300K requests | $60 |
| Lambda Compute (1GB, 3s) | $0.0000166/s | 900K seconds | $15 |
| Lambda Concurrent Execution | $0.000004/GB-s | 1000 concurrent GB | $100 |
| **NETWORKING** | | | |
| CloudFront (Global CDN) | $0.085/GB | 300GB | $25.50 |
| NAT Gateway (US) | $0.045/hour x 2 | 1,460 hours | $131 |
| NAT Gateway (EU) | $0.045/hour | 730 hours | $33 |
| Route53 | $0.50/hosted zone | 1 zone | $0.50 |
| VPC Peering | Free | - | $0 |
| **MONITORING, LOGGING & SECURITY** | | | |
| CloudWatch Logs | $0.50/GB ingested | 50GB | $25 |
| CloudWatch Metrics | $0.30 | 100 metrics | $30 |
| CloudWatch Alarms | $0.10 | 50 alarms | $5 |
| Datadog Enterprise APM | $100/host | 6 hosts | $600 |
| AWS Config | $2/rule/month | 5 rules | $10 |
| Secrets Manager | $0.40 + $0.05/API | 100 API calls | $5.40 |
| GuardDuty (threat detection) | $4/1M events | 10M events | $40 |
| **PAYMENTS & SERVICES** | | | |
| Stripe | 2.9% + $0.30 | $13.2M revenue | $382,800 |
| SendGrid | $10-100/month | 1M emails | $100 |
| **BACKUP & DISASTER RECOVERY** | | | |
| AWS Backup | $5/month + usage | 500GB | $10 |
| Disaster Recovery SLA | Insurance | - | $500 |
| **TEAM & OPERATIONS** | | | |
| On-call engineer stipend | $5,000/month | 1 person | $5,000 |
| **TOTAL MONTHLY COST** | | | **$53,816.80** |

### Cost Optimization at Large Scale

**By using AWS Bedrock with provisioned throughput:**
- Bedrock: $3,000/month (vs. $14,400 for Claude API)
- **Saves: $11,400/month on LLM**

**Total cost with Bedrock:** $42,417/month

### Revenue at Large Scale

```
Monthly Forms: 300,000
Price per form: $44
Gross Revenue: $13,200,000
Stripe Fees: -$382,800
Net Revenue: $12,817,200

Operating Costs: -$42,417 (with Bedrock optimization)
NET PROFIT: $12,774,783
Profit Margin: 99.7%
```

### Large Scale Summary

| Metric | Value |
|--------|-------|
| **Infrastructure Cost** | $8,000/month |
| **LLM Cost (with Bedrock)** | $3,000/month |
| **Total Operating Cost** | $42,417/month |
| **Break-even Forms/Day** | ~180 |
| **Expected Profit at 10K forms/day** | $12.77M/month |
| **Cost per Form** | $0.14 |
| **Revenue per Form** | $44 |
| **Gross Margin** | 99.7% |
| **Team Needed** | 1 founder + 2-3 ops + 1 DBA |
| **Ops Overhead** | 40-50 hours/week |

---

## SCALING TIMELINE & COST PROJECTION

```
Month 1-3: Vercel/Railway
├─ Daily forms: 50-100
├─ Monthly cost: $600
├─ Monthly revenue: $66K
├─ Monthly profit: $65.4K
└─ ROI: 10,900% / month

Month 4-6: AWS Medium
├─ Daily forms: 300-1,000
├─ Monthly cost: $33.7K
├─ Monthly revenue: $440K - $1.32M
├─ Monthly profit: $406K - $1.25M
└─ ROI: 1,200%-3,700% / month

Month 7-12: AWS Large
├─ Daily forms: 2,000-10,000
├─ Monthly cost: $42.4K
├─ Monthly revenue: $2.6M - $13.2M
├─ Monthly profit: $2.56M - $12.77M
└─ ROI: 6,000%-30,000% / month
```

---

## COST COMPARISON TABLE

### All-In Operating Costs (Infrastructure + LLM + Payments)

| Scale | Forms/Day | Monthly Cost | Revenue | Profit | Margin |
|-------|-----------|---|---|---|---|
| Small (Vercel) | 100 | $6.4K | $132K | $125.6K | 95.2% |
| Small (Vercel) | 150 | $9.1K | $198K | $188.9K | 95.4% |
| Medium (AWS) | 500 | $20K | $660K | $640K | 97% |
| Medium (AWS) | 1,000 | $33.7K | $1.32M | $1.28M | 97.4% |
| Large (AWS) | 5,000 | $42.4K | $6.6M | $6.56M | 99.4% |
| Large (AWS) | 10,000 | $42.4K | $13.2M | $13.16M | 99.7% |

**Key Insight:** Operating costs grow sub-linearly with volume due to economies of scale.

---

## DEPLOYMENT CHECKLIST

### Phase 1: MVP Launch (Week 1-6) - Vercel/Railway

#### Week 1: Infrastructure Setup
- [ ] Create Vercel account and project
- [ ] Set up Railway PostgreSQL instance (free tier)
- [ ] Set up Railway Redis instance
- [ ] Configure environment variables (.env)
- [ ] Set up GitHub repo with CI/CD
- [ ] Create Stripe test account

**Estimated time:** 4 hours
**Cost:** $0

#### Week 2: Backend Development
- [ ] Build Q&A engine API routes
- [ ] Implement database schema (60 form fields)
- [ ] Set up OpenAI API integration
- [ ] Implement form validation logic
- [ ] Create PDF generation endpoints

**Estimated time:** 40 hours
**Cost:** $0

#### Week 3: Frontend Development
- [ ] Build React form component (branching logic)
- [ ] Implement client-side validation
- [ ] Create PDF download button
- [ ] Add offline capability (service worker)
- [ ] Design error handling UI

**Estimated time:** 40 hours
**Cost:** $0

#### Week 4: Payment & Delivery
- [ ] Integrate Stripe payment flow
- [ ] Create payment success page
- [ ] Set up email delivery (SendGrid)
- [ ] Implement analytics tracking (Plausible)
- [ ] Set up error monitoring (Sentry)

**Estimated time:** 20 hours
**Cost:** $0 + Stripe + Plausible ($9)

#### Week 5: Testing & Security
- [ ] Manual QA testing (all form paths)
- [ ] Test payment flow (Stripe test mode)
- [ ] Security audit (HTTPS, headers, CSRF)
- [ ] Performance testing (load test with 100 concurrent)
- [ ] Backup strategy (automated PostgreSQL backups)

**Estimated time:** 20 hours
**Cost:** $0

#### Week 6: Launch & Monitoring
- [ ] Domain setup + SSL (Let's Encrypt)
- [ ] DNS configuration
- [ ] Enable Vercel analytics
- [ ] Create status page
- [ ] Launch on Product Hunt
- [ ] Monitor for errors (Sentry + CloudWatch)

**Estimated time:** 10 hours
**Cost:** $1 (domain)

### Phase 1 Summary
- **Total dev time:** 134 hours (3-4 weeks for 1-2 developers)
- **Infrastructure cost:** $10/month (first month)
- **Ready for:** 100-200 forms/day

---

### Phase 2: Scale to AWS Medium (Month 4-6)

#### Week 1: AWS Infrastructure
- [ ] Create AWS account and set up billing alerts
- [ ] Create VPC with public/private subnets (3 AZ)
- [ ] Set up RDS Aurora PostgreSQL cluster
- [ ] Create ElastiCache Redis instance
- [ ] Set up Application Load Balancer
- [ ] Create ECS cluster and task definition
- [ ] Configure CloudFront CDN
- [ ] Set up CloudWatch logs and alarms

**Estimated time:** 20 hours
**Cost:** $500 (infrastructure for 1 week testing)

#### Week 2: Application Deployment
- [ ] Containerize Node.js app (Docker)
- [ ] Push image to ECR (Elastic Container Registry)
- [ ] Deploy to ECS (staging environment)
- [ ] Test with production database size
- [ ] Configure auto-scaling policies
- [ ] Set up CI/CD pipeline (GitHub Actions → ECR → ECS)

**Estimated time:** 15 hours
**Cost:** $500 (infrastructure testing)

#### Week 3: Cutover & Monitoring
- [ ] Run parallel (Vercel + AWS) for 1 week
- [ ] Route 10% traffic to AWS (canary deployment)
- [ ] Monitor metrics (latency, errors, CPU)
- [ ] Fix issues discovered in canary
- [ ] Route 100% traffic to AWS
- [ ] Decommission Vercel
- [ ] Document runbooks (how to scale, disaster recovery)

**Estimated time:** 15 hours
**Cost:** $2,000 (parallel infrastructure)

#### Week 4: Optimization & Hardening
- [ ] Performance tuning (database indexes, caching)
- [ ] Security hardening (WAF, VPC security groups)
- [ ] Cost optimization (spot instances, reserved capacity)
- [ ] Backup testing (restore from snapshot)
- [ ] Load testing (1,000 concurrent users)

**Estimated time:** 10 hours
**Cost:** $1,000 (infrastructure)

### Phase 2 Summary
- **Total ops time:** 60 hours (2 weeks for 1-2 ops engineers)
- **Infrastructure cost:** $4,000 (migration period)
- **Ready for:** 500-2,000 forms/day
- **Team:** 1 founder + 1 part-time ops

---

### Phase 3: Scale to AWS Large (Month 7-12)

#### Week 1: Multi-Region Setup
- [ ] Create secondary region (EU-West)
- [ ] Set up Aurora Global Database
- [ ] Create ECS cluster in EU
- [ ] Set up Route53 latency-based routing
- [ ] Configure CloudFront multi-region

**Estimated time:** 20 hours
**Cost:** $2,000

#### Week 2: Deployment & Testing
- [ ] Deploy application to EU ECS
- [ ] Test failover (simulate region failure)
- [ ] Load test (5,000 concurrent users across regions)
- [ ] Verify data replication (< 1 second lag)
- [ ] Route 20% traffic to EU

**Estimated time:** 15 hours
**Cost:** $2,000

#### Week 3: Optimization
- [ ] Monitor latency improvements
- [ ] Route 50% traffic to EU
- [ ] Optimize database queries per region
- [ ] Implement regional caching (DynamoDB, Elasticache per region)
- [ ] Cost analysis and optimization

**Estimated time:** 10 hours
**Cost:** $2,000

#### Week 4: Finalization
- [ ] Route 100% traffic split (70% US, 30% EU)
- [ ] Document disaster recovery procedures
- [ ] Implement automated backups to Glacier
- [ ] Conduct full failover test
- [ ] Document on-call procedures

**Estimated time:** 10 hours
**Cost:** $1,000

### Phase 3 Summary
- **Total ops time:** 55 hours (2 weeks for 2-3 ops engineers)
- **Infrastructure cost:** $7,000 (migration period)
- **Ready for:** 5,000-15,000 forms/day
- **Team:** 1 founder + 2-3 ops + 1 DBA

---

## DEPLOYMENT DECISION TREE

```
Start: Scale with Vercel/Railway
│
├─ 100 forms/day?
│  └─ YES → Continue with Vercel/Railway
│           Cost: $600/month, Profit: $125K/month ✓
│
├─ 150 forms/day?
│  └─ YES → SCALE: Migrate to AWS Medium
│           Cost: $2,000/month, Profit: $1.3M/month
│           Migration effort: 60 hours
│
├─ 2,000 forms/day?
│  └─ YES → SCALE: Migrate to AWS Large Multi-Region
│           Cost: $42K/month, Profit: $12.7M/month
│           Migration effort: 55 hours
│
└─ > 10,000 forms/day?
   └─ YES → EXPAND: Consider enterprise tier
            Potential: $13.2M+ monthly profit
            Requires: 5+ person ops team
```

---

## RISK MITIGATION DURING SCALING

### Vercel → AWS Migration Risks

| Risk | Mitigation | Cost |
|------|-----------|------|
| **Data loss during migration** | Automated backups, snapshot before cutover | $50 |
| **Downtime during migration** | Blue-green deployment, 99.9% uptime SLA | Included |
| **Performance degradation** | Load testing before cutover, canary rollout | 10 hours ops |
| **Cost overrun** | Billing alerts, spot instance limits | $100-200 |
| **Database migration failure** | DMS (Database Migration Service), test restore | $200 |

### AWS Medium → Large Migration Risks

| Risk | Mitigation | Cost |
|------|-----------|------|
| **Cross-region replication lag** | RDS Global Database (< 1 second lag) | Included |
| **Regional failover issues** | Automated failover testing monthly | 5 hours ops/month |
| **Cost explosion** | Reserved instances, spot pricing, budget alerts | $100-200 |
| **DNS latency routing issues** | Route53 health checks, terraform testing | 10 hours ops |

---

## BUDGET TEMPLATES

### Small Scale Budget (Vercel/Railway)
```
Monthly Budget: $1,000
├─ Infrastructure: $150
├─ LLM API: $1,500 (scaled back by caching)
├─ Payments: $100
├─ Tools (monitoring, analytics): $100
└─ Buffer (10%): $50

Recommendation: Cap LLM at $500 via caching, batch processing
```

### Medium Scale Budget (AWS)
```
Monthly Budget: $20,000
├─ Infrastructure: $2,000
├─ LLM API: $8,000 (with batch optimization)
├─ Payments: $3,000
├─ Monitoring & Tools: $500
├─ On-call / SLA: $1,000
├─ DBA Time: $3,000
└─ Buffer (10%): $2,500

Recommendation: Monitor LLM costs weekly, use batch APIs aggressively
```

### Large Scale Budget (AWS)
```
Monthly Budget: $100,000
├─ Infrastructure: $8,000
├─ LLM API: $20,000 (with Bedrock provisioned)
├─ Payments: $10,000
├─ Monitoring & Tools: $2,000
├─ On-call / SLA: $5,000
├─ DBA Team (1 FTE): $8,000
├─ Ops Team (2 FTE): $15,000
├─ Legal & Compliance: $5,000
└─ Buffer (15%): $27,000

Recommendation: Implement cost allocation tags, charge-back per region
```

---

## FINAL RECOMMENDATIONS

### If You Have $1,000/month budget:
→ Start with Vercel/Railway
→ Scale to 100-150 forms/day
→ Profit: $125K/month

### If You Have $20,000/month budget:
→ Start with Vercel/Railway
→ Migrate to AWS Medium at 150 forms/day
→ Scale to 1,000 forms/day
→ Profit: $1.3M/month

### If You Have $100,000/month budget:
→ Start with Vercel/Railway
→ Migrate to AWS Medium at 150 forms/day
→ Migrate to AWS Large at 2,000 forms/day
→ Scale to 10,000+ forms/day
→ Profit: $12.7M+/month

**Bottom line:** Every dollar spent on infrastructure returns $100-300 in profit at scale. Invest aggressively in infrastructure, not marketing.
