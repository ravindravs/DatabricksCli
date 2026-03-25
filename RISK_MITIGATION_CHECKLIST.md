# API Dependency Risk - Quick Reference Checklist

## IMMEDIATE ACTIONS (Week 1)

### Financial Risk Mitigation
- [ ] **Review Anthropic API contract**
  - Check: Price increase clauses, minimum commitment terms
  - Action: Request 24-month pricing lock, negotiate 30-40% discount
  - Owner: Finance Lead
  - Timeline: 1 week
  - Cost: Negotiate volume commit

- [ ] **Review OpenAI API contract**
  - Check: Same as Anthropic
  - Action: Establish secondary account, set usage limits
  - Owner: Finance Lead
  - Timeline: 1 week
  - Cost: Free (account setup only)

- [ ] **Set up cost monitoring dashboard**
  - Tool: Anthropic + OpenAI usage APIs → Datadog/CloudWatch
  - Alert: If daily cost > $50, email finance
  - Target: Identify cost creep within 24 hours
  - Owner: DevOps Lead
  - Timeline: 2 days
  - Cost: $0 (use free tier)

### Content Risk Mitigation
- [ ] **Add legal disclaimers to product**
  - Location: Landing page, every form page, PDF export
  - Text: "This tool provides guidance only, not legal advice. Consult immigration attorney before filing."
  - Owner: Product + Legal
  - Timeline: 2 days
  - Cost: Legal review $500-1K

- [ ] **Create incident response plan for content restrictions**
  - Scenario: "Claude API becomes unavailable"
  - Action: Switch to Llama2 fallback within 2 hours
  - Communication: Notify customers via email, banner
  - Owner: Technical Lead
  - Timeline: 3 days
  - Cost: $0

### Outage Risk Mitigation
- [ ] **Enable response caching (Phase 1)**
  - Target: Reduce API calls by 40%
  - Implementation: LRU cache for 1,000 most common responses
  - Technology: Redis or in-memory cache
  - Owner: Backend Engineer
  - Timeline: 1 week
  - Cost: $0 (implement in-memory first)

- [ ] **Implement basic health monitoring**
  - Check: Claude API health every 60 seconds
  - Alert: If unavailable for >5 minutes
  - Escalation: Slack notification to on-call engineer
  - Owner: DevOps Lead
  - Timeline: 3 days
  - Cost: $0 (use free monitoring APIs)

---

## MONTH 2 IMPLEMENTATION (Months 2-4)

### Cost Optimization
- [ ] **Implement advanced caching strategy**
  - Pre-warm cache with 100 most common questions
  - Target: 60% cache hit rate (down from 0%)
  - Batch API calls (10 requests → 1 API call)
  - Owner: Backend Engineer
  - Timeline: 3 weeks
  - Cost: $500/month storage (Redis)

- [ ] **Use request compression**
  - Gzip all API payloads before sending
  - Reduce token overhead by 20-30%
  - Owner: Backend Engineer
  - Timeline: 1 week
  - Cost: $0

### Fallback Infrastructure
- [ ] **Deploy Llama2 7B in staging**
  - Setup: Use replicate.com API (managed service, no self-hosting)
  - Test: Compare output quality vs. Claude (target: 80%+)
  - Fine-tuning: Add LoRA adapter for immigration forms
  - Owner: ML Engineer
  - Timeline: 4 weeks
  - Cost: $2K (development) + $100/month (API credits)

- [ ] **Build provider abstraction layer**
  - Route requests: Claude → OpenAI → Llama2 → Fallback
  - Health checks: Every 60 seconds
  - Auto-failover: On timeout or error
  - Owner: Backend Engineer
  - Timeline: 2 weeks
  - Cost: $0 (development only)

- [ ] **Create fallback template system**
  - Pre-write 200+ template responses for common questions
  - Organize by form type (I-130, I-485, etc.)
  - Test: Verify readability (6th grade level)
  - Owner: Product + Legal
  - Timeline: 2 weeks
  - Cost: $1K (legal review)

### Monitoring & Alerting
- [ ] **Set up comprehensive logging**
  - Log: Every API call (provider, latency, cost, response)
  - Exclude: All PII (encrypted in logs)
  - Retention: 30 days (searchable)
  - Tool: CloudWatch, Datadog, or ELK
  - Owner: DevOps Lead
  - Timeline: 1 week
  - Cost: $200-500/month

- [ ] **Create provider health dashboard**
  - Metrics: Uptime %, latency, error rate, cost/24h
  - Audience: Engineering + Finance teams
  - Update: Real-time
  - Tool: Grafana or built-in dashboard
  - Owner: DevOps Lead
  - Timeline: 1 week
  - Cost: $0 (use free tier)

- [ ] **Set up alerting rules**
  - Alert 1: If any provider unavailable >5 min (Slack + PagerDuty)
  - Alert 2: If error rate >10% (Slack)
  - Alert 3: If daily cost >$100 (Email to Finance)
  - Alert 4: If latency >3s (Slack, non-critical)
  - Owner: DevOps Lead
  - Timeline: 1 week
  - Cost: $50/month (PagerDuty)

