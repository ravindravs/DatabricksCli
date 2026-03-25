# Self-Hosted LLM Implementation Guide
## Technical Deep Dive for Immigration Form Tool
**Date:** March 25, 2026

---

## PART 1: QUICK START (LOCAL TESTING)

### 1.1 Option A: Ollama (Easiest)

**Installation:**
```bash
# macOS
brew install ollama

# Linux
curl https://ollama.ai/install.sh | sh

# Windows
# Download from https://ollama.ai/
```

**Run Llama 3.1 70B locally (requires 20GB+ VRAM):**
```bash
ollama pull llama2:70b
# or for smaller GPU:
ollama pull mistral:latest
```

**Test API:**
```bash
curl http://localhost:11434/api/generate \
  -d '{
    "model": "llama2:70b",
    "prompt": "Extract form fields from this I-130 form..."
  }'
```

**Cost:** Free (local GPU)
**Time to deployment:** 15 minutes

---

### 1.2 Option B: Run in Docker

**Docker Compose (vLLM backend):**
```yaml
version: "3.8"
services:
  llm:
    image: vllm/vllm-openai:latest
    ports:
      - "8000:8000"
    volumes:
      - ./models:/root/.cache/huggingface/hub
    environment:
      - VLLM_CPU_KVCACHE_SPACE=4
      - MAX_MODEL_LEN=2048
    command: >
      --model meta-llama/Llama-2-70b-hf
      --dtype float16
      --max-num-seqs 256
      --gpu-memory-utilization 0.9
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

**Deploy:**
```bash
docker-compose up -d
```

**Test (OpenAI-compatible API):**
```bash
curl http://localhost:8000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-2-70b-hf",
    "prompt": "Extract form fields...",
    "max_tokens": 512
  }'
```

---

## PART 2: PRODUCTION DEPLOYMENT

### 2.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                  Immigration Form Tool                   │
│                                                          │
│  ┌──────────────┐        ┌─────────────┐               │
│  │   Frontend   │───────►│  API Server │               │
│  │   (Browser)  │        │  (FastAPI)  │               │
│  └──────────────┘        └─────────────┘               │
│                                │                         │
│                          ┌─────▼─────┐                  │
│                          │   Router   │                  │
│                          └─────┬─────┘                  │
│                ┌─────────────────┴──────────────────┐   │
│                │                                    │   │
│         ┌──────▼──────┐               ┌─────────────▼──┐ │
│         │ Llama 70B   │               │  Claude API    │ │
│         │ (Self-Host) │               │  (Fallback)    │ │
│         └─────────────┘               └────────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Option 1: RunPod Deployment

**Provider:** RunPod (https://www.runpod.io/)

**Setup:**
1. Create account
2. Select A40 GPU pod ($0.60/hr)
3. Template: "vLLM OpenAI"
4. Volume: 50GB (for model + cache)
5. Expose port 8000

**Estimated Monthly Cost:**
- 330 hours × $0.60/hr = $198/month
- Volume storage: $5/month
- Total: ~$203/month

**Post-Deployment:**
```bash
# SSH into pod
ssh root@pod-id.io

# Install dependencies
pip install vllm transformers torch

# Download model (20GB)
git clone https://huggingface.co/meta-llama/Llama-2-70b-hf
# (requires HuggingFace login)

# Start vLLM server
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Llama-2-70b-hf \
  --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.95 \
  --dtype float16 \
  --port 8000

# Expose public URL
# RunPod provides HTTP endpoint automatically
```

**Test Endpoint:**
```bash
curl https://pod-[id].runpod.io/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-2-70b-hf",
    "prompt": "Validate SSN format: 123-45-6789",
    "max_tokens": 100,
    "temperature": 0.1
  }'
