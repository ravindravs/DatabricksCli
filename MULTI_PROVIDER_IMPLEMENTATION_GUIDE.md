# Multi-Provider LLM Implementation Guide
## Technical Reference for API Abstraction Layer

---

## PART 1: PROVIDER ABSTRACTION LAYER (PAL)

### High-Level Architecture

```
┌──────────────────────────────┐
│   Frontend (React)           │
│   Form Q&A Interface         │
└──────────────┬───────────────┘
               │
               ↓
┌──────────────────────────────────────────┐
│   LLM Provider Abstraction Layer (PAL)   │
│                                          │
│   - Route request to best provider      │
│   - Health check before sending         │
│   - Automatic failover on error         │
│   - Log all provider interactions       │
│   - Track cost per query                │
└──────────────┬───────────────────────────┘
       ┌───────┼────────┬──────────┐
       ↓       ↓        ↓          ↓
   ┌────────┐┌────────┐┌────────┐┌──────────┐
   │ Claude ││ OpenAI ││Llama 2 ││ Fallback │
   │API Key ││API Key ││(Local) ││(Rules)   │
   └────────┘└────────┘└────────┘└──────────┘
```

### Core Functions

**1. Provider Router (Intelligent Dispatch)**

```typescript
// llm-provider-abstraction.ts

enum Provider {
  CLAUDE = 'claude',
  OPENAI = 'openai',
  LLAMA2 = 'llama2',
  FALLBACK = 'fallback'
}

interface LLMRequest {
  prompt: string;
  type: 'explanation' | 'validation' | 'risk-assessment';
  fallbackOnError: boolean;
  timeout: number; // ms
}

interface LLMResponse {
  text: string;
  provider: Provider;
  latency: number;
  cost: number;
  timestamp: Date;
}

class LLMProviderRouter {
  private providerHealth = new Map<Provider, ProviderStatus>();
  private costTracker = new Map<Provider, CostMetrics>();
  private cache = new LRUCache<string, CachedResponse>(1000);

  async route(request: LLMRequest): Promise<LLMResponse> {
    // 1. Check cache first
    const cached = this.cache.get(request.prompt);
    if (cached) {
      return { ...cached, provider: Provider.FALLBACK };
    }

    // 2. Get best available provider
    const provider = await this.selectProvider(request.type);

    // 3. Send request with timeout
    try {
      const response = await this.callProvider(provider, request);
      this.recordSuccess(provider, request, response);
      return response;
    } catch (error) {
      // 4. Fallback logic
      if (request.fallbackOnError) {
        return await this.failover(request);
      }
      throw error;
    }
  }

  private async selectProvider(type: string): Promise<Provider> {
    // Priority: Claude > OpenAI > Llama2 > Fallback
    // But check health, latency, and cost first

    const healthy = Array.from(this.providerHealth.entries())
      .filter(([_, status]) => status.isHealthy)
      .map(([provider, _]) => provider);

    if (healthy.includes(Provider.CLAUDE)) {
      return Provider.CLAUDE; // Primary (lowest cost + best quality)
    }
    if (healthy.includes(Provider.OPENAI)) {
      return Provider.OPENAI; // Secondary (higher cost, good quality)
    }
    if (healthy.includes(Provider.LLAMA2)) {
      return Provider.LLAMA2; // Tertiary (free, local, slower)
    }
    return Provider.FALLBACK; // Last resort (template-based)
  }

  private async callProvider(
    provider: Provider,
    request: LLMRequest
  ): Promise<LLMResponse> {
    const startTime = performance.now();

    switch (provider) {
      case Provider.CLAUDE:
        return await this.callClaude(request, startTime);
      case Provider.OPENAI:
        return await this.callOpenAI(request, startTime);
      case Provider.LLAMA2:
        return await this.callLlama2(request, startTime);
      case Provider.FALLBACK:
        return await this.callFallback(request, startTime);
    }
  }

  private async callClaude(
    request: LLMRequest,
    startTime: number
  ): Promise<LLMResponse> {
    const client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const response = await client.messages.create(
      {
        model: 'claude-3-sonnet-20240229',
        max_tokens: 1024,
        system: IMMIGRATION_FORM_SYSTEM_PROMPT,
        messages: [{ role: 'user', content: request.prompt }],
      },
      { timeout: request.timeout }
    );

    const latency = performance.now() - startTime;

    // Track cost: Anthropic pricing
    // Input: $3/MTok, Output: $15/MTok
    const inputTokens = request.prompt.split(' ').length * 1.3; // estimate
    const outputTokens =
      response.content[0].type === 'text'
        ? response.content[0].text.split(' ').length * 1.3
        : 0;
    const cost = (inputTokens / 1_000_000) * 3 + (outputTokens / 1_000_000) * 15;

    return {
      text:
        response.content[0].type === 'text' ? response.content[0].text : '',
      provider: Provider.CLAUDE,
      latency,
      cost,
      timestamp: new Date(),
    };
  }

  private async callOpenAI(
    request: LLMRequest,
    startTime: number
  ): Promise<LLMResponse> {
    const client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });

    const response = await client.chat.completions.create({
      model: 'gpt-4',
      messages: [
        {
          role: 'system',
          content: IMMIGRATION_FORM_SYSTEM_PROMPT,
        },
        { role: 'user', content: request.prompt },
      ],
      max_tokens: 1024,
      timeout: request.timeout,
    });

    const latency = performance.now() - startTime;

    // Track cost: OpenAI GPT-4 pricing
    // Input: $0.03/1K, Output: $0.06/1K
    const inputTokens = request.prompt.split(' ').length * 1.3;
    const outputTokens = response.choices[0].message.content.split(' ').length * 1.3;
    const cost = (inputTokens / 1000) * 0.03 + (outputTokens / 1000) * 0.06;

    return {
      text: response.choices[0].message.content || '',
      provider: Provider.OPENAI,
      latency,
      cost,
      timestamp: new Date(),
    };
  }

  private async callLlama2(
    request: LLMRequest,
    startTime: number
  ): Promise<LLMResponse> {
    // Llama2 running locally (replicate.com API or self-hosted)
    const response = await fetch(
      process.env.LLAMA2_ENDPOINT || 'http://localhost:8000/v1/completions',
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          prompt: request.prompt,
          max_tokens: 1024,
          temperature: 0.3,
        }),
      }
    );

    const data = await response.json();
    const latency = performance.now() - startTime;

    // Llama2 cost: $0 (self-hosted) or $0.0001/token (replicate)
    const cost = 0;

    return {
      text: data.choices[0].text || '',
      provider: Provider.LLAMA2,
      latency,
      cost,
      timestamp: new Date(),
    };
  }

  private async callFallback(
    request: LLMRequest,
    startTime: number
  ): Promise<LLMResponse> {
    // Template-based response (no API call)
    const template = FALLBACK_TEMPLATES[request.type] || 'Unable to generate response.';
    const latency = performance.now() - startTime;

    return {
      text: template,
      provider: Provider.FALLBACK,
      latency,
      cost: 0,
      timestamp: new Date(),
    };
  }

  private async failover(request: LLMRequest): Promise<LLMResponse> {
    // Try providers in order: Claude → OpenAI → Llama2 → Fallback
    const providers = [
      Provider.CLAUDE,
      Provider.OPENAI,
      Provider.LLAMA2,
      Provider.FALLBACK,
    ];

    for (const provider of providers) {
      try {
        return await this.callProvider(provider, {
          ...request,
          timeout: 2000, // Shorter timeout for failover attempts
        });
      } catch (error) {
        logger.warn(`Provider ${provider} failed, trying next`, { error });
        continue;
      }
    }

    // All providers failed, return degraded response
    return {
      text: 'The form assistant is temporarily unavailable. Please try again in a few minutes.',
      provider: Provider.FALLBACK,
      latency: 0,
      cost: 0,
      timestamp: new Date(),
    };
  }

  private recordSuccess(
    provider: Provider,
    request: LLMRequest,
    response: LLMResponse
  ): void {
    // Update health metrics
    this.providerHealth.set(provider, {
      isHealthy: true,
      lastCheckTime: new Date(),
      consecutiveFailures: 0,
      averageLatency: this.calculateRunningAverage(
        this.providerHealth.get(provider)?.averageLatency || 0,
        response.latency
      ),
    });

    // Track cost
    const metrics = this.costTracker.get(provider) || {
      totalCost: 0,
      totalQueries: 0,
    };
    metrics.totalCost += response.cost;
    metrics.totalQueries += 1;
    this.costTracker.set(provider, metrics);

    // Log for audit
    logger.info('LLM request succeeded', {
      provider,
      latency: response.latency,
      cost: response.cost,
      type: request.type,
    });
  }

  async healthCheck(): Promise<void> {
    // Run every 60 seconds
    for (const provider of [
      Provider.CLAUDE,
      Provider.OPENAI,
      Provider.LLAMA2,
    ]) {
      try {
        const startTime = performance.now();
        await this.callProvider(provider, {
          prompt: 'Hello',
          type: 'explanation',
          fallbackOnError: false,
          timeout: 5000,
        });

        const latency = performance.now() - startTime;
        this.providerHealth.set(provider, {
          isHealthy: true,
          lastCheckTime: new Date(),
          consecutiveFailures: 0,
          averageLatency: latency,
        });
      } catch (error) {
        const status = this.providerHealth.get(provider);
        const failures = (status?.consecutiveFailures || 0) + 1;

        this.providerHealth.set(provider, {
          isHealthy: failures < 3, // Mark unhealthy after 3 failures
          lastCheckTime: new Date(),
          consecutiveFailures: failures,
          averageLatency: status?.averageLatency || 0,
        });

        if (failures >= 3) {
          logger.error(`Provider ${provider} marked unhealthy`, {
            consecutiveFailures: failures,
          });
          // Alert on-call engineer
          await this.sendAlert(`Provider ${provider} is down`);
        }
      }
    }
  }
}
```

