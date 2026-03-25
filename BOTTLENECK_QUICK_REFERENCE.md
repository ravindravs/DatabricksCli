# AI Immigration Form Tool - Bottleneck & Scaling Quick Reference

---

## EXECUTIVE SUMMARY: WHAT BREAKS FIRST AT EACH SCALE

| Scale | Primary Bottleneck | Constraint | Solution | Cost Impact |
|-------|-------------------|-----------|----------|------------|
| **100/day** | None | All systems underutilized | Vercel/Railway is overkill | $0 |
| **150-200/day** | LLM API rate limits | OpenAI free tier (90 RPM) | Upgrade to paid OpenAI ($15/mo) | +$15 |
| **300/day** | Redis memory | Railway free tier (1GB) → 3GB needed | Upgrade Redis to paid tier | +$50 |
| **500/day** | Database CPU | RDS t3.micro is 90%+ utilized | Upgrade to db.t4g.small | +$60 |
| **1,000/day** | PDF generation concurrency | Need 15+ concurrent Lambda | Increase Lambda limit, add queue | +$100 |
| **2,000/day** | Database I/O | Aurora capacity exceeded | Add read replicas (2-3) | +$250 |
| **5,000/day** | Application compute | ECS CPU usage > 80% | Increase task count to 15+ | +$300 |
| **10,000/day** | LLM API throughput | Need 400K+ TPM (OpenAI max 200K) | Multi-region OpenAI or Bedrock | +$40K |

---

## BOTTLENECK #1: LLM API RATE LIMITS (PRIMARY)

### The Problem
LLM APIs have strict rate limiting that becomes the hard cap on forms/day.

### Rate Limit Constraints

**OpenAI:**
- Free tier: 3 RPM (requests/minute), 90 TPM (tokens/minute)
- Paid tier: 3,500 RPM, 200,000 TPM
- Cost: $0.50/form (GPT-4)

**Claude (Anthropic):**
- Free API: 10 RPM, 100K TPM
- Paid API: 100+ RPM (can request higher), 100K+ TPM
- Cost: $0.15/form (3.5 Sonnet)

**AWS Bedrock:**
- Provisioned throughput: Unlimited RPM, fixed TPM
- Cost: $1.50/hour per 1M input tokens
- Advantage: No per-request limits

### Forms/Day Limits by API Configuration

```
OpenAI Free (90 TPM):
├─ 1,000 tokens/form
├─ 90,000 TPM ÷ 1,000 = 90 forms/hour
├─ 90 × 24 hours = 2,160 forms/day MAX
└─ Reality: ~50 forms/day (with other traffic)

OpenAI Paid (200K TPM):
├─ 1,000 tokens/form
├─ 200,000 TPM ÷ 1,000 = 200 forms/hour
├─ 200 × 24 hours = 4,800 forms/day MAX
└─ Reality: ~1,500 forms/day (realistic load)

Claude Batch (Unlimited):
├─ 1,000 tokens/form
├─ Can process 50K+ tokens per batch
├─ 8-24 hour processing window
├─ Can handle 10,000+ forms/day with queue
└─ Cost: 50% less than real-time API

AWS Bedrock Provisioned (3M TPM):
├─ 1,000 tokens/form
├─ 3,000,000 TPM ÷ 1,000 = 3,000 forms/hour
├─ 3,000 × 24 hours = 72,000 forms/day MAX
└─ Cost: $4,500/month (fixed, regardless of usage)
```

### How to Handle Each Scale

**At 100 forms/day:**
- Use OpenAI free API tier
- Cost: $0 (free tier credit)
- Constraint: None (only 50 forms/day hitting API)

**At 500 forms/day:**
- Upgrade to OpenAI paid tier ($15/month)
- Use batch processing for 50% of forms (reduce rate limit pressure)
- Cost: $250/month
- Latency: Real-time for 50%, 8-16 hours for 50%

**At 1,000 forms/day:**
- Use Claude batch API for 70% ($630/month)
- Use Claude real-time for 30% ($2,000/month)
- OR: Use AWS Bedrock provisioned throughput ($3,000/month)
- Cost: $2,600/month (batch + real-time)
- Latency: Real-time for interactive, batch for background jobs

**At 10,000 forms/day:**
- Use AWS Bedrock provisioned throughput (unlimited TPM)
- OR: Multi-region OpenAI accounts (3x rate limit)
- Cost: $4,500/month (Bedrock is cheaper)
- Latency: < 5 seconds guaranteed