---

## MONTHS 4-8 HARDENING (Months 4-8)

### Production Deployment
- [ ] **Deploy Llama2 to production**
  - Setup: Use replicate.com (managed), not self-hosted
  - Traffic: Route 1% of requests to Llama2 (test)
  - Week 2: Route 5% to Llama2
  - Week 3: Route 10% to Llama2
  - Monitor: Error rate, latency, customer feedback
  - Owner: DevOps + Backend
  - Timeline: 2 weeks
  - Cost: $200/month (production traffic)

- [ ] **Enable auto-failover**
  - Trigger: Claude timeout >2s or error
  - Action: Switch to OpenAI (if healthy)
  - Fallback: Switch to Llama2 (if OpenAI fails)
  - Last resort: Use template response
  - Verify: 100% coverage (no missing responses)
  - Owner: Backend Engineer
  - Timeline: 1 week
  - Cost: $0

- [ ] **Test disaster scenarios**
  - Scenario 1: Kill Claude API, verify failover to OpenAI
  - Scenario 2: Kill OpenAI, verify failover to Llama2
  - Scenario 3: Kill all APIs, verify template fallback works
  - Scenario 4: Internet down on customer device, verify offline PWA works
  - Measure: Time to failover, user experience impact
  - Owner: QA + DevOps
  - Timeline: 2 weeks
  - Cost: $0

- [ ] **Implement customer-facing status page**
  - Tool: Statuspage.io or custom build
  - Content: Real-time API provider status
  - Transparency: Show which provider being used
  - Incidents: Log all outages >5 minutes
  - Owner: Product + DevOps
  - Timeline: 1 week
  - Cost: $20/month (Statuspage) or $0 (custom)

### Compliance & Documentation
- [ ] **Document data flow for compliance**
  - Create: Architecture diagram with encryption points marked
  - Map: Where data goes, how it's encrypted, which providers see what
  - Audience: Legal, security reviewers
  - Owner: Technical Lead
  - Timeline: 1 week
  - Cost: $0

- [ ] **Create incident response playbook**
  - Scenario 1: Claude API goes down
    - Detection: <5 min (health check fails)
    - Response: Switch to failover provider
    - Communication: Slack alert to team
    - Timeline: Recovery within 5-10 minutes
  - Scenario 2: Content restriction policy
    - Detection: Account suspended without warning
    - Response: Switch to Llama2 immediately
    - Communication: Email to all customers
    - Timeline: Recovery within 1 hour
  - Scenario 3: Data breach (hypothetical)
    - Detection: Notification from provider
    - Response: Rotate API keys, audit logs
    - Communication: Notify customers if PII exposed
    - Timeline: Full incident report within 48 hours
  - Owner: Technical Lead + Security
  - Timeline: 2 weeks
  - Cost: $0

- [ ] **Create runbooks for on-call**
  - Runbook 1: "API health check failing, what to do?"
    - Step 1: Check provider status page
    - Step 2: Check our health check logic (false alarm?)
    - Step 3: Verify failover working
    - Step 4: If broken, page backup engineer
  - Runbook 2: "Cost spike detected, what to do?"
    - Step 1: Check which provider increased
    - Step 2: Did we add new features?
    - Step 3: Is caching working?
    - Step 4: Alert finance
  - Owner: DevOps Lead
  - Timeline: 1 week
  - Cost: $0

---

## MONTHS 9-12 ONGOING (Months 9-12)

### Monthly Reviews
- [ ] **First Friday of each month: Cost review**
  - Check: Cost per customer (target: <$25)
  - Compare: Month-over-month trend
  - Action: If >$30/customer, trigger cost optimization
  - Owner: Finance Lead
  - Timeline: 1 hour
  - Cost: $0

- [ ] **First Friday of each month: Provider health review**
  - Check: Uptime % for each provider (target: >99%)
  - Check: Latency trends (target: <1s)
  - Check: Error rates (target: <1%)
  - Action: If degraded, notify provider + escalate
  - Owner: DevOps Lead
  - Timeline: 1 hour
  - Cost: $0

### Quarterly Deep Dives
- [ ] **Q1: Risk assessment update**
  - Question: Any new risks emerged?
  - Check: Provider contract updates, new policies
  - Action: Update contingency plans if needed
  - Owner: Technical Lead + Finance
  - Timeline: 4 hours
  - Cost: $0

- [ ] **Q2: Test disaster recovery**
  - Simulate: Full API outage for 2 hours
  - Measure: How many customers impacted? How long to recover?
  - Document: Lessons learned, improvements needed
  - Owner: DevOps + QA
  - Timeline: 8 hours (full simulation day)
  - Cost: $0 (internal only)

