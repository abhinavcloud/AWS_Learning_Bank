# AWS Bedrock & Generative AI — Principal Solutions Architect Master Guide  
**Purpose:** Deep, interview‑ready reference for AWS SA loop on GenAI with Bedrock. Use this to prepare architecture answers, whiteboard designs, and scenario responses.  
**Audience:** AWS SA candidate (Abhinav) preparing for an AWS loop focused on GenAI / Bedrock.

---

## Contents
1. **Executive summary**  
2. **Bedrock fundamentals: models, APIs, endpoints**  
3. **Model selection framework (decision process + examples)**  
4. **Guardrails: design, policies, and operationalization**  
5. **Model evaluation: LLM, RAG, and LLM‑as‑a‑judge**  
6. **Model observability: metrics, logs, alerts, and debugging**  
7. **RAG architecture deep dive and anti‑hallucination patterns**  
8. **Agentic workflows, tool use, and mantle specifics**  
9. **Security, IAM, governance, and compliance**  
10. **Scaling, cost optimization, and Provisioned Throughput**  
11. **Production patterns, diagrams, and example architectures**  
12. **Interview‑ready phrasing, common pitfalls, and bar‑raiser signals**  
13. **Two‑page cheat sheet**

---

## 1. Executive summary
**Core thesis:** Build GenAI systems by *eliminating* unsuitable models first, enforce safety with *guardrails*, ground outputs with *RAG*, and validate with *evaluation* and *observability* on customer data. For production, prioritize predictable latency, cost control, and auditable safety controls.

**Four pillars of GenAI production readiness:**
- **Latency & performance** — predictable response times (first‑token, OTPS).  
- **Cost & efficiency** — token budgeting, caching, model sizing.  
- **Safety & compliance** — guardrails, PII detection, audit trails.  
- **Quality & trust** — evaluation, RAG grounding, human/LLM review.

---

## 2. Bedrock fundamentals: models, APIs, endpoints

### Models and modalities
- **Text**: small/fast (Nova Micro, Nova Lite, Claude Haiku) vs frontier/reasoning (Claude Opus, Sonnet, GPT‑5.5).  
- **Vision**: models that accept images/PDFs (Claude Opus/Haiku, Nova Pro, Pixtral, Gemma).  
- **Image generation**: Nova Canvas, Stable Image family.  
- **Video**: Nova Reel (async), TwelveLabs Pegasus (video understanding).  
- **Speech**: Nova Sonic (bidirectional streaming).  
- **Embeddings**: Titan Text Embeddings V2, Cohere Embed v4, Nova multimodal embeddings.

### API families (what to use when)
- **InvokeModel / InvokeModelWithResponseStream / AsyncInvoke** — universal, sync/stream/async inference.  
- **Converse / ConverseStream** — model‑agnostic multi‑turn chat interface. Good for vendor neutrality.  
- **ChatCompletions / Responses** — OpenAI‑compatible; Responses supports stateful agentic tool use. Use for minimal migration effort.  
- **Messages** — Anthropic‑compatible (Claude) interface.

### Endpoints
- **bedrock‑runtime** — broad support; safe default.  
- **bedrock‑mantle** — newer, OpenAI/Anthropic SDK compatibility, agentic workflows; not all models available.

---

## 3. Model selection framework (decision process + examples)

**Five elimination questions (in order):**
1. **Task type** — reasoning vs fluency vs extraction vs code.  
2. **Modality** — text, vision, audio, video, embeddings.  
3. **Latency / throughput / cost** — real‑time vs batch; QPS and PT needs.  
4. **Architecture constraints** — existing SDKs, vendor neutrality, region/residency.  
5. **Customization** — fine‑tuning vs RAG vs embeddings.

