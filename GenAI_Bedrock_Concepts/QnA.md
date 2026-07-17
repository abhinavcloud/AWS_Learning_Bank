# ⭐ **AWS SA Loop — GenAI & Bedrock Interview Simulation **

---

## **Q1. “Let’s start simple. A customer says: ‘Which model should I use for my GenAI application?’ How do you approach model selection?”**

### **Strong Answer**
> “I never start by naming a model. I start by eliminating models based on five filters:  
> **(1)** Task type (reasoning vs fluency)  
> **(2)** Modality (text, image, video, speech)  
> **(3)** Latency/cost constraints  
> **(4)** Architecture constraints (OpenAI SDK migration → mantle; vendor neutrality → Converse)  
> **(5)** Customization needs (fine‑tuning vs RAG).  
>  
> Only after narrowing the field do I benchmark the remaining models on the customer’s own dataset.”

### **What the interviewer is testing**
- Whether you avoid “model name dropping”  
- Whether you think like an architect  
- Whether you understand Bedrock’s decision framework  

---

## **Q2. “Good. Now suppose the customer needs a chatbot that handles 10,000 daily conversations with strict latency SLAs. Which models would you shortlist and why?”**

### **Strong Answer**
> “This is a high‑volume, low‑latency workload. I’d shortlist **Nova Micro**, **Nova Lite**, and **Claude Haiku**.  
>  
> They offer:  
> - Low latency  
> - Low cost per token  
> - High throughput  
> - PT (Provisioned Throughput) availability for predictable scaling  
>  
> I would avoid frontier models unless the workload requires deep reasoning.”

### **What the interviewer is testing**
- Whether you understand cost/latency tradeoffs  
- Whether you know PT matters more than model size  
- Whether you avoid over‑provisioning  

---

## **Q3. “Now the customer says: ‘We also need the bot to read PDFs and images.’ How does your recommendation change?”**

### **Strong Answer**
> “Modality eliminates most models. I’d move to **Claude Opus**, **Claude Haiku**, **Nova Pro**, or **Pixtral Large**, depending on cost vs quality.  
>  
> These support multimodal input (images/PDFs).  
>  
> I’d still benchmark on their documents because vision quality varies by domain.”

### **What the interviewer is testing**
- Whether you understand modality constraints  
- Whether you know which models support multimodal input  

---

## **Q4. “The customer wants to migrate from OpenAI’s ChatCompletions API with minimal code changes. What do you recommend?”**

### **Strong Answer**
> “I’d recommend using **bedrock‑mantle** models that support **Chat Completions** or **Responses**.  
>  
> This allows near drop‑in compatibility with their existing OpenAI SDK code.  
>  
> If they want vendor neutrality long‑term, I’d still architect the system around **Converse**, but mantle gives them the fastest migration path.”

### **What the interviewer is testing**
- Understanding of mantle vs runtime  
- Ability to reduce migration friction  
- Awareness of API compatibility  

---

## **Q5. “Let’s shift to Guardrails. The customer wants to ensure no medical advice is generated. How would you design this?”**

### **Strong Answer**
> “I’d use **Topic Filters** in Guardrails to block medical advice.  
>  
> I’d configure:  
> - A prohibited topic: ‘Medical advice’  
> - A standard fallback response  
> - Input + output filtering  
>  
> And ensure the invoking resource (Lambda, ECS, Step Functions) has:  
> - `bedrock:InvokeModel` on the model ARN  
> - `bedrock:ApplyGuardrail` on the guardrail ARN.”

### **What the interviewer is testing**
- Whether you understand Guardrail components  
- Whether you know IAM requirements  
- Whether you can apply guardrails to real use cases  

---

## **Q6. “The customer now asks: ‘Can Guardrails prevent hallucinations?’ How do you explain context grounding?”**

### **Strong Answer**
> “Guardrails include **Context Grounding**, which checks whether the model’s output is supported by the provided context.  
>  
> It doesn’t rewrite the answer — it flags or blocks unsupported claims.  
>  
> It’s especially useful in RAG systems where hallucinations come from generation, not retrieval.”