---

## PART 2: COST OPTIMIZATION STRATEGIES

### Strategy 1: Response Caching

**Goal:** Reduce API calls by 40-60%

```typescript
// response-cache.ts

interface CachedResponse {
  text: string;
  provider: Provider;
  timestamp: Date;
  hitCount: number;
}

class ResponseCache {
  private cache = new Map<string, CachedResponse>();
  private maxSize = 5000;

  // Generate cache key from question + form context
  private getCacheKey(prompt: string, formType: string): string {
    // Hash question (normalize whitespace)
    const normalized = prompt.trim().toLowerCase();
    return `${formType}:${sha256(normalized)}`;
  }

  async getOrFetch(
    prompt: string,
    formType: string,
    fetchFn: () => Promise<LLMResponse>
  ): Promise<LLMResponse> {
    const key = this.getCacheKey(prompt, formType);

    // Check cache first
    if (this.cache.has(key)) {
      const cached = this.cache.get(key)!;
      cached.hitCount++;

      // Refresh if older than 24 hours
      const age = Date.now() - cached.timestamp.getTime();
      if (age < 24 * 60 * 60 * 1000) {
        logger.info('Cache hit', { key, hitCount: cached.hitCount });
        return {
          ...cached,
          timestamp: new Date(),
        };
      }
    }

    // Cache miss or expired, fetch from provider
    const response = await fetchFn();

    // Store in cache (with size limit)
    if (this.cache.size >= this.maxSize) {
      // Remove least-hit item (LRU)
      const leastHit = Array.from(this.cache.values()).sort(
        (a, b) => a.hitCount - b.hitCount
      )[0];
      this.cache.delete(
        Array.from(this.cache.entries()).find(
          ([_, v]) => v === leastHit
        )?.[0] || ''
      );
    }

    this.cache.set(key, {
      text: response.text,
      provider: response.provider,
      timestamp: response.timestamp,
      hitCount: 1,
    });

    logger.info('Cache miss, stored', { key });
    return response;
  }

  // Pre-compute common questions offline
  async preWarm(questions: string[], formType: string): Promise<void> {
    logger.info('Pre-warming cache', { count: questions.length });

    for (const question of questions) {
      const key = this.getCacheKey(question, formType);
      if (!this.cache.has(key)) {
        // Fetch during off-peak hours (batch job)
        const response = await claudeClient.messages.create({
          model: 'claude-3-sonnet-20240229',
          max_tokens: 1024,
          messages: [{ role: 'user', content: question }],
        });

        this.cache.set(key, {
          text: response.content[0].type === 'text' ? response.content[0].text : '',
          provider: Provider.CLAUDE,
          timestamp: new Date(),
          hitCount: 0,
        });
      }
    }
  }

  // Analytics
  getCacheStats(): {
    size: number;
    hitRate: number;
    topQueries: string[];
  } {
    const entries = Array.from(this.cache.values());
    const totalHits = entries.reduce((sum, e) => sum + e.hitCount, 0);
    const hitRate = totalHits / entries.length;

    const topQueries = entries
      .sort((a, b) => b.hitCount - a.hitCount)
      .slice(0, 10)
      .map((e) => e.text);

    return {
      size: this.cache.size,
      hitRate,
      topQueries,
    };
  }
}
```