**Practical mapping (examples):**
- **High‑volume support chatbot**: Nova Micro / Nova Lite / Claude Haiku (low latency, low cost).  
- **Complex agent with tools**: Claude Opus / Sonnet / GPT‑5.5 (reasoning, tool orchestration).  
- **Document analysis with images**: Claude Opus / Nova Pro / Pixtral Large.  
- **Semantic search / RAG**: Titan or Nova embeddings + a strong generator (separate choices).  
- **Image generation**: Nova Canvas or Stable Image family.

**Interview tip:** Always say you will *benchmark on the customer’s dataset* before finalizing.

---

## 4. Guardrails: design, policies, and operationalization

### What Guardrails provide
- **Content filters**: hate, profanity, sexual, violence, insult, misconduct, prompt attack.  
- **Topic filters**: block categories (e.g., medical, legal, financial advice) and return a standard fallback.  
- **Sensitive information filters**: PII detection (names, SSN, phone, email, addresses, race).  
- **Word filters**: deny or redact specific words/phrases.  
- **Context grounding**: detect unsupported claims; prevent hallucinations by blocking or flagging.  
- **Automated reasoning checks**: validate outputs against logical rules.

### Safeguard tiers
- **Classic** — baseline languages (English, French, Spanish).  
- **Standard** — broader language and code support, improved performance.

### IAM and invocation model
- **ApplyGuardrail** must be allowed on the guardrail resource.  
- **InvokeModel** must be allowed on the model resource.  
- Example policy fragments:
```json
{
  "Action": ["bedrock:InvokeModel","bedrock:InvokeModelWithResponseStream"],
  "Resource": ["arn:aws:bedrock:region::foundation-model/*"]
}
```
and
```json
{
  "Action": ["bedrock:ApplyGuardrail"],
  "Resource": ["arn:aws:bedrock:region:acct:guardrail/guardrail-id"]
}
```

### Operational patterns
- **Input filtering**: block or sanitize before model sees prompt.  
- **Output filtering**: post‑generation check and redact/replace.  
- **Logging**: guardrail triggers must be logged to Model Invocation Logging for audit and metrics.  
- **Integration with RAG**: apply guardrails to retrieved context + generated answer to ensure safety.

**Interview phrasing:** “Guardrails filter input and output, but they do not remove or alter internal reasoning traces.”

---

## 5. Model evaluation: LLM, RAG, and LLM‑as‑a‑judge

### Why evaluate
- Align model behavior to customer data and use case.  
- Make tradeoffs between **quality**, **latency**, and **cost**.  
- Monitor bias, safety, and trust.

### Evaluation methods
- **Programmatic**: BERTScore, BLEU, F1, classification metrics, robustness tests, toxicity checks.  
- **Human**: Likert scales, binary choices, ordinal ranking for style, tone, brand voice.  
- **LLM‑as‑a‑Judge**: use a trusted LLM to score correctness, completeness, helpfulness, coherence; correlate with human judgments.

### RAG evaluation specifics
- **Retrieval metrics**: context coverage, context relevance, retrieval latency.  
- **Retrieve+Generate metrics**: correctness, completeness, citation precision/coverage, faithfulness, logical coherence.  
- **Responsible AI metrics**: harmfulness, stereotyping, refusal behavior.

### Evaluation workflow (Bedrock)
1. Select knowledge base (S3, vector store).  
2. Choose retrieval‑only or retrieval+generation evaluation.  
3. Define system metrics and custom metrics (system prompts).  
4. Provide S3 prompt source and evaluation output destination.  
5. Provide IAM role for evaluation job.  
6. Run asynchronous evaluation job; results land in S3.

**Interview tip:** Emphasize *asynchronous evaluation jobs* and *benchmarking on customer data*.

---

## 6. Model observability: metrics, logs, alerts, and debugging

### Observability pillars for GenAI
- **Latency observability**: first‑token latency, time‑to‑last‑token, OTPS (Output Tokens Per Second).  
- **Cost observability**: input/output token counts, token billing units, cache hits/misses.  
- **Safety observability**: guardrail triggers, blocked inputs/outputs, PII detection events.  
- **Quality observability**: hallucination detection, grounding failures, RAG retrieval relevance.