### Cost Per Form at Each API Choice

| Choice | Token Cost | Per-Form Cost | Best For | Caveat |
|--------|-----------|--------------|----------|--------|
| OpenAI GPT-4 | $0.50 | $0.50/form | Real-time guidance | Rate limit at 1,500/day |
| Claude Real-time | $0.15 | $0.15/form | Real-time guidance | Rate limit at 3,000/day |
| Claude Batch | $0.08 | $0.08/form | Async validation | 8-24 hour latency |
| Bedrock Provisioned | $1.50/hour | $0.03/form | High-volume (10K+) | $4,500/mo fixed |

### Mitigation Strategy: 3-Tier LLM Architecture

```
User submits form
│
├─ Real-time (Interactive guidance): Claude 3.5 Sonnet
│  └─ Instant feedback on field
│  └─ Rate limit: 100 concurrent users
│  └─ Cost: $0.15/form
│
├─ Batch (Background validation): Claude Batch API
│  └─ Check form overnight
│  └─ Flag inconsistencies
│  └─ Cost: $0.08/form
│
└─ Fallback (If rate limited): Pre-cached responses
   └─ Serve from database
   └─ No API call
   └─ Cost: $0
```

### Action Items by Scale

**At 150 forms/day:**
- [ ] Upgrade to OpenAI paid tier ($15/month)
- [ ] Add request queue (SQS)
- [ ] Implement retry logic with exponential backoff
- [ ] Set up rate limit alerts

**At 500 forms/day:**
- [ ] Switch to 70% batch, 30% real-time
- [ ] Implement Claude batch processing pipeline
- [ ] Add caching for common Q&A responses
- [ ] Set up CloudWatch alerts for TPM usage

**At 1,000 forms/day:**
- [ ] Monitor OpenAI TPM usage daily
- [ ] Consider AWS Bedrock provisioned throughput
- [ ] Implement client-side caching (Redux)
- [ ] Set budget alerts at 80% usage

**At 10,000+ forms/day:**
- [ ] Migrate to AWS Bedrock provisioned throughput
- [ ] Implement multi-region failover
- [ ] Cache 80% of responses locally
- [ ] Use Bedrock with SageMaker endpoint

---

## BOTTLENECK #2: PDF GENERATION (SECONDARY)

### The Problem
PDFs must be generated sequentially; concurrent generation can exhaust Lambda/container memory.

### PDF Generation Capacity by Tool

| Tool | Speed | Memory | Concurrency | Suited For | Cost |
|------|-------|--------|-------------|-----------|------|
| **PDFKit (Node.js)** | 100 PDFs/s | 50MB/PDF | 10-20 concurrent | Vercel/small AWS | Free |
| **LibreOffice + Lambda** | 10-20 PDFs/s | 500MB/PDF | 2-5 concurrent | Medium scale | Included |
| **ECS Fargate Cluster** | 50-100 PDFs/s | 100MB/PDF | 50-100 concurrent | Large scale | $300-500/mo |
| **AWS Bedrock Titan PDF** | 200+ PDFs/s | 20MB/PDF | Unlimited | Enterprise | $1-5/PDF |

### Forms/Day Limits by PDF Tool

```
PDFKit (Vercel):
├─ 1 container, 512MB memory
├─ 1 PDF at 3 seconds (worst case)
├─ 1 PDF/3 sec = 20 PDFs/minute
├─ 20 × 60 × 24 hours = 28,800 PDFs/day
└─ Reality: ~500 forms/day (concurrent limitations)

Lambda (AWS):
├─ Default concurrency: 1,000 simultaneous
├─ Each Lambda: 1GB memory, 1 vCPU
├─ 1 PDF at 3 seconds, sequential
├─ 1,000 concurrent × 20 = 20,000 PDFs/minute
└─ Reality: ~300,000 forms/day

ECS Fargate (AWS):
├─ 10 tasks, 2 vCPU each, 4GB memory
├─ 10 containers × 3 PDFs/second = 30 PDFs/second
├─ 30 × 60 × 24 = 2,592,000 PDFs/day
└─ Reality: ~500,000 forms/day
```

### PDF Generation Latency by Scale

