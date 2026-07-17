# ⭐ **Model Observability — AWS SA Loop Interview‑Ready Version**

## **1. What Model Observability Means in GenAI Systems**
> “For GenAI workloads, observability is not just about uptime — it’s about understanding latency, cost, safety, and quality across every model invocation.”

Amazon Bedrock integrates with existing AWS observability tools, primarily:

- **CloudWatch** → metrics, dashboards, alarms  
- **CloudTrail** → API audit logs  
- **Model Invocation Logging** → structured logs of prompts, responses, tokens, and guardrail results  
- **X-Ray** → tracing for application-level latency (Lambda, API Gateway, etc.)

This gives you full visibility into how models behave in production.

---

## **2. Observability for bedrock‑runtime (Invoke / Converse APIs)**

### **A. Model Invocation Logging**
You can deliver invocation logs to **S3** or **CloudWatch Logs**.

**Setup steps:**
1. **Choose a destination**  
   - S3 bucket (with `s3:PutObject` permission for `bedrock.amazonaws.com`)  
   - CloudWatch Log Group (via an IAM role Bedrock can assume)

2. **Create IAM role for CloudWatch Logs**  
   - Trust policy: `bedrock.amazonaws.com`  
   - Permissions: `logs:CreateLogStream`, `logs:PutLogEvents`

3. **Enable Model Invocation Logging**  
   - Select destination  
   - Attach IAM role  
   - Choose modalities to log: text, image, embedding, video

**What the logs contain:**
- Input prompt (redacted if configured)  
- Model output  
- Input/output token counts  
- Latency  
- Guardrail evaluation results  
- Model ARN  
- Request ID  

These logs are essential for debugging, cost analysis, and safety monitoring.

---

### **B. CloudWatch Metrics (runtime)**

Key metrics include:

- **Invocations**  
- **InvocationLatency**  
- **InputTokenCount**  
- **OutputTokenCount**  
- **CacheReadInputTokens / CacheWriteInputTokens**  
- **OTPS (Output Tokens Per Second)** → critical for performance & cost

> “OTPS is one of the most important GenAI metrics — slow token generation increases latency and cost.”

---

### **C. CloudTrail (runtime)**  
CloudTrail logs all Bedrock API calls, including:

- `InvokeModel`  
- `InvokeModelWithResponseStream`  
- `Converse`  
- `ConverseStream`  
- `ListAsyncInvokes`

This is essential for auditability and debugging production issues.

---

## **3. Observability for bedrock‑mantle (ChatCompletions / Responses / Messages)**

Mantle provides **richer observability** because it supports agentic workflows and OpenAI/Anthropic‑compatible APIs.

### **A. CloudWatch Metrics (mantle)**

Mantle publishes metrics at **four granularity levels**:

1. **Account level**  
2. **Project level**  
3. **Model level**  
4. **Project + Model level**

### **Metric categories**

#### **1. Inference Metrics**
- `Inferences`  
- `InferenceClientErrors`  
- `InferenceServerErrors`  
- `InferenceThrottles`

#### **2. Token Metrics**
- `TotalInputTokens`  
- `TotalOutputTokens`  
- `InputTokens`  
- `OutputTokens`  
- `TokenBillingUnits` (mantle-specific)

#### **3. Dimensions**
- `Project`  
- `Model`

Mantle metrics help you understand cost, throughput, and error patterns at a granular level.

---

## **4. Why Observability Matters in GenAI Architectures**

### **A. Latency Observability**
- First-token latency  
- End-to-end inference latency  
- OTPS  
- Streaming vs non-streaming performance

### **B. Cost Observability**
- Token usage  
- Cache hits/misses  
- Billing units (mantle)  
- High-volume workload optimization

### **C. Safety Observability**
- Guardrail triggers  
- Blocked inputs/outputs  
- Sensitive information detection  
- Topic filter violations

### **D. Quality Observability**
- Hallucination detection  
- Context grounding failures  
- RAG retrieval relevance  
- Citation precision (mantle + RAG)

---

## **5. Putting It All Together — How an SA Explains It**

> “I use CloudWatch for metrics, CloudTrail for API auditing, and Model Invocation Logging for deep visibility into prompts, responses, tokens, and guardrail outcomes. For mantle workloads, I also monitor project‑level and model‑level metrics to understand cost and throughput. This gives me a complete picture of latency, cost, safety, and quality — the four pillars of GenAI observability.”

---