### Tools & artifacts
- **CloudWatch**: metrics, dashboards, alarms (runtime and mantle metrics).  
- **CloudTrail**: API audit logs (InvokeModel, Converse, ChatCompletions, Responses, ApplyGuardrail, CreateGuardrail, evaluation APIs).  
- **Model Invocation Logging**: structured logs to S3 or CloudWatch Logs (prompts, responses, tokens, guardrail results, request IDs).  
- **X‑Ray**: application tracing (API Gateway, Lambda, step functions).

### Key runtime metrics
- **Invocations**  
- **InvocationLatency** (end‑to‑end)  
- **InputTokenCount / OutputTokenCount**  
- **CacheReadInputTokens / CacheWriteInputTokens**  
- **OTPS** — monitor token generation rate; slow OTPS → higher latency and cost.

### Mantle metrics (richer)
- **Inferences**, **InferenceClientErrors**, **InferenceServerErrors**, **InferenceThrottles**  
- **TotalInputTokens**, **TotalOutputTokens**, **TokenBillingUnits**  
- Dimensions: **Project**, **Model**; granularity: account, project, model, project+model.

### Logging & redaction
- Invocation logs include prompts and outputs — **redact or mask PII** before storing if required by policy.  
- Guardrail results must be logged for safety telemetry.  
- For agentic workflows, log tool calls, function inputs/outputs, and step traces.

### Alerting & SLOs
- Set alarms on **InvocationLatency**, **OTPS anomalies**, **Guardrail trigger spikes**, **Inference errors**, **PT throttles**.  
- Define SLOs for first‑token latency and tail latency (p95/p99).

### Debugging workflow
1. Reproduce with invocation logs (request ID).  
2. Inspect token counts and OTPS.  
3. Check cache hit/miss and retrieval logs (for RAG).  
4. Check guardrail triggers and safety logs.  
5. If agentic, inspect tool traces.

---

## 7. RAG architecture deep dive and anti‑hallucination patterns

### RAG components
- **Document ingestion**: OCR, normalization, chunking (semantic vs fixed).  
- **Embeddings**: choose embedding model for semantic fidelity (Titan, Cohere, Nova).  
- **Vector store**: Faiss, OpenSearch, Pinecone, AWS OpenSearch, or managed vector DB.  
- **Retriever**: hybrid search (sparse + dense), reranker.  
- **Generator**: foundation model that consumes retrieved context.  
- **Guardrails**: applied to retrieved context and generated output.  
- **Evaluation**: retrieval‑only and retrieval+generation jobs.

### Chunking & context engineering
- **Semantic chunking** (sentence/paragraph-level) often outperforms fixed-size chunking for relevance.  
- Keep chunks small enough to fit context window but large enough to preserve meaning.  
- Store chunk metadata (source doc, offset, citation info).

### Retrieval strategies
- **Hybrid search**: BM25 + dense embeddings.  
- **Reranking**: use a smaller model to rerank top K candidates.  
- **Context selection**: choose top N chunks that maximize coverage and relevance.

### Anti‑hallucination techniques
- **Context grounding**: require generator to cite sources; use guardrails to block unsupported claims.  
- **Citation prompting**: instruct model to include citations and refuse when unsupported.  
- **Answer refusal**: design model/system to refuse when confidence is low.  
- **Post‑generation verification**: use LLM‑as‑a‑judge to check factuality against retrieved context.  
- **Retrieval evaluation**: measure context coverage and relevance to debug hallucinations.

---

## 8. Agentic workflows, tool use, and mantle specifics

### Agentic patterns
- **Tool orchestration**: model calls external tools (search, DB queries, calculators).  
- **Function calling**: Responses API supports structured function calls and tool invocation.  
- **Step tracing**: log each step for observability and audit.

### Mantle specifics
- **Responses** supports stateful agentic workflows and function/tool calls.  
- Mantle provides richer metrics (token billing units) and traces for tool use.  
- Use mantle when migrating OpenAI SDK apps or when agentic orchestration is required.