| Scale | P50 | P95 | P99 | Max |
|-------|-----|-----|-----|-----|
| **100/day** | 1s | 2s | 3s | 5s |
| **500/day** | 2s | 4s | 6s | 10s |
| **1,000/day** | 3s | 5s | 8s | 12s |
| **5,000/day** | 2s | 3s | 4s | 6s |
| **10,000/day** | 2s | 3s | 4s | 5s |

**Note:** As scale increases, latency DECREASES due to better load distribution across containers.

### How to Optimize PDF Generation

**1. Use PDFKit (cheapest, fastest for small scale)**
```javascript
const PDFDocument = require('pdfkit');

function generateFormPDF(formData) {
  const doc = new PDFDocument();

  // Add form fields (1-2 seconds per PDF)
  formData.fields.forEach(field => {
    doc.text(field.value, field.x, field.y);
  });

  return doc.pipe(fs.createWriteStream(`/tmp/${formData.id}.pdf`));
}
```

**2. Implement PDF Queue (reduce memory spikes)**
```javascript
// Queue PDFs if too many concurrent
const pdfQueue = new Queue('pdf-generation', redisURL);

pdfQueue.add({ form_id, user_id }, {
  delay: 0,
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 2000
  }
});

// Process max 5 PDFs concurrently
pdfQueue.process(5, async (job) => {
  return generatePDF(job.data);
});
```

**3. Cache PDFs (60-80% hit rate)**
```javascript
// Check cache first
const cachedPDF = await s3.getObject({
  Bucket: 'pdf-cache',
  Key: `${form_id}.pdf`
});

if (cachedPDF) {
  return cachedPDF; // Instant delivery
}

// Generate if not cached
const pdf = await generatePDF(formData);
await s3.putObject({
  Bucket: 'pdf-cache',
  Key: `${form_id}.pdf`,
  Body: pdf
});
```

**4. Compress PDFs (reduce storage cost by 40%)**
```javascript
const compress = require('pdf-compressor');

const originalSize = fs.statSync(`${form_id}.pdf`).size;
const compressed = await compress(`${form_id}.pdf`);
const compressedSize = fs.statSync(`${form_id}.pdf`).size;

console.log(`Compression: ${originalSize}B → ${compressedSize}B`);
// Typical: 500KB → 300KB (40% reduction)
```

### When to Upgrade PDF Infrastructure

**At 100 forms/day:**
- Use PDFKit in-process (Vercel/Lambda)
- Cost: $0

**At 500 forms/day:**
- Keep PDFKit, add queue (SQS)
- Implement caching (S3)
- Cost: $20

**At 1,000 forms/day:**
- Migrate to ECS Fargate (3 tasks)
- Implement Redis cache
- Cost: $150

**At 5,000+ forms/day:**
- Scale ECS to 10+ tasks
- Consider AWS Bedrock Titan PDF (if many documents)
- Cost: $500-1,000

### Action Items by Scale

**At 500 forms/day:**
- [ ] Implement SQS queue for PDF generation
- [ ] Add S3 caching for PDFs
- [ ] Monitor Lambda memory usage
- [ ] Set up CloudWatch alarms for queue depth

**At 1,000 forms/day:**
- [ ] Migrate to ECS Fargate
- [ ] Implement Redis cache for templates
- [ ] Add PDF compression
- [ ] Monitor P99 latency (target < 8s)

**At 5,000+ forms/day:**
- [ ] Scale ECS to 10+ tasks
- [ ] Implement PDF pre-generation (queue overnight)
- [ ] Use CloudFront cache for downloads
- [ ] Monitor cost per PDF

---

## BOTTLENECK #3: DATABASE I/O (TERTIARY)

### The Problem
PostgreSQL has limited IOPS (I/O operations per second) at smaller instance sizes.

### Database Capacity by Instance Type

| Instance | vCPU | Memory | Max IOPS | Connections | Cost/mo |
|----------|------|--------|----------|------------|---------|
| **db.t3.micro** | 1 | 1GB | 100 | 20 | $11 |
| **db.t4g.small** | 1 | 2GB | 400 | 100 | $60 |
| **db.r6g.large** | 2 | 16GB | 2,000 | 500 | $300 |
| **db.r6g.2xlarge** | 8 | 64GB | 6,000 | 2,000 | $1,200 |

### Forms/Day Limits by Database