### Strategy 2: Request Batching

**Goal:** Combine multiple requests into single API call

```typescript
// batch-processor.ts

class BatchProcessor {
  private queue: Array<{
    request: LLMRequest;
    resolve: (response: LLMResponse) => void;
    reject: (error: Error) => void;
  }> = [];

  private batchSize = 10;
  private batchTimeout = 100; // ms

  async add(request: LLMRequest): Promise<LLMResponse> {
    return new Promise((resolve, reject) => {
      this.queue.push({ request, resolve, reject });

      // Flush if batch full
      if (this.queue.length >= this.batchSize) {
        this.flush();
      }

      // Or flush after timeout
      setTimeout(() => {
        if (this.queue.length > 0) {
          this.flush();
        }
      }, this.batchTimeout);
    });
  }

  private async flush(): Promise<void> {
    if (this.queue.length === 0) return;

    const batch = this.queue.splice(0, this.batchSize);
    const prompts = batch.map((b) => b.request.prompt);

    try {
      // Send all prompts in single API call (if provider supports batch)
      const response = await claudeClient.messages.create({
        model: 'claude-3-sonnet-20240229',
        max_tokens: 4096, // Larger for batch
        messages: [
          {
            role: 'user',
            content: `Respond to the following ${prompts.length} questions concisely:\n\n${prompts.map((p, i) => `${i + 1}. ${p}`).join('\n\n')}`,
          },
        ],
      });

      // Parse response and distribute to callers
      const responseText =
        response.content[0].type === 'text' ? response.content[0].text : '';
      const answers = this.parseResponses(responseText, batch.length);

      batch.forEach((item, index) => {
        item.resolve({
          text: answers[index],
          provider: Provider.CLAUDE,
          latency: 0,
          cost: 0,
          timestamp: new Date(),
        });
      });
    } catch (error) {
      batch.forEach((item) => {
        item.reject(error as Error);
      });
    }
  }

  private parseResponses(text: string, count: number): string[] {
    // Split response by numbered sections
    return text.split(/\n(?=\d+\.)/);
  }
}
```