```

---

### 2.3 Option 2: AWS EC2 Deployment

**Instance Specification:**

```hcl
# Terraform configuration
resource "aws_instance" "llm_server" {
  ami           = "ami-0c55b159cbfafe1f0"  # Deep Learning AMI
  instance_type = "g4dn.2xlarge"           # 1x T4 GPU, $0.752/hr

  key_name = "your-keypair"

  tags = {
    Name = "immigration-form-llm"
  }
}
```

**Manual Setup:**
1. Launch EC2 instance (g4dn.2xlarge or g4dn.12xlarge)
2. Choose Deep Learning AMI (Ubuntu 22.04)
3. Configure security group (allow port 8000)
4. SSH in and install:

```bash
# Install NVIDIA CUDA drivers (pre-installed on DL AMI)
nvidia-smi

# Install vLLM
pip install vllm transformers torch

# Download model
huggingface-cli login
git clone https://huggingface.co/meta-llama/Llama-2-70b-hf

# Start server
python -m vllm.entrypoints.openai.api_server \
  --model ./Llama-2-70b-hf \
  --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.9
```

**Cost (g4dn.2xlarge, on-demand):**
- Instance: $0.752/hour = $549/month
- EBS storage (100GB): $10/month
- Data transfer: ~$5/month
- **Total: ~$564/month** (similar to RunPod but less management)

**Cost (g4dn.2xlarge, reserved 1-year):**
- Instance: $0.375/hour = $273/month
- EBS: $10/month
- **Total: ~$283/month** (30% cheaper)

---

### 2.4 Option 3: Databricks MLflow (Managed)

**Platform:** Databricks (https://www.databricks.com/)

**Setup:**
1. Create Databricks workspace
2. Create compute cluster (4-GPU, auto-scaling)
3. Deploy model via MLflow

```python
import mlflow.pyfunc
from transformers import AutoTokenizer, AutoModelForCausalLM

# Register model in MLflow
model_info = mlflow.transformers.log_model(
    transformers_model={
        "model": "meta-llama/Llama-2-70b-hf",
        "tokenizer": "meta-llama/Llama-2-70b-hf",
    },
    artifact_path="llama70b",
    input_example=["Extract form fields..."],
    signature=mlflow.models.infer_signature(
        model_input=["test"],
        model_output={"extracted_fields": {}}
    )
)

# Deploy for serving
mlflow.deployments.deploy_model(
    model_uri=f"runs:/{model_info.run_id}/llama70b",
    config={"served_entities": [{"name": "llama-form-extractor"}]}
)
```

**Endpoint:**
```bash
# MLflow auto-generates endpoint URL
curl https://databricks-workspace.cloud.databricks.com/serving/llama-form-extractor \
  -H "Authorization: Bearer $DATABRICKS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"dataframe_split": {"columns": ["text"], "data": [["Extract fields..."]]}}'
```

**Cost:** $400-500/month (managed service, auto-scaling)

---

## PART 3: STRUCTURED OUTPUT (JSON EXTRACTION)

### 3.1 Using Ollama Structured Output

**Ollama now supports constrained JSON output:**

```bash
# Define JSON schema
SCHEMA='{
  "type": "object",
  "properties": {
    "ssn": {"type": "string"},
    "passport": {"type": "string"},
    "visa_status": {"type": "string"},
    "inconsistencies": {
      "type": "array",
      "items": {"type": "string"}
    }
  }
}'

# Call with schema constraint
curl http://localhost:11434/api/generate \
  -d '{
    "model": "llama2:70b",
    "prompt": "Extract immigration form fields: {{form_text}}",
    "format": "json",
    "schema": '"$SCHEMA"',
    "stream": false
  }'
```

**Response Example:**
```json
{
  "ssn": "123-45-6789",
  "passport": "N12345678",
  "visa_status": "F-1",
  "inconsistencies": ["Employment dates overlap", "Missing visa expiration date"]
}
```

### 3.2 Using vLLM with Guided Generation

```python
from vllm import LLM, SamplingParams
import json

llm = LLM(
    model="meta-llama/Llama-2-70b-hf",
    tensor_parallel_size=1,
    dtype="float16"
)