```
db.t3.micro (100 IOPS):
├─ 65 I/O per form (60 fields + validation)
├─ 100 IOPS ÷ 65 = 1.5 forms/second
├─ 1.5 × 60 × 60 × 24 = 129,600 forms/day MAX
└─ Reality: ~50 forms/day (with connection limits)

db.t4g.small (400 IOPS):
├─ 65 I/O per form
├─ 400 IOPS ÷ 65 = 6 forms/second
├─ 6 × 60 × 60 × 24 = 518,400 forms/day MAX
└─ Reality: ~500 forms/day (hitting connection limit at 100 connections)

db.r6g.large (2,000 IOPS):
├─ 65 I/O per form
├─ 2,000 IOPS ÷ 65 = 30 forms/second
├─ 30 × 60 × 60 × 24 = 2,592,000 forms/day MAX
└─ Reality: ~2,000 forms/day (connection pool management)

db.r6g.2xlarge (6,000 IOPS):
├─ 65 I/O per form
├─ 6,000 IOPS ÷ 65 = 92 forms/second
├─ 92 × 60 × 60 × 24 = 7,948,800 forms/day MAX
└─ Reality: ~5,000+ forms/day (compute becomes bottleneck)
```

### Database Query Performance by Scale

| Scale | Avg Query Time | P99 Query Time | Bottleneck |
|-------|---|---|---|
| **100/day** | 10ms | 50ms | Connection pool |
| **500/day** | 20ms | 100ms | IOPS |
| **1,000/day** | 30ms | 150ms | IOPS + CPU |
| **5,000/day** | 50ms | 200ms | CPU + connection pool |
| **10,000/day** | 100ms | 500ms | Storage I/O |

### How to Optimize Database Performance

**1. Add Connection Pooling (RDS Proxy)**
```
Without proxy: 1 connection per user (100-1,000 connections needed)
With proxy: Connection pooling (20-50 connections max)
Cost: $30/month
Benefit: 80% reduction in connection overhead
```

**2. Implement Read Replicas (for read-heavy workloads)**
```
Write: Primary database (100% of writes)
Read: Replica 1 + Replica 2 (100% of reads split)
Cost: +$150/month per replica
Benefit: 3x read throughput
```

**3. Add Database Indexes (huge impact, free)**
```sql
-- Index on most-queried fields
CREATE INDEX idx_form_user_id ON forms(user_id);
CREATE INDEX idx_form_status ON forms(status);
CREATE INDEX idx_form_created_at ON forms(created_at DESC);

-- Results: 10x faster queries
-- Before: 100ms per query
-- After: 10ms per query
```

**4. Archive Old Data (reduce database size)**
```
Keep: Forms < 1 year in PostgreSQL (hot storage)
Archive: Forms > 1 year in S3 Glacier ($0.004/GB/month)
Result: Database stays small (faster queries, smaller backups)
```

**5. Use Read-Only Replicas for Analytics**
```
OLTP Workload: Primary database (form submissions)
OLAP Workload: Analytics replica (dashboards, reports)
Cost: +$100/month
Benefit: Production database unaffected by report queries
```

### When to Upgrade Database

**At 50 forms/day:**
- Use db.t3.micro (free tier, 1 year)
- Add basic indexing
- Cost: $0 (AWS free tier)

**At 150 forms/day:**
- Upgrade to db.t4g.small
- Add RDS Proxy
- Cost: $90/month

**At 500 forms/day:**
- Keep db.t4g.small
- Add 1 read replica
- Cost: $150/month

**At 1,000 forms/day:**
- Upgrade to db.r6g.large
- Add 2 read replicas
- Cost: $600/month

**At 5,000+ forms/day:**
- Upgrade to db.r6g.2xlarge
- Add 3+ read replicas
- Consider sharding (3-5 database instances)
- Cost: $3,000+/month

### Action Items by Scale

**At 200 forms/day:**
- [ ] Create indexes on user_id, form_id, status
- [ ] Enable slow query logging (queries > 100ms)
- [ ] Monitor database CPU (should be < 30%)
- [ ] Set up CloudWatch alarms for IOPS usage

**At 500 forms/day:**
- [ ] Implement RDS Proxy
- [ ] Add read replica for analytics
- [ ] Monitor connection pool saturation
- [ ] Implement connection retry logic

**At 1,000 forms/day:**
- [ ] Upgrade to db.r6g.large
- [ ] Add 2 read replicas
- [ ] Implement query result caching (Redis)
- [ ] Monitor P99 latency (target < 150ms)