---

## PART 3: FALLBACK TEMPLATE SYSTEM

### Pre-Computed Responses (Zero API Cost)

```typescript
// fallback-templates.ts

const FALLBACK_TEMPLATES: Record<string, string> = {
  // I-130 Form Field Explanations
  'i-130_relationship': `
    <strong>Relationship to Petitioner</strong>

    This field asks about your family relationship to the person filing this petition.
    Common values:
    - Spouse: You are legally married to the petitioner
    - Parent: The petitioner is your parent
    - Child: The petitioner is your parent
    - Brother/Sister: The petitioner is your sibling

    Enter the relationship exactly as it appears on your vital records.
  `,

  'i-130_beneficiary_name': `
    <strong>Beneficiary's Legal Name</strong>

    Enter your full legal name as it appears on your passport.
    - First Name: Your given name
    - Middle Name: Your middle name (if any)
    - Last Name/Family Name: Your surname
    - Name as it appears in passport: Use official spelling (may differ from how you write it casually)

    Important: The name must match your birth certificate or passport exactly.
  `,

  'i-130_date_of_birth': `
    <strong>Date of Birth</strong>

    Enter your date of birth in MM/DD/YYYY format.
    - Month: 01-12
    - Day: 01-31
    - Year: 1900-2024

    This must match your passport and birth certificate.
  `,

  // Common validation rules (no API needed)
  'i-485_income_threshold': `
    <strong>Federal Poverty Guideline Income Requirement</strong>

    Your household income must meet the federal poverty guideline based on:
    1. Household size (including you, sponsor, and dependents)
    2. Current year guideline (updated annually)

    As of 2024:
    - Household of 1: $15,060/year
    - Household of 2: $19,480/year
    - Household of 3: $24,860/year
    - Household of 4: $30,000/year
    - Household of 5: $35,520/year
    - Household of 6: $41,020/year

    Sponsor income + spouse income must exceed guideline.
  `,

  // Risk flags (template-based)
  'validation_no_employment_history': `
    <strong>⚠️ No Employment History Provided</strong>

    This form asks for employment history for the past 5 years.
    You indicated no employment history.

    Common reasons:
    - You are a student (may need to explain)
    - You are retired (provide retirement documentation)
    - You are a homemaker (provide family dependency proof)
    - You are unemployed (provide job search documentation)

    If you have no employment, you MUST include a written explanation.
  `,

  'validation_gaps_in_dates': `
    <strong>⚠️ Gaps in Employment/Residence History</strong>

    There are time periods where you did not provide information.

    This can trigger a Request for Evidence (RFE).

    For each gap, provide:
    - Written explanation of what you were doing
    - Supporting documentation (school enrollment, travel records, etc.)
  `,
};
```