# Define output schema
output_schema = {
    "type": "object",
    "properties": {
        "form_type": {"type": "string", "enum": ["I-130", "I-485", "I-539"]},
        "is_complete": {"type": "boolean"},
        "extracted_fields": {"type": "object"},
        "errors": {"type": "array", "items": {"type": "string"}}
    },
    "required": ["form_type", "is_complete"]
}

# Sampling parameters
sampling_params = SamplingParams(
    temperature=0,  # Deterministic (required for JSON)
    top_p=1.0,
    max_tokens=512
)

# Generate with schema
prompts = [
    f"Extract fields from immigration form and return JSON:\n{form_text}"
    for form_text in forms
]

outputs = llm.generate(
    prompts,
    sampling_params,
    use_tqdm=True
)

# Parse results
results = [json.loads(output.outputs[0].text) for output in outputs]
```

### 3.3 Fallback for Structured Output

**Some models struggle with perfect JSON. Handle gracefully:**

```python
import json
import re

def extract_json_fallback(text, max_retries=3):
    """Extract JSON from model output with fallback parsing."""

    # Attempt 1: Direct JSON parsing
    try:
        return json.loads(text)
    except json.JSONDecodeError:
        pass

    # Attempt 2: Extract JSON from markdown code blocks
    match = re.search(r'```(?:json)?\n(.*?)\n```', text, re.DOTALL)
    if match:
        try:
            return json.loads(match.group(1))
        except json.JSONDecodeError:
            pass

    # Attempt 3: Retry with instruction to output pure JSON
    if max_retries > 0:
        retry_prompt = f"""Output ONLY valid JSON (no markdown, no explanation):
        {text[:500]}..."""
        # Retry LLM call with stricter prompt
        return extract_json_fallback(retry_prompt, max_retries - 1)

    # Fallback: Return structured error
    return {
        "error": "Failed to extract JSON",
        "raw_output": text[:500]
    }

# Usage
result = extract_json_fallback(llm_output)
```

---

## PART 4: INTEGRATION WITH EXISTING BACKEND

### 4.1 FastAPI Wrapper

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import httpx
import json
from typing import Optional

app = FastAPI()

# Configuration
LLM_ENDPOINT = "http://localhost:8000"  # or RunPod URL
LLM_MODEL = "meta-llama/Llama-2-70b-hf"

class FormRequest(BaseModel):
    form_text: str
    form_type: str
    user_id: str

class FormResponse(BaseModel):
    extracted_fields: dict
    inconsistencies: list
    confidence: float
    processing_time_ms: int

@app.post("/api/extract-form", response_model=FormResponse)
async def extract_form(request: FormRequest):
    """Extract structured data from immigration form."""

    prompt = f"""Extract immigration form fields and identify inconsistencies.

Form Type: {request.form_type}
Form Text:
{request.form_text}

Return valid JSON with:
{{"
  "extracted_fields": {{ /* All form fields */ }},
  "inconsistencies": [ /* List of errors */ ],
  "missing_fields": [ /* List of required fields not present */ ]
}}"""

    try:
        # Call self-hosted LLM
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{LLM_ENDPOINT}/v1/completions",
                json={
                    "model": LLM_MODEL,
                    "prompt": prompt,
                    "max_tokens": 1024,
                    "temperature": 0,  # Deterministic
                    "top_p": 1.0
                },
                timeout=30.0
            )

        # Parse response
        completion = response.json()
        raw_output = completion["choices"][0]["text"]

        # Extract JSON
        extracted = extract_json_fallback(raw_output)

        return FormResponse(
            extracted_fields=extracted.get("extracted_fields", {}),
            inconsistencies=extracted.get("inconsistencies", []),
            confidence=0.85,  # Placeholder
            processing_time_ms=completion.get("usage", {}).get("prompt_tokens", 0)
        )

    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health_check():
    """Check LLM endpoint availability."""
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{LLM_ENDPOINT}/health",
                timeout=5.0
            )
        return {"status": "healthy"}
    except:
        return {"status": "unhealthy"}
```

