# Model Evaluation

Evaluation can be of multiple types
- Model Evaluation
- RAG Evaluation
- Post Training and Fine Tuning Model Evaluation and comparison against the base model

## What is evaluation

Its the tradeoff between:
   - Quality
   - Latency
   - Cost

## Why is evaluation important
- Make quality, cost and latency tradeoff 
- Align for the organization specific use case
- Evaluate with company data
- Monitor biases, safety and trust

## Amazon Bedrock Model Evaluation
- Bring in your own data set for tailored result
- Use automatic algorithms or LLM As A Judge or human evaluation methods
- Leverage in house team or aws managed reviewers
- Predefined and custom metrics
- Evaluate any model or app hosted anywhere

## Choice of Evaluation Methods

### Programmatic Evaluatiion
- Acccuracy
- Robustness
- Toxicity

    - Metrics
        - BERT Score
        - Classification Metrics
        - F1
        - Real world knowledge score


### Human Evaluation
- Creativity
- Style
- Tone
- Accuracy
- Consistency
- Brand Voice
    
    - Metrics
        - Like/Dislike
        - 5 point Likert scale
        - Binary Choice Button
        - Ordinal Ranking

### LLM as a Judge
- Correctness
- Completeness
- Helpfulness
- Relevance
- Coherence
- Readability
    
    - Metric
        - Multistep reasoning
        - Corelation with expert human evaluators


## LLM as a Judge
- Correctness
- Completeness
- Helpfulness
- Relevance
- Coherence
- Readability
- Harmfulness
- StereoTyping
- Answer refusal
- Custom Metrics 


In LLM as a Judge, do evaluation on two entities
- Either we can do evaluation on a bedrock LLM or or one custom models.
- Or we can do evaluation on our own inferences responses\

- In both case we can define the system metrics and the custom metrics.
- Custom metrics are nothing but system prompts for evaluations.

- Then we pass on the model or the source bucket for inferences responses.

- We can set scores against the quality of repsponses and then run through the model.
- Then it will give us the evaluation score.

## Amazon RAG Evaluation
- Useful for properetary data
- Reduce hallucination by grounding the prompt.


## Special challenges with RAG evalualtion
- Use relevant data from your knowledge base
- Retrieve the right context from documets
- Generate a correct complete and grounded answer minimizing hallucinations
- Iteratively imporve RAG system and compare across changes
- Evaluate biases, safety and trust.

## Amazon Bedrock RAG Evaluation
- Bring in your own data set for tailored result
- Evaluate retrieval alone or retrieval + generation with a choice of LLM as a judge
- Built in metrics of quality, create custom metrics, compatible with AWS Bedrock Guardrails
- Compare across multiple evaluation jobs
- Evaluate any RAG system hosten anywhere


## Metrics
- Retrieval
    - Context Coverage
    - Context Relevance

- Retrieve and Generate
    - Correctness
    - Completeness
    - Helpfulness
    - Citation precision
    - Citation coverage
    - Logical coherence
    - Faithfullness

- Responsible AI
    - Harmfulness
    - Stereotyping
    - Refusal

The process is same for LLM as  A judge.

1. Select the RAG Knowledge Base
2. Select whether evalation is to be done on retreival only or retrieval and response
3. Select system metrics
4. Select custom metrics
5. Select s3 prompt source
6. Select s3 evaluation result desitination
7. IAM role