---

## PART 4: MONITORING & ALERTING

### Real-Time Provider Health Dashboard

```typescript
// monitoring.ts

interface ProviderMetrics {
  provider: Provider;
  availability: number; // 0-100%
  averageLatency: number; // ms
  errorRate: number; // 0-100%
  cost24h: number;
  status: 'healthy' | 'degraded' | 'down';
}

class ProviderMonitor {
  private metrics: Map<Provider, ProviderMetrics> = new Map();

  // Check provider health every 60 seconds
  async startHealthCheck(): Promise<void> {
    setInterval(async () => {
      for (const provider of [
        Provider.CLAUDE,
        Provider.OPENAI,
        Provider.LLAMA2,
      ]) {
        const health = await this.checkProvider(provider);
        this.metrics.set(provider, health);

        // Alert if degraded
        if (health.status !== 'healthy') {
          await this.sendAlert(`Provider ${provider} is ${health.status}`, {
            ...health,
          });
        }

        // Log metrics to CloudWatch/Datadog
        logger.info('Provider health check', health);
      }
    }, 60 * 1000);
  }

  private async checkProvider(provider: Provider): Promise<ProviderMetrics> {
    const startTime = performance.now();
    let success = true;
    let error: Error | null = null;

    try {
      // Simple test request
      const response = await this.router.route({
        prompt: 'Test',
        type: 'explanation',
        fallbackOnError: false,
        timeout: 5000,
      });

      if (response.provider === Provider.FALLBACK) {
        success = false;
      }
    } catch (err) {
      success = false;
      error = err as Error;
    }

    const latency = performance.now() - startTime;

    // Calculate rolling metrics
    const previousMetrics = this.metrics.get(provider);
    const errorCount = success ? 0 : 1;
    const newErrorRate = previousMetrics
      ? (previousMetrics.errorRate * 0.9 + errorCount * 10) / 1.9
      : errorCount ? 100 : 0;

    const newAvailability = 100 - newErrorRate;

    let status: 'healthy' | 'degraded' | 'down';
    if (newAvailability >= 95) {
      status = 'healthy';
    } else if (newAvailability >= 50) {
      status = 'degraded';
    } else {
      status = 'down';
    }

    return {
      provider,
      availability: newAvailability,
      averageLatency: previousMetrics
        ? (previousMetrics.averageLatency + latency) / 2
        : latency,
      errorRate: newErrorRate,
      cost24h: this.costTracker.get(provider)?.totalCost || 0,
      status,
    };
  }

  async sendAlert(title: string, data: ProviderMetrics): Promise<void> {
    // Send to PagerDuty, Slack, etc.
    if (data.status === 'down') {
      // Critical alert
      await pagerduty.trigger({
        severity: 'critical',
        title,
        details: data,
      });
    } else if (data.status === 'degraded') {
      // Warning alert
      await slack.send({
        channel: '#alerts',
        text: `⚠️ ${title}`,
        blocks: [{ type: 'section', text: { type: 'mrkdwn', text: JSON.stringify(data) } }],
      });
    }
  }

  getMetrics(): ProviderMetrics[] {
    return Array.from(this.metrics.values());
  }
}
```