### Safety & observability for agents
- Log tool inputs/outputs and enforce guardrails on tool outputs.  
- Monitor tool error rates and latencies.  
- Ensure idempotency and retry semantics for tool calls.

---

## 9. Security, IAM, governance, and compliance

### IAM best practices
- **Least privilege**: separate roles for model invocation vs guardrail management vs evaluation jobs.  
- **Resource scoping**: apply `bedrock:ApplyGuardrail` to guardrail ARNs and `bedrock:InvokeModel` to model ARNs.  
- **Audit**: CloudTrail for all Bedrock API calls.

### Network & data controls
- **VPC endpoints / PrivateLink** for private connectivity.  
- **Encryption**: TLS in transit; SSE for S3 logs and artifacts.  
- **Data residency**: choose region where model and data residency requirements are met.

### Data usage & privacy
- Confirm provider policies: many managed models do not use customer data for training by default; verify with AWS documentation and contracts.  
- **PII handling**: redact or avoid logging sensitive fields; use guardrails to detect PII.

### Governance
- **Model catalog governance**: approved models list, allowed endpoints, and guardrail templates.  
- **Change control**: require evaluation jobs and safety review before deploying new models or guardrail changes.  
- **Incident response**: playbooks for safety incidents (e.g., guardrail bypass, data leak).

---

## 10. Scaling, cost optimization, and Provisioned Throughput

### Cost levers
- **Model choice**: smaller models for classification/extraction; frontier models only when needed.  
- **Token budgeting**: limit prompt and response lengths; use compression.  
- **Caching**: cache common responses or embeddings.  
- **Async vs sync**: use async for batch jobs to reduce cost.  
- **Embeddings vs fine‑tuning**: RAG often cheaper than fine‑tuning for domain knowledge.

### Provisioned Throughput (PT)
- Use PT for predictable high QPS and low tail latency.  
- Monitor PT metrics: utilization, consumed, throttles.  
- PT availability varies by model — check model support.

### Autoscaling & capacity planning
- Combine PT for baseline capacity and on‑demand for spikes.  
- Use load testing with representative prompts and token distributions.

---

## 11. Production patterns, diagrams, and example architectures

### Example 1 — High‑volume support chatbot (low latency)
```
User → API Gateway → Lambda (stateless) → Bedrock Invoke (Nova Micro) → Guardrails → Response
```
- **Key points:** PT for baseline, token budgeting, caching, CloudWatch alarms on latency and OTPS.

### Example 2 — RAG for legal documents
```
User → App → Retriever (vector DB) → Top K chunks → Bedrock FM (Claude Opus) → Guardrails → Response + Citations
```
- **Key points:** semantic chunking, reranker, citation prompting, evaluation jobs on legal dataset.

### Example 3 — Agentic workflow (mantle)
```
User → bedrock-mantle Responses → Agent → Tool calls (DB, search, calculator) → Agent → Guardrails → Response
```
- **Key points:** tool traces, function call logging, guardrails on tool outputs.

---

## 12. Architecture diagrams (ASCII)

### RAG + Guardrails
```
[User] → [API GW] → [App] → [Retriever] → [Vector DB]
                                 ↓
                              [Top K]
                                 ↓
                            [Bedrock FM]
                                 ↓
                           [Guardrails]
                                 ↓
                              [Response]
```

### Mantle agentic flow
```
[User] → [bedrock-mantle Responses]
           ↓
         [Agent]
      ↙    ↓     ↘
[Tool A] [Tool B] [Tool C]
   ↓       ↓        ↓
[Results aggregated] → [Guardrails] → [Response]
```

---

## 13. Interview‑ready phrasing and example answers