**At 5,000+ forms/day:**
- [ ] Implement database sharding (3-5 shards)
- [ ] Use DynamoDB for sessions (cheaper, faster)
- [ ] Archive forms > 1 year to S3
- [ ] Implement distributed tracing (X-Ray)

---

## BOTTLENECK #4: CONCURRENT USERS (SECONDARY)

### The Problem
Vercel/Lambda have connection limits that can be exceeded under high concurrency.

### Concurrency Limits by Platform

| Platform | Concurrent Users | Connection Limit | Scaling |
|----------|---|---|---|
| **Vercel Serverless** | 100-200 | 500 | Manual |
| **Lambda** | 1,000 | 1,000 per container | Auto-scales |
| **ECS Fargate** | 10,000+ | 20,000+ | Auto-scales |
| **EC2 with ALB** | 50,000+ | 100,000+ | Manual capacity planning |

### User Concurrency by Scale

```
100 forms/day:
├─ Peak forms/hour: ~10
├─ Avg form time: 15 minutes
├─ Concurrent users: 10 × 15/60 = 2.5 users
├─ Platform: Vercel (handles 100+)
└─ Overhead: None

500 forms/day:
├─ Peak forms/hour: ~25
├─ Concurrent users: 25 × 15/60 = 6.25 users
├─ Platform: Vercel (handles 100+)
└─ Overhead: None

1,000 forms/day:
├─ Peak forms/hour: ~50
├─ Concurrent users: 50 × 15/60 = 12.5 users
├─ Platform: Vercel/Lambda (handles 100+)
└─ Overhead: None

5,000 forms/day:
├─ Peak forms/hour: ~250
├─ Concurrent users: 250 × 15/60 = 62.5 users
├─ Platform: Lambda/ECS (handles 1,000)
└─ Overhead: Connection pooling

10,000 forms/day:
├─ Peak forms/hour: ~500
├─ Concurrent users: 500 × 15/60 = 125 users
├─ Platform: ECS (handles 10,000+)
└─ Overhead: RDS Proxy required
```

### How to Handle Concurrency

**1. Implement Connection Pooling**
```javascript
// Without pooling: 1 connection per user
// With pooling: Reuse connections (20-50 max)

const pool = new Pool({
  host: 'db.example.com',
  user: 'postgres',
  password: 'password',
  database: 'forms',
  max: 50, // Max connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

const result = await pool.query('SELECT * FROM forms WHERE id = $1', [formId]);
```

**2. Use RDS Proxy (automatic connection pooling)**
```
Benefits:
- 1,000+ connection support with 50 backend connections
- Automatic failover
- Query result caching
- Cost: $30-50/month
```

**3. Queue Long-Running Operations**
```javascript
// Don't process PDFs in request (blocks connection)
// Queue for async processing

app.post('/api/form/submit', async (req, res) => {
  // Save form quickly (< 1 second)
  await saveForm(req.body);

  // Queue PDF generation for later
  await pdfQueue.add({ form_id: form.id });

  res.json({ success: true });
});

// Process queue in background (doesn't block users)
pdfQueue.process(10, async (job) => {
  return generatePDF(job.data);
});
```

### Action Items by Scale

**At 100 forms/day:**
- [ ] No action needed (concurrency is minimal)

**At 500 forms/day:**
- [ ] Monitor concurrent connections (should be < 10)
- [ ] Implement request queueing
- [ ] Set up CloudWatch alarms

**At 1,000 forms/day:**
- [ ] Implement RDS Proxy
- [ ] Increase connection pool size to 50
- [ ] Monitor connection pool saturation
- [ ] Set up auto-scaling triggers

**At 5,000+ forms/day:**
- [ ] Scale to ECS (auto-scaling handles concurrency)
- [ ] Implement distributed session management (DynamoDB)
- [ ] Use RDS Proxy for database connections
- [ ] Monitor connection pool metrics continuously

---

## QUICK REFERENCE: WHEN TO SCALE

### Scale from Vercel → AWS Medium
**Trigger: 150 forms/day consistently for 1 week**

Symptoms:
- Redis memory > 800MB
- Lambda timeout errors (> 5% of requests)
- OpenAI rate limit errors (> 1% of requests)
- P99 latency > 10 seconds

Action:
1. Migrate to AWS Medium (RDS + ECS + ALB)
2. Keep Vercel as backup (blue-green)
3. Monitor for 1 week
4. Decommission Vercel

Effort: 40-60 hours
Cost during migration: $2,000
Downtime: < 1 hour

