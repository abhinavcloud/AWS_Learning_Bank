# AWS Bedrock — Model Modality, API, and Endpoint Support

> Reference table for GenAI fluency prep (model selection, API design). Compiled from AWS Bedrock's API compatibility and endpoint availability docs.

**Note:** Not every "famous" model is actually on Bedrock. GPT‑4o and Gemini proper aren't there — OpenAI is represented by the open‑weight `gpt-oss` models plus newer GPT‑5.5/5.4 (mantle-only), and Google is represented by open‑weight Gemma, not Gemini.

## API families

| Family | APIs | What it's for |
|---|---|---|
| **Invoke** | `InvokeModel`, `InvokeModelWithResponseStream`, `InvokeModelWithBidirectionalStream`, `AsyncInvoke` | Synchronous, streaming, full‑duplex, and long‑running calls. Universal fallback — nearly every model supports some form of it. |
| **Converse** | `Converse`, `ConverseStream` | Model‑agnostic, unified interface for multi‑turn chat. Only available on "message‑based" (chat) models. |
| **OpenAI‑compatible** | `ChatCompletions`, `Responses` | Lets existing OpenAI SDK code run on Bedrock with minimal changes. `Responses` adds stateful, agentic tool use. |
| **Messages** | `Messages` | Anthropic‑compatible interface. Claude‑only, and only on select model versions. |

## Endpoints

| Endpoint | Supported APIs | Notes |
|---|---|---|
| `bedrock-runtime` | Invoke, Converse, Chat Completions, Messages | The original, broadly‑supported endpoint. Safe default if a model isn't yet on mantle. |
| `bedrock-mantle` | Responses, Chat Completions, Messages | Newer endpoint built for OpenAI/Anthropic SDK compatibility and agentic workflows. Not every model is on it yet. |

## Model support matrix

| Model | Provider | Modality | Invoke | Converse | Messages | Chat Completions | Responses | bedrock-runtime | bedrock-mantle |
|---|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Claude Opus 4.7 | Anthropic | Text, Image (multimodal) | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ |
| Claude Sonnet 4.6 | Anthropic | Text, Image (multimodal) | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Claude Haiku 4.5 | Anthropic | Text, Image (multimodal) | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ |
| Nova Pro / Nova Lite | Amazon | Multimodal (text, image, video-in) | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Nova Micro | Amazon | Text | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Nova Canvas | Amazon | Image (generation) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Nova Reel | Amazon | Video (generation, async-only) | ❌* | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Nova Sonic | Amazon | Speech (bidirectional stream) | ✅* | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Titan Text Embeddings V2 | Amazon | Embeddings (text) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Titan / Nova Multimodal Embeddings | Amazon | Embeddings (multimodal) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| GPT‑5.5 | OpenAI | Text, Image (multimodal) | ❌ | ❌ | — | ✅ | ✅ | ❌ | ✅ |
| gpt-oss-120b | OpenAI | Text | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Llama 4 Maverick | Meta | Text, Image (multimodal) | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Llama 3.3 70B | Meta | Text | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Mistral Large 3 | Mistral AI | Text | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ |
| Pixtral Large | Mistral AI | Text, Image (multimodal) | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| DeepSeek V3.2 | DeepSeek | Text | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ |
| Gemma 3 27B | Google | Text, Image (multimodal) | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ |
| Qwen3 VL 235B | Qwen | Text, Image (multimodal) | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ |
| Command R+ | Cohere | Text | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Embed v4 | Cohere | Embeddings (multimodal) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Marengo Embed 3.0 | TwelveLabs | Embeddings (video/multimodal) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Pegasus v1.2 | TwelveLabs | Video (understanding) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Stable Image family | Stability AI | Image (generation/editing) | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |

\* Nova Reel is `StartAsyncInvoke`-only (not sync `InvokeModel`); Nova Sonic uses `InvokeModelWithBidirectionalStream` rather than plain `InvokeModel`.

## Patterns worth having ready

- **Invoke** is the universal fallback — every model supports it (or its async/bidirectional variant). **Converse** is Bedrock's model-agnostic layer, but only "message-based" (chat) models get it — pure embedding/image/video models don't.
- **bedrock-mantle** is the newer, OpenAI/Anthropic-SDK-compatible endpoint (Responses, Chat Completions, Messages) — AWS's answer to "make migrating off OpenAI/Anthropic direct APIs frictionless." Not every model is on it yet; check `bedrock-runtime` as the safe default.
- **Messages API** (Anthropic-compatible) is Claude-only, and even then only on select versions (Opus 4.7, Haiku 4.5 — not Sonnet 4.6 yet).
- Modality and API support are separate axes — e.g., Nova Reel is video but *only* async, which is a good example if asked about production RAG/agentic design trade-offs around sync vs. async inference.