### **What the interviewer is testing**
- Whether you understand hallucination mitigation  
- Whether you know Guardrails don’t filter reasoning traces  

---

## **Q7. “Let’s talk evaluation. The customer wants to compare Claude Opus vs GPT‑5.5 for legal document summarization. How would you design the evaluation?”**

### **Strong Answer**
> “I’d run a **Bedrock Model Evaluation job** with:  
> - Their own legal documents in S3  
> - System metrics: correctness, completeness, coherence  
> - Custom metrics: legal‑specific criteria via system prompts  
> - LLM‑as‑a‑judge for scalable scoring  
>  
> Evaluation jobs are asynchronous and results land in S3.”

### **What the interviewer is testing**
- Understanding of evaluation jobs  
- Ability to design custom metrics  
- Awareness of LLM‑as‑a‑judge limitations  

---

## **Q8. “The customer is building a RAG system. How do you evaluate retrieval quality separately from generation quality?”**

### **Strong Answer**
> “I’d use **RAG Evaluation** in two phases:  
>  
> **Phase 1 — Retrieval Only**  
> - Context coverage  
> - Context relevance  
>  
> This helps debug chunking, embeddings, and indexing.  
>  
> **Phase 2 — Retrieval + Generation**  
> - Correctness  
> - Completeness  
> - Citation precision  
> - Faithfulness  
>  
> This isolates hallucinations caused by generation.”

### **What the interviewer is testing**
- Deep understanding of RAG architecture  
- Ability to isolate retrieval vs generation issues  

---

## **Q9. “The customer complains: ‘Our RAG system still hallucinates.’ What architectural improvements would you suggest?”**

### **Strong Answer**
> “I’d look at:  
> - Chunking strategy (semantic vs fixed)  
> - Embedding model quality (Titan vs Cohere vs Nova Multimodal)  
> - Retrieval method (hybrid search, reranking)  
> - Context window utilization  
> - Guardrails context grounding  
> - Prompting for citations  
>  
> Hallucinations often come from poor retrieval, not the model.”

### **What the interviewer is testing**
- Whether you can diagnose RAG issues  
- Whether you understand retrieval architecture deeply  

---

## **Q10. “Final question. If you had to summarize your GenAI architecture philosophy for customers in one sentence, what would it be?”**

### **Strong Answer**
> “Choose models by elimination, enforce safety with guardrails, ground answers with RAG, and validate everything with evaluation on the customer’s own data — not public benchmarks.”

### **What the interviewer is testing**
- Your ability to articulate a clear architectural philosophy  
- Your ability to simplify complex concepts  
- Your ability to demonstrate leadership principles  

---

Here is a **10‑question, observability‑only mock interview**, crafted exactly like an AWS SA loop: progressive, scenario‑driven, probing deeper with each answer, and focused on Bedrock + GenAI observability.  
Each question is followed by **what a strong candidate should answer** and **what the interviewer is actually testing**.

Use this to practice — this is the level that gets a “Hire”.

---

# ⭐ **AWS SA Loop — Model Observability Mock Interview (10 Progressive Questions)**

---

## **Q1. “Let’s start simple. How do you define observability for GenAI workloads on Bedrock?”**

### **Strong Answer**
> “GenAI observability means tracking latency, cost, safety, and quality across every model invocation.  
>  
> On Bedrock, this is done through CloudWatch metrics, CloudTrail API logs, and Model Invocation Logging to S3 or CloudWatch Logs.”

### **Interviewer is testing**
- Whether you understand observability beyond “monitoring”  
- Whether you know the core AWS tools involved  

---

## **Q2. “What specific metrics do you monitor for Bedrock model invocations?”**

### **Strong Answer**
> “For bedrock‑runtime, I monitor:  
> - Invocation count  
> - Invocation latency  
> - Input/output token counts  
> - Cache read/write tokens  
> - OTPS (Output Tokens Per Second)  
>  
> OTPS is critical because slow token generation increases both latency and cost.”

### **Interviewer is testing**
- Whether you know GenAI‑specific metrics  
- Whether you understand token‑driven cost models  

---

## **Q3. “How do you enable Model Invocation Logging, and what do those logs contain?”**