- [ ] **Q3: Competitive landscape review**
  - Question: Did competitors build multi-provider?
  - Action: Benchmark against competitors' reliability
  - Update: Marketing messaging about uptime SLA
  - Owner: Product + Marketing
  - Timeline: 4 hours
  - Cost: $0

- [ ] **Q4: Annual security audit**
  - Check: Any new vulnerabilities in dependencies?
  - Check: API credentials rotated recently?
  - Check: Access logs reviewed for anomalies?
  - Owner: Security Lead
  - Timeline: 8 hours
  - Cost: $1K (annual penetration test)

### Annual Planning
- [ ] **Review contract renewals**
  - Anthropic: Renegotiate pricing lock (every 24 months)
  - OpenAI: Review tier and usage limits
  - AWS: Optimize instance sizes, reserve capacity
  - Owner: Finance Lead
  - Timeline: 20 hours (negotiation)
  - Cost: Negotiate discounts

- [ ] **Budget for next year**
  - Llama2 infrastructure: $1-2K/month
  - Monitoring services: $50-200/month
  - Caching infrastructure: $500/month
  - Annual penetration test: $5-10K
  - Total: ~$25-30K/year
  - Owner: Finance Lead
  - Timeline: 8 hours
  - Cost: Included in budget

---

## QUICK STATUS DASHBOARD

### Risk Metrics (Updated Monthly)

**API Price Risk**
- Current unit cost (per 1K tokens): _____ (target: <$0.05)
- Price trend (6-month): _____ (target: flat or down)
- Mitigation readiness: _____ (target: 90%+)

**Content Restriction Risk**
- Llama2 deployed: ☐ Yes ☐ No (target: deployed by Month 4)
- Fallback coverage: ____% (target: 95%+)
- Customer awareness: ☐ Informed ☐ Not informed (target: informed)

**Outage Risk**
- API uptime (Claude): ___% (target: >99.5%)
- API uptime (OpenAI): ___% (target: >99.5%)
- Cache hit rate: ___% (target: >60%)
- Failover tested: ☐ Yes ☐ No (target: tested monthly)

**Financial Health**
- Month revenue: $_____ (target: >$5K Month 6+)
- API cost % of revenue: ___% (target: <20%)
- Profit margin: ___% (target: >50%)

---

## COST TRACKING TEMPLATE

| Month | Claude Cost | OpenAI Cost | Llama2 Cost | Infrastructure | Total | Cost/Customer |
|-------|-----------|-----------|-----------|-----------------|--------|-----------------|
| M1 | $0 | $0 | $0 | $0 | $0 | $0 |
| M2 | $100 | $0 | $0 | $500 | $600 | $60 |
| M3 | $150 | $0 | $0 | $500 | $650 | $22 |
| M4 | $200 | $50 | $0 | $800 | $1,050 | $13 |
| M5 | $250 | $100 | $100 | $1,000 | $1,450 | $11 |
| M6 | $300 | $150 | $150 | $1,200 | $1,800 | $9 |

---

## ESCALATION MATRIX

| Issue | Severity | Detection | Response | Owner |
|-------|----------|-----------|----------|-------|
| Provider latency >3s | Low | Automated alert | Investigate, no action needed | DevOps |
| Provider error rate >5% | Medium | Automated alert | Failover to backup, notify team | DevOps + Backend |
| Provider down >5 min | High | Automated alert | Activate failover, page on-call | DevOps + Manager |
| Cost spike >30% MoM | Medium | Monthly review | Investigate, optimize caching | Finance + Backend |
| New content restriction | Critical | Manual (Twitter, email) | Activate Llama2, notify customers | Tech Lead + Legal |
| Customer outage claim | High | Support ticket | Investigate, document, refund if warranted | Support + Tech Lead |

---

## SUCCESS CRITERIA (Month 12)

- [ ] **Cost:** API costs <20% of revenue (confirmed by finance)
- [ ] **Availability:** >99% uptime across all providers (confirmed by monitoring)
- [ ] **Failover:** Zero undetected outages (100% caught by health checks)
- [ ] **Customer impact:** <1% of customers experience any disruption (surveyed)
- [ ] **Preparedness:** All team members trained on runbooks (via quiz)
- [ ] **Compliance:** SOC2 ready, passed internal security audit
- [ ] **Financial:** Margin remains >50% despite price shocks (confirmed by accounting)

---

## KEY CONTACTS & ESCALATION

**Technical Lead (Overall Owner):**
- Name: _____________
- Email: _____________
- Slack: _____________
- On-call: _____________

**Finance Lead (Cost Owner):**
- Name: _____________
- Email: _____________
- Slack: _____________

**DevOps Lead (Infrastructure Owner):**
- Name: _____________
- Email: _____________
- Slack: _____________

**On-Call Engineer (Incident Response):**
- Primary: _____________
- Secondary: _____________
- Escalation: _____________

**External Contacts:**
- Anthropic API Support: support@anthropic.com
- OpenAI API Support: support@openai.com
- PagerDuty: [account URL]
- AWS Support: [account contact]