## The framework: 5 questions before you name a model

When a client asks "which model should I use," walk through these in order — this order matters because each one *eliminates* candidates before you optimize among what's left.

**1. What's the task, and does it need reasoning or just fluency?**
- Simple classification, extraction, short-form chat → small/fast models are enough (Nova Micro, Claude Haiku, Llama 3.3 8B-class)
- Multi-step reasoning, complex agentic tool use, ambiguous instructions → frontier models (Claude Opus/Sonnet, GPT‑5.5, DeepSeek-R1)
- Code generation → coding-specialized (Qwen3 Coder, Devstral, Claude Sonnet)

**2. What modalities are actually in play?**
- Text only vs. needs to read images/PDFs vs. needs to generate images vs. video/speech — this alone rules out most of the catalog immediately (e.g. no point discussing Claude for video generation).

**3. What are the latency/throughput/cost constraints?**
- Real-time customer-facing chat → low-latency, cheap models (Nova Lite/Micro, Haiku)
- Batch/offline processing → can use a bigger, slower, cheaper-per-token model, or even async invoke
- High QPS at scale → check Provisioned Throughput availability, not just on-demand pricing

**4. What are the compliance/architecture constraints?**
- Data residency / region availability (not every model is in every region)
- Does it need to integrate with an existing OpenAI or Anthropic SDK codebase already in production? → steers you toward `bedrock-mantle` (Responses/Chat Completions/Messages) for a low-friction lift-and-shift, vs. `Converse` if they're building fresh and want to stay model-agnostic in case they swap providers later
- Guardrails/safety requirements → check ApplyGuardrail support, especially for marketplace models

**5. Does it need customization?**
- Fine-tuning / continued pre-training needed → narrows to models that support it on Bedrock
- RAG only, no fine-tuning → any strong base model + the right embedding model (this is a separate decision — see below)

## Use-case → model mapping (concrete examples to cite)

| Business case | What matters most | Model direction |
|---|---|---|
| Customer support chatbot, high volume | Latency + cost | Nova Lite/Micro, Claude Haiku |
| Complex internal agent (multi-tool, long chains) | Reasoning quality | Claude Opus/Sonnet, GPT‑5.5 |
| Code review / dev assistant | Code-specific quality | Qwen3 Coder, Devstral, Claude Sonnet |
| Document/contract analysis with images or scans | Vision + long context | Claude, Nova Pro, Pixtral Large |
| Product image generation for e-commerce | Image gen quality/control | Nova Canvas, Stable Image family |
| Video content moderation or search | Video understanding | Nova Pro, TwelveLabs Pegasus/Marengo |
| Voice assistant / IVR | Speech-to-speech latency | Nova Sonic |
| Semantic search / RAG retrieval layer | Embedding quality, not chat quality | Titan/Cohere/Nova embeddings — a *separate* choice from your generation model |
| Migrating an existing OpenAI-SDK app to Bedrock | API compatibility, low rewrite cost | Models on `bedrock-mantle` (Chat Completions/Responses) |
| Regulated industry, need model portability | Vendor neutrality | `Converse` API + any model that supports it, so switching providers later doesn't mean a rewrite |

## How to actually say it in the interview

> "I'd start by clarifying the task type and modality, because that eliminates most of the catalog immediately. Then I'd weight latency/cost against quality based on whether it's real-time customer-facing or batch/internal. For example, if a client says 'we need a support chatbot handling 10K conversations a day,' I'd lean toward Nova Lite or Claude Haiku for cost and latency rather than defaulting to the biggest model — over-provisioning quality you don't need is a common anti-pattern. I'd also ask about existing tooling: if they already have OpenAI SDK code, `bedrock-mantle`'s Chat Completions API minimizes migration cost. I'd validate the choice with a small benchmark against their actual data rather than picking from a leaderboard, since published benchmarks don't always reflect their specific task."

That last line is worth keeping — "benchmark on their own eval set" is the kind of answer that signals Dive Deep / Ownership rather than reciting a spec sheet, which matches what your recruiter flagged.