### 4.2 Router for Hybrid (Self+API)

```python
from enum import Enum

class RoutingStrategy(Enum):
    SELF_HOSTED = "self"
    CLAUDE_API = "claude"
    AUTO_ROUTE = "auto"

async def extract_form_hybrid(
    request: FormRequest,
    strategy: RoutingStrategy = RoutingStrategy.AUTO_ROUTE
):
    """Route to self-hosted or API based on strategy."""

    if strategy == RoutingStrategy.SELF_HOSTED:
        return await extract_form_self_hosted(request)

    elif strategy == RoutingStrategy.CLAUDE_API:
        return await extract_form_claude(request)

    elif strategy == RoutingStrategy.AUTO_ROUTE:
        # Try self-hosted first
        result = await extract_form_self_hosted(request)

        # If quality is low, use Claude as fallback
        if result.confidence < 0.80:
            logging.warning(f"Low confidence ({result.confidence}) for form {request.form_type}, using Claude")
            result = await extract_form_claude(request)

        return result

async def extract_form_self_hosted(request):
    """Use self-hosted Llama 70B."""
    # ... implementation from 4.1
    pass

async def extract_form_claude(request):
    """Fallback to Claude API with redaction."""
    # ... existing redaction proxy logic
    pass
```

---

## PART 5: MONITORING & OBSERVABILITY

### 5.1 Prometheus Metrics

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Metrics
llm_requests = Counter(
    'llm_requests_total',
    'Total LLM requests',
    ['model', 'status', 'form_type']
)

llm_latency = Histogram(
    'llm_request_duration_seconds',
    'LLM request latency',
    ['model'],
    buckets=(0.1, 0.5, 1.0, 5.0, 10.0)
)

llm_gpu_memory = Gauge(
    'llm_gpu_memory_bytes',
    'GPU memory usage'
)

llm_accuracy = Gauge(
    'llm_accuracy_score',
    'Form extraction accuracy (user-reviewed)',
    ['form_type']
)

# Track in FastAPI
@app.post("/api/extract-form")
async def extract_form(request: FormRequest):
    start_time = time.time()

    try:
        result = await extract_form_impl(request)
        llm_requests.labels(
            model="llama70b",
            status="success",
            form_type=request.form_type
        ).inc()
        return result

    except Exception as e:
        llm_requests.labels(
            model="llama70b",
            status="error",
            form_type=request.form_type
        ).inc()
        raise

    finally:
        latency = time.time() - start_time
        llm_latency.labels(model="llama70b").observe(latency)
```

### 5.2 Logging Configuration

```python
import logging
from pythonjsonlogger import jsonlogger

# JSON logging (structured)
logHandler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter()
logHandler.setFormatter(formatter)
logger = logging.getLogger()
logger.addHandler(logHandler)
logger.setLevel(logging.INFO)

# Log extraction request (redact PII)
def log_extraction(request, result):
    logger.info(
        "Form extraction complete",
        extra={
            "form_type": request.form_type,
            "user_id_hash": hash(request.user_id),  # Hash, don't log raw
            "extracted_fields_count": len(result.extracted_fields),
            "inconsistencies": result.inconsistencies,
            "processing_time_ms": result.processing_time_ms,
            "model": "llama70b",
            "confidence": result.confidence
        }
    )
```

---

## PART 6: FINE-TUNING (OPTIONAL)

### 6.1 Prepare Training Data

```python
# Format: JSONL (one JSON object per line)
# Example training data
{
  "instruction": "Extract immigration form fields and identify issues",
  "input": "I-130 Form:\nSponsoring Relative: John Smith\nRelationship: Spouse\nSSN: 123-45-6789...",
  "output": "{\"extracted_fields\": {\"form_type\": \"I-130\", \"sponsor_name\": \"John Smith\"...}, \"inconsistencies\": []}"
}
```

**Collect 500-1,000 examples:**
1. Run current model on actual forms
2. Have human verify/correct outputs
3. Save as JSONL training data

### 6.2 Fine-Tune with LoRA

**Using QLoRA (memory-efficient):**

```bash
# Install LLaMA-Factory
pip install llamafactory