- **Model selection:** “I eliminate models by task, modality, latency/cost, architecture constraints, and customization needs, then benchmark on customer data.”  
- **Guardrails:** “Guardrails enforce safety at input and output; they log triggers for audit and do not alter internal reasoning traces.”  
- **Observability:** “OTPS and first‑token latency are critical; I instrument token counts, guardrail triggers, and retrieval relevance.”  
- **RAG:** “Isolate retrieval vs generation: evaluate retrieval-only first to debug chunking and embeddings.”  
- **Agentic:** “Use mantle Responses for function calls and tool orchestration; log each tool call for observability and safety.”

---

## 14. Common pitfalls candidates make (and how to avoid them)
- **Pitfall:** Naming a model first. **Fix:** Use the elimination framework.  
- **Pitfall:** Ignoring token costs and OTPS. **Fix:** Discuss token budgeting and OTPS monitoring.  
- **Pitfall:** Overlooking guardrail IAM and logging. **Fix:** Mention `ApplyGuardrail` and invocation logging.  
- **Pitfall:** Treating RAG as a single black box. **Fix:** Explain retrieval vs generation separation and evaluation.  
- **Pitfall:** Not discussing production SLOs and PT. **Fix:** Bring up PT, p95/p99 latency, and capacity planning.

---

## 15. What bar‑raisers look for
- **Decision process** over memorized facts.  
- **Architectural tradeoffs** (latency vs cost vs quality).  
- **Safety & governance** awareness.  
- **Operational readiness** (observability, SLOs, incident response).  
- **Customer obsession** — benchmark on customer data, not public leaderboards.

---

## 16. Hands‑on architecture examples (concise)

### Customer support bot (10k conv/day)
- **Model:** Nova Micro / Claude Haiku  
- **PT:** Provisioned Throughput for baseline QPS  
- **Observability:** CloudWatch dashboards for InvocationLatency, OTPS, token counts; CloudTrail for audit; Invocation logs to S3.  
- **Guardrails:** Topic filters (no medical/legal advice), profanity filter, PII detection.

### Legal document summarization (sensitive)
- **Model:** Claude Opus (large context + vision)  
- **RAG:** semantic chunking, Titan embeddings, hybrid retrieval, citation prompting.  
- **Evaluation:** Bedrock evaluation job with legal metrics and LLM‑as‑a‑judge + human review.  
- **Governance:** strict logging redaction, VPC endpoints, compliance review.

---

## 17. Final two‑page cheat sheet (condensed)

### Page 1 — Quick decision checklist
- Task → Modality → Latency/Cost → Architecture constraints → Customization  
- If migrating OpenAI SDK → use **bedrock‑mantle**.  
- If vendor neutrality → use **Converse**.  
- For RAG: choose embedding model separately from generator.

### Page 2 — Must‑know commands & metrics
- **APIs:** InvokeModel, Converse, ChatCompletions, Responses, Messages.  
- **Metrics:** Invocations, InvocationLatency, Input/OutputTokenCount, OTPS, InferenceErrors, TokenBillingUnits.  
- **Logs:** Model Invocation Logging (S3/CloudWatch), CloudTrail.  
- **Guardrails:** ApplyGuardrail permission + logging of triggers.

---

## 18. Suggested study & prep actions (next 48 hours)
1. **Memorize the 5 elimination questions** for model selection.  
2. **Practice 5 whiteboard scenarios**: high‑volume chat, RAG legal, multimodal doc analysis, agentic tool chain, fine‑tuning pipeline.  
3. **Prepare 3 short architecture diagrams** and rehearse explaining tradeoffs in 90 seconds each.  
4. **Review CloudWatch metrics and CloudTrail basics** and be ready to explain OTPS and token billing.  
5. **Practice guardrail IAM and logging**: be able to recite which actions go on which resource.

---

## 19. Closing: how to use this doc in the loop
- **Start answers with the decision framework.** Interviewers want process.  
- **Use concrete examples** from the mapping table.  
- **Mention observability and safety early** — these are differentiators.  
- **If pressed, dive into PT, OTPS, and RAG retrieval debugging.** That’s where seniority shows.

---