### **Strong Answer**
> “I configure a destination (S3 or CloudWatch Logs), attach an IAM role Bedrock can assume, and enable logging in Bedrock settings.  
>  
> Invocation logs include:  
> - Input prompt (redacted if configured)  
> - Model output  
> - Input/output tokens  
> - Latency  
> - Guardrail results  
> - Model ARN and request ID.”

### **Interviewer is testing**
- Whether you understand the logging pipeline  
- Whether you know what data is actually logged  

---

## **Q4. “How does observability differ between bedrock‑runtime and bedrock‑mantle?”**

### **Strong Answer**
> “Runtime gives invocation‑level metrics and logs.  
>  
> Mantle adds richer observability:  
> - Project‑level and model‑level metrics  
> - Token billing units  
> - Agentic workflow traces (tool-use, function calls)  
> - More granular error metrics  
>  
> Mantle is designed for OpenAI/Anthropic‑compatible workloads, so its observability aligns with those SDKs.”

### **Interviewer is testing**
- Whether you understand mantle vs runtime  
- Whether you can articulate architectural differences  

---

## **Q5. “How do you use CloudTrail for Bedrock observability?”**

### **Strong Answer**
> “CloudTrail logs every Bedrock API call — InvokeModel, Converse, ChatCompletions, Responses, Messages, and guardrail operations.  
>  
> It’s essential for auditability, debugging production issues, and detecting unexpected usage.”

### **Interviewer is testing**
- Whether you understand audit logging  
- Whether you know Bedrock API surfaces  

---

## **Q6. “A customer complains their model latency is inconsistent. How would you diagnose this?”**

### **Strong Answer**
> “I’d check:  
> - InvocationLatency in CloudWatch  
> - OTPS to see if token generation slowed  
> - First-token latency vs total latency  
> - Cache hit/miss metrics  
> - Provisioned Throughput utilization (if PT is enabled)  
>  
> Then I’d inspect invocation logs for unusually large prompts or spikes in output tokens.”

### **Interviewer is testing**
- Whether you can diagnose real-world latency issues  
- Whether you understand tokenization and PT  

---

## **Q7. “How do you observe safety events, such as guardrail triggers?”**

### **Strong Answer**
> “Guardrail results appear directly in Model Invocation Logs.  
>  
> I monitor:  
> - Blocked inputs  
> - Blocked outputs  
> - Sensitive information detection  
> - Topic filter violations  
>  
> I also set CloudWatch alarms on guardrail-trigger metrics to detect spikes in unsafe content.”

### **Interviewer is testing**
- Whether you understand safety observability  
- Whether you know guardrail outputs are logged  

---

## **Q8. “Let’s talk RAG. How do you observe retrieval quality in a RAG system?”**

### **Strong Answer**
> “I monitor retrieval-only metrics using Bedrock RAG Evaluation:  
> - Context coverage  
> - Context relevance  
>  
> I also log:  
> - Retrieved chunks  
> - Embedding similarity scores  
> - Retrieval latency  
>  
> This helps isolate retrieval issues from generation issues.”

### **Interviewer is testing**
- Whether you understand RAG observability  
- Whether you can separate retrieval vs generation  

---

## **Q9. “How do you observe agentic workflows, such as tool-use or function calling?”**

### **Strong Answer**
> “On mantle, agentic workflows produce detailed traces:  
> - Tool-use events  
> - Function call inputs/outputs  
> - Step-by-step reasoning traces (if enabled)  
>  
> These appear in invocation logs and CloudWatch metrics, helping debug multi-step workflows.”

### **Interviewer is testing**
- Whether you understand agentic observability  
- Whether you know mantle supports tool-use traces  

---

## **Q10. “If you had to summarize your observability strategy for GenAI workloads in one sentence, what would it be?”**

### **Strong Answer**
> “I use CloudWatch for metrics, CloudTrail for API auditing, and Model Invocation Logging for deep visibility into prompts, responses, tokens, guardrail outcomes, and agentic traces — giving me complete insight into latency, cost, safety, and quality.”

### **Interviewer is testing**
- Your ability to synthesize  
- Your ability to communicate clearly  
- Your architectural maturity  

---