### Scale from AWS Medium → AWS Large
**Trigger: 2,000 forms/day consistently for 1 week**

Symptoms:
- Database CPU > 80%
- Lambda concurrency limit hit
- OpenAI TPM limit exceeded
- P99 latency > 8 seconds

Action:
1. Migrate to AWS Large (multi-region Aurora + ECS + Lambda)
2. Set up RDS Global Database
3. Route traffic to both regions (weighted)
4. Optimize per-region resources

Effort: 50-60 hours
Cost during migration: $3,000
Downtime: < 30 minutes

---

## PRIORITY OPTIMIZATIONS BY ROI

### High ROI (Do First)
1. **LLM API caching** (saves 70% of API costs)
   - Effort: 5 hours
   - Savings: $5,000-10,000/month at 10K scale
   - ROI: 100,000%

2. **Database indexing** (10x faster queries)
   - Effort: 2 hours
   - Savings: $500-1,000/month (smaller database)
   - ROI: 50,000%

3. **PDF caching** (80% hit rate)
   - Effort: 3 hours
   - Savings: $200-500/month (less S3 traffic)
   - ROI: 20,000%

### Medium ROI (Do Second)
4. **Connection pooling** (reduce overhead)
   - Effort: 3 hours
   - Savings: $100-300/month (less database CPU)
   - ROI: 5,000%

5. **Read replicas** (offload reads)
   - Effort: 5 hours
   - Savings: $50-200/month (smaller primary DB)
   - ROI: 3,000%

### Low ROI (Do Last)
6. **CloudFront caching** (edge delivery)
   - Effort: 4 hours
   - Savings: $10-50/month
   - ROI: 500%

7. **Compression** (smaller file sizes)
   - Effort: 2 hours
   - Savings: $5-20/month
   - ROI: 300%

---

## FINAL CHECKLIST

### Vercel/Railway Readiness
- [ ] LLM API integrated (OpenAI or Claude)
- [ ] Redis queue working (async jobs)
- [ ] PDF generation functional
- [ ] Payment processing (Stripe)
- [ ] Error monitoring (Sentry)
- [ ] Load testing done (100+ concurrent users)

### AWS Medium Readiness
- [ ] ECS task definition ready
- [ ] RDS Aurora provisioned
- [ ] Auto-scaling configured
- [ ] CloudWatch alarms set up
- [ ] Backup strategy tested
- [ ] Load testing passed (1,000 concurrent users)

### AWS Large Readiness
- [ ] Multi-region setup verified
- [ ] RDS Global Database replicating
- [ ] Route53 failover tested
- [ ] Aurora auto-scaling tested
- [ ] Disaster recovery plan documented
- [ ] Load testing passed (10,000 concurrent users)

---

## RECOMMENDED READING

1. **LLM API Optimization**
   - OpenAI Rate Limit Guide: https://platform.openai.com/docs/guides/rate-limits
   - Claude Batch Processing: https://docs.anthropic.com/en/docs/build/batch-processing-guide
   - AWS Bedrock Throughput: https://docs.aws.amazon.com/bedrock/latest/userguide/provisioned-throughput.html

2. **Database Performance**
   - PostgreSQL Indexing: https://www.postgresql.org/docs/current/sql-createindex.html
   - RDS Aurora Auto-Scaling: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Aurora.Managing.Performance.html
   - RDS Proxy Guide: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html

3. **AWS Scaling**
   - ECS Auto-Scaling: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html
   - Lambda Concurrency: https://docs.aws.amazon.com/lambda/latest/dg/concurrency.html
   - CloudFront Caching: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cache-hit-ratio.html

---

## CONTACT & ESCALATION

**If bottleneck is unclear:**
1. Check CloudWatch metrics (CPU, memory, IOPS, connections)
2. Look at error logs (timeout, rate limit, connection pool exhausted)
3. Review application metrics (request latency, queue depth, cache hit rate)
4. Compare actual vs. projected numbers above

**If cost is higher than expected:**
1. Check LLM API usage (usually 60-80% of costs)
2. Review database instance size (may be oversized)
3. Look for unused resources (old Lambda functions, S3 buckets)
4. Enable CloudWatch Cost Anomaly Detection

**If latency is degrading:**
1. Check database query times (index missing?)
2. Look at Lambda cold start rate (provisioned concurrency?)
3. Monitor cache hit rates (PDFs, LLM responses)
4. Check for database connection pool exhaustion