---

## PART 5: DEPLOYMENT CHECKLIST

### Pre-Production (Staging)

- [ ] Test Claude failover to OpenAI (stop Claude API, verify switch)
- [ ] Test OpenAI failover to Llama2 (stop OpenAI API, verify switch)
- [ ] Test all providers timeout (verify fallback response)
- [ ] Verify caching reduces API calls 40-60%
- [ ] Verify offline PWA works without internet
- [ ] Test cost tracking (verify cost per provider calculated correctly)
- [ ] Load test with 100 concurrent requests
- [ ] Verify monitoring alerts trigger correctly

### Production (Go-Live)

- [ ] Deploy provider abstraction layer (no user-facing changes)
- [ ] Enable Claude as primary provider (existing behavior)
- [ ] Monitor for 24 hours (verify no regressions)
- [ ] Gradually enable caching (start 10%, ramp to 100%)
- [ ] Enable Llama2 failover (non-critical first, then critical)
- [ ] Set up daily cost reports
- [ ] Schedule weekly health reviews

---

## PART 6: COST PROJECTION (12 Months)

| Scenario | Claude | OpenAI | Llama2 | Total Cost | Notes |
|----------|--------|--------|--------|------------|-------|
| **Month 1-3** (100 customers) | $200 | $0 | $0 | $200 | Primary only |
| **Month 4-6** (250 customers) | $1,200 | $100 | $0 | $1,300 | Start caching |
| **Month 7-9** (400 customers) | $1,800 | $500 | $200 | $2,500 | Llama2 active |
| **Month 10-12** (600 customers) | $2,400 | $1,200 | $600 | $4,200 | All providers active |
| **Year 1 Total** | **$5,600** | **$1,800** | **$800** | **$8,200** | |

**Assumptions:**
- 60% of queries hit cache (no API call)
- Average 50 API calls per customer per month
- Claude: $0.01 per query
- OpenAI: $0.015 per query
- Llama2: Free (self-hosted)

---

## CONCLUSION

This multi-provider architecture provides:
1. **Cost insurance:** Lock in Claude pricing, know OpenAI as backup
2. **Availability insurance:** Three independent providers + fallback
3. **Content policy insurance:** Open-source Llama2 not subject to provider restrictions
4. **Graceful degradation:** Product works in all scenarios, just with different quality

Implementation effort: 4-6 weeks of engineering
Ongoing cost: $1-2K/month for Llama2 infrastructure
Break-even: 3-4 months of avoided outage costs