# Prepare config
cat > lora_config.json << EOF
{
  "output_dir": "./output/llama70b_finetuned",
  "overwrite_output_dir": true,
  "per_device_train_batch_size": 4,
  "gradient_accumulation_steps": 4,
  "learning_rate": 5e-4,
  "num_train_epochs": 3,
  "save_strategy": "epoch",
  "logging_steps": 10,
  "bf16": true,
  "lora_r": 64,
  "lora_alpha": 16,
  "lora_dropout": 0.05,
  "target_modules": ["q_proj", "v_proj"],
  "modules_to_save": ["lm_head"]
}
EOF

# Run training
llamafactory train lora_config.json
```

**Cost:** $5-10K (4-week cloud GPU rental) or free (local if you have H100)

---

## PART 7: TROUBLESHOOTING

### 7.1 Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| JSON extraction fails (30% of requests) | Model struggles with instruction | Add retry with stricter prompt |
| Out of memory (OOM) | Model too large for GPU | Use 4-bit quantization (reduces by 75%) |
| Slow inference (>10s per request) | GPU underutilized | Batch requests or use multi-GPU |
| Hallucination (SSN invented) | Temperature too high | Set temperature=0 for deterministic output |
| Model drift (accuracy decreases) | New form types | Fine-tune monthly on new examples |

### 7.2 Optimization Checklist

```
[ ] Use 4-bit quantization (reduce memory 75%)
[ ] Set temperature=0 for JSON extraction
[ ] Use vLLM (faster than llama.cpp)
[ ] Batch requests (5-10 at a time)
[ ] Monitor GPU memory (target: 80% utilization)
[ ] Cache model weights on disk (avoid re-downloading)
[ ] Use TLS for API calls (security)
[ ] Rate limit (100 req/min per user)
[ ] Add request timeout (30 seconds max)
[ ] Implement circuit breaker (fallback if model errors)
```

---

## PART 8: DEPLOYMENT CHECKLIST

### Pre-Production

- [ ] Test model on 100+ sample forms
- [ ] Achieve >80% accuracy vs GPT-4o baseline
- [ ] Load test (simulate 100 concurrent requests)
- [ ] Monitor latency (target: <5 seconds)
- [ ] Set up logging & monitoring
- [ ] Document API endpoints
- [ ] Create runbooks (troubleshooting guides)
- [ ] Backup model weights
- [ ] Test failover (self-hosted → Claude)
- [ ] Review security (TLS, API keys, auth)

### Production Launch

- [ ] Deploy to RunPod/AWS/Databricks
- [ ] Set up alerting (uptime monitoring)
- [ ] Route 10% traffic initially (canary)
- [ ] Monitor for 1 week
- [ ] Gradually increase traffic (10% → 50% → 100%)
- [ ] Have human review tier for edge cases
- [ ] Measure cost per form vs baseline

### Post-Launch (Ongoing)

- [ ] Monthly accuracy reviews
- [ ] Quarterly fine-tuning on new examples
- [ ] Monitor for model drift
- [ ] Track customer feedback
- [ ] Benchmark against latest models
- [ ] Plan for model upgrades (Llama 4, etc.)

---

## COST SUMMARY TABLE

| Deployment | Setup Time | Monthly Cost | Notes |
|------------|-----------|--------------|-------|
| **Local (Ollama)** | 15 min | $0 | Dev only |
| **RunPod T4** | 1 hour | $50 | Batch processing |
| **RunPod A40** | 1 hour | $200 | Recommended |
| **AWS EC2 g4dn** | 2 hours | $280-550 | More control |
| **Databricks** | 2 hours | $400-500 | Fully managed |
| **Groq API** | 10 min | $15/month (at your volume) | No infrastructure |

---

**Document Version:** 1.0
**Last Updated:** March 25, 2026
