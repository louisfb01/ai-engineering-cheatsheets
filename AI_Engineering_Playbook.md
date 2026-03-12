**Towards AI’s AI Engineering Playbook – Decision Frameworks**

By Louis-Francois Bouchard ([LinkedIn](https://www.linkedin.com/in/whats-ai/), [X](https://twitter.com/Whats_AI), [YouTube](https://youtube.com/whats_ai), [other AI resources here](https://links.louisbouchard.ai/))

Goal: Pick the right technique, architecture, model, and pipeline for your AI project, using actionable decision frameworks and proven patterns.

## **1\. Which AI Technique Should I Start With?**

**Progression principle:** Prompting → CAG → RAG → Workflows → Agents → Multi-Agent. Advance only when evaluation shows the current level is insufficient.

| Q | Question | If… | → Use This |
| :---- | :---- | :---- | :---- |
| 1 | Does the model already know enough? | System prompt \+ few-shot achieves sufficient quality on eval set | **Prompting**: stop here |
| 2 | Do you need external context? | Yes, but context is known at query time or full KB \< 200K tokens | **CAG / Long context**: inject directly into prompt; use prompt caching for large static context |
| 3 | Is context unknown until query time? | Private, recent, or domain-specific data needed; citations required | **RAG**: semantic search to find and inject relevant context |
| 4 | Are there multiple steps in a fixed sequence? | Combine retrieval, processing, validation in predetermined order | **Workflow**: orchestrate LLM calls \+ tools \+ logic with code |
| 5 | Must the system take actions or branch dynamically? | API calls, DB writes, multi-step reasoning where the next step depends on findings | **Agent \+ Tools**: LLM decides next steps dynamically |
| 6 | Do you need multiple specialized agents? | \>20 tools after trying dynamic discovery, distinct competencies requiring isolated reasoning, or security/compliance boundaries | **Multi-Agent**: orchestrator → workers with explicit handoffs |

| Default: If you are uncertain, start with prompting or CAG. Only move down in the progression when evaluation data proves the simpler approach is insufficient. Each step increases cost (\~2× to \~15× more tokens), latency, and debugging complexity. Stay as simple as you can. |
| :---- |

### **Should I Fine-Tune?**

**Fine-tuning** is orthogonal to the progression above. It changes model *behavior* (style, tone, format), not architecture. Default to PEFT/LoRA.

| Question | If Yes… | If No… |
| :---- | :---- | :---- |
| Is the issue behavior (style, tone, format) rather than knowledge? | Fine-tuning may help | Use CAG/RAG for knowledge instead |
| Can prompting (few-shot, role prompting) achieve the desired behavior? | Stop; no fine-tuning needed | Fine-tuning is a candidate |
| Do you have enough labeled examples for the target behavior? | Proceed with fine-tuning | Collect more data first, or use few-shot |

### **How Do I Choose Between Workflow, Agent, and Multi-Agent?**

| Q | Question | Answer Signal | → Build This |
| :---- | :---- | :---- | :---- |
| 1a | Task shape: sequential or branching? | **Sequential** / linear steps → stable order | **Workflow** |
| 1b |  | **Branching** / looping → conditional paths, retries | **Agent** (advanced branching with uncertainty) |
| 2a | Reasoning mode: deterministic or exploratory? | **Deterministic**: rules, constraints, known checks, strict formats | **Workflow \+ tools** |
| 2b |  | **Exploratory**: search/pivot, unknown next step | **Agent \+ tools** |
| 3a | How many tool types? | **≤10–20** tools → reliable selection | **Single agent** |
| 3b |  | **\>20** heterogeneous tools → selection degrades | First try dynamic tool discovery; split by domain only if reasoning must be isolated → **Multi-agent** |
| 4a | Can it decompose into distinct competencies? | **Yes**: phases with different reasoning modes (e.g., research vs writing) | **Multi-agent** with explicit handoffs |
| 4b |  | **No**: tightly coupled, global context needed | **Single agent** or **Workflow** |

### **Common Pitfalls**

| Symptom | Root Cause | Fix |
| :---- | :---- | :---- |
| Deploying RAG when context is known at query time | Unnecessary retrieval infra | Use CAG or prompting instead |
| Building agents when steps are predictable | Over-engineering | Use deterministic workflows; they are more reliable, cheaper, and easier to debug |
| Fine-tuning to inject knowledge | Wrong technique for the goal | Use CAG/RAG for knowledge; use fine-tuning for behavior only |
| Fine-tuning without RAG context in training data | Train/production mismatch | Include retrieved context in fine-tuning examples if production uses RAG |
| Agent calls wrong tool | Tool definitions unclear | (1) Improve tool descriptions with examples. (2) Reduce tool count (\< 20). (3) Few-shot tool use examples. |
| Agent loops infinitely | No termination condition | (1) Set max iterations. (2) Explicit success/failure conditions. (3) Loop detection. (4) Timeout. |

## **2\. Which Model Should I Use?**

| Constraint | → Use |
| :---- | :---- |
| Prototyping / exploring | Proprietary APIs (e.g., OpenAI, Anthropic, Google) |
| Low budget / high volume | Smaller/lightweight models (e.g., GPT-5-mini, Claude Haiku 4.5, Gemini 3 Flash) |
| Complex reasoning / high-stakes | Frontier models (e.g., GPT-5.2, Claude Opus 4.6, Gemini 3 Pro) |
| Multi-step planning / ambiguous problems | Reasoning models (e.g., GPT-5.2, Claude Opus 4.6, Gemini 3 Pro, DeepSeek-V3.2, GLM-5) |
| Data privacy non-negotiable | Open-weights, self-hosted (e.g., Llama 3.3 70B, DeepSeek V3.2, Qwen3 via Ollama/vLLM) |
| Real-time UX (\< 2s) | Smaller models \+ aggressive caching \+ streaming |
| Long documents (\> 100K tokens) | Large-context models (e.g., Gemini 3 Pro 1M, Claude Opus 4.6, GPT-5.2) |

| 💡 Use Artificial Analysis (artificialanalysis.ai) to compare models on speed, price, and quality benchmarks before choosing. |
| :---- |

### **Which Embedding Model Should I Use?**

| Condition | → Do This |
| :---- | :---- |
| Prototyping | Start with text-embedding-3-small or BGE-M3 |
| Production | Consult MTEB (Retrieval filter). Top: NV-Embed, Cohere embed-v4, text-embedding-3-large |
| Domain-specific vocabulary / low retrieval metrics / query-doc language mismatch | Fine-tune embeddings |
| Changing embedding model | **Re-embed entire knowledge base** (non-negotiable: old embeddings are incompatible) |

### **Common Pitfalls**

| Symptom | Root Cause | Fix |
| :---- | :---- | :---- |
| High cost | Wrong model tier | Right-size per subtask: small for routing, frontier only for reasoning. |
| Small model underperforms | Task too complex | (1) Larger model. (2) Few-shot examples. (3) Break into subtasks. (4) Distillation. |
| Quantized model quality drop | Too aggressive quantization | (1) 8-bit instead of 4-bit. (2) Try Q5\_K\_M, Q8\_0. (3) Larger base model with 4-bit. |

## **3\. How Should I Structure My Prompts?**

### **Which Prompting Technique Should I Use?**

| Condition | → Technique |
| :---- | :---- |
| Straightforward task, no examples needed | Zero-shot: direct instruction |
| Need specific style, structure, or edge-case handling | Few-shot: 2–3 exemplars with consistent formatting (XML tags, delimiters) |
| Complex reasoning required | Chain-of-thought: “Think step by step” in \<thinking\> tags |
| Need specific tone/depth modulation | Role prompting: assign expertise/persona |
| Using reasoning models | **Simpler is better**: direct, high-level instructions. Remove step-by-step guidance. |
| Output consumed by code (not displayed to users) | **Structured outputs**: constrained decoding (strict: true, JSON Schema, Pydantic/Zod) |

### **What Security Layers Are Non-Negotiable?**

| Layer | Action |
| :---- | :---- |
| 1 | Input validation and sanitization before LLM processing |
| 2 | Clear delimiters (XML tags, triple backticks) isolating untrusted input |
| 3 | Explicit guardrails in system prompt \+ few-shot rejection examples |
| 4 | Output validation: sanitize all LLM responses before executing actions |
| 5 | Input guardrails (jailbreak detection, relevance filtering) \+ output guardrails (blocklist, safety classification) |
| 6 | Privilege minimization: LLMs receive only minimum access required |
| 7 | Monitor for anomalous instruction patterns and output deviations |

| Iteration loop: Draft → test (diverse \+ adversarial inputs) → evaluate (LLM-as-judge or human) → refine → version-control prompts in Git. |
| :---- |

### **Common Pitfalls**

| Symptom | Root Cause | Fix |
| :---- | :---- | :---- |
| Inconsistent output format | No structured output constraints | Use structured outputs with JSON Schema (strict mode). |
| Reasoning models underperform | Over-specified prompts | Simplify: direct, high-level instructions. Remove step-by-step guidance. |

## **4\. How Should I Build My RAG Pipeline?**

### **How Should I Chunk My Documents?**

| Document Type / Need | → Strategy |
| :---- | :---- |
| Default / unknown structure | Fixed-size: 500–800 tokens, 10–15% overlap, sentence-aware splitting |
| Topic-rich documents, varied content | Semantic chunking: split by topic/section boundaries |
| Well-structured docs (headings, lists) | Document-structure chunking: split by natural boundaries |
| PDFs with complex layouts, tables | Layout-aware parsing: detect headings, paragraphs, tables; include ancestor headings |
| Tables or code blocks | **Never split**: detect boundaries with code; store as complete markdown/HTML |
| Images/charts | (1) Multimodal embeddings \+ image URL, or (2) Vision model description at ingestion → store in chunk\_text |

### **How Do I Improve Retrieval Step by Step?**

Apply in order. Stop when evaluation metrics are satisfactory.

| Step | Technique | When to Add |
| :---- | :---- | :---- |
| 1 | **Baseline**: semantic search, top\_k \= 5, chunk\_size \~800 | Always start here. Record Hit Rate \+ MRR. |
| 2 | Audit data quality | Before any tuning, clean, de-duplicate, and fill gaps |
| 3 | Increase top\_k (10–15) | Need wider recall (accept more noise) |
| 4 | Add reranker (cross-encoder) | Top-k results contain noise. Retrieve 20–150 → rerank → select top 3–5 |
| 5 | Optimize chunking | Test sizes \+ overlap \+ contextual prefixes |
| 6 | **Hybrid search** (vector \+ BM25 via RRF) | Technical terms, acronyms, exact-match identifiers failing. Recommended production default. |
| 7 | **Contextual retrieval** | Retrieval precision is low despite clean data. LLM-generated 50–100 token context prefix per chunk; embed enriched text, generate from raw chunk |
| 8 | Agentic RAG (CRAG, Self-RAG) | Retrieval quality varies; need self-correction or web fallback |

### 

### **Common Pitfalls**

| Symptom | Root Cause | Fix |
| :---- | :---- | :---- |
| Retrieved docs irrelevant | Poor embedding model or data quality | (1) Test alternative embedding models on your domain. (2) Audit and clean source data. (3) Adjust chunk\_size/overlap. (4) Add metadata filtering. (5) Try hybrid search (vector \+ BM25). |
| Hallucination despite RAG | Retrieval failing or model ignoring context | (1) Check Hit Rate/MRR first. (2) Position retrieved docs at prompt start/end. (3) Add: “Answer ONLY from provided context.” (4) Context sufficiency check before generation. (5) Use stronger model. |
| Good retrieval, poor generation | Model ignoring context or prompt issue | (1) Reduce chunk size (500–800 tokens). (2) Position context at start or end, never middle. (3) Reduce total context length. (4) Improve prompt structure (XML tags). (5) Add few-shot examples. (6) Use stronger model. |
| Tables/code malformed in output | Chunking breaking structure | (1) Detect table/code boundaries; never split. (2) Store as markdown/HTML. (3) Instruct to preserve formatting. |

# **5\. How Should I Manage Context and Memory?**

### **Which Memory Pattern Should I Use?**

| Condition | → Pattern |
| :---- | :---- |
| Short conversations | Full history: append all prior messages |
| Long conversations, cost-sensitive | Sliding window: retain last N turns |
| Multi-turn RAG (default) | **Condense \+ Retrieve**: LLM rewrites history into standalone query → retrieval |
| Very long sessions | Summary memory: periodically summarize, carry forward |
| Cross-session memory / personalization | External memory: store past interactions as embeddings in vector store |

### **Common Pitfalls**

| Symptom | Root Cause | Fix |
| :---- | :---- | :---- |
| Context overflow in long conversations | No memory management | Implement sliding window, summarization, or condense \+ retrieve. |
| Agent “forgets” earlier context | Summarization too aggressive | (1) Preserve critical entities in structured memory. (2) Hierarchical memory. (3) Increase summary budget. |
| Performance degrades in long conversations | Context rot | (1) Compaction: summarize and reinitialize. (2) Keep under 10K tokens when possible. (3) External memory for long-term facts. |

# **6\. How Should I Evaluate My System?**

### **What Should I Measure?**

**For RAG applications**: evaluate retrieval and generation independently. If retrieval is broken, generation improvements are futile. Fix retrieval metrics first, then evaluate generation.

| Component | Metrics | Minimum Eval Set |
| :---- | :---- | :---- |
| Retrieval | Hit Rate, MRR, Precision@k, nDCG | Q\&A pairs with expected source docs |
| Generation (faithfulness) | Faithfulness, Groundedness, Citation accuracy | Same golden dataset |
| Generation (quality) | Relevancy, Correctness, Completeness | Same golden dataset |

**For non-RAG applications**: focus on task-specific output quality.

| Aspect | Metrics | Minimum Eval Set |
| :---- | :---- | :---- |
| Task accuracy | Correctness, Exact match, F1 | Representative input/output pairs covering edge cases |
| Output quality | Coherence, Helpfulness, Completeness | Same dataset \+ adversarial inputs |
| Format compliance | Schema adherence, Structure validity | Inputs that stress output format requirements |
| Safety | Refusal rate on adversarial inputs, Guardrail pass rate | Adversarial \+ out-of-scope test cases |

### **Should I Use LLM-as-Judge?**

| Condition | → Evaluation Method |
| :---- | :---- |
| Output is deterministic or verifiable (code correctness, math, factual lookup) | **Deterministic evals**: unit tests, exact match, regex, assertions |
| Output has a valid structured format (JSON, SQL, API calls) | **Schema validation**: programmatic checks against expected structure |
| Output quality is subjective (style, tone, helpfulness, creativity) | **LLM-as-judge**: automated qualitative scoring |
| Comparing two outputs for relative quality | **LLM-as-judge (pairwise)**: present both, judge selects better |
| Need multi-dimensional quality assessment (faithfulness \+ relevancy \+ completeness) | **LLM-as-judge (rubric)**: single output against criteria (binary) |
| High-stakes decisions or evaluating the judge itself | **Human evaluation**: gold standard, use to calibrate automated evals |

| Start with deterministic evals wherever possible. Layer LLM-as-judge only for dimensions that can’t be checked programmatically. Always validate LLM-as-judge against human labels before trusting at scale. No matter how automated your eval pipeline is, always keep human review in the loop. |
| :---- |

### **When Should Humans Review?**

| Eval Stage | Human Action | Cadence |
| :---- | :---- | :---- |
| Initial calibration | Label 50–100 examples; compare against LLM-as-judge scores | Once, before trusting automated evals |
| Ongoing spot-check | Random-sample 5–10% of automated judgments; flag disagreements | Weekly or per release |
| Edge-case triage | Route low-confidence or borderline scores to human review | Continuous |
| Eval drift detection | Re-score a fixed “canary” set with humans; compare to automated scores over time | Monthly or after model/prompt changes |

### **How Do I Set Up LLM-as-Judge?**

| Decision | → Recommendation |
| :---- | :---- |
| Which model to judge | Most capable model available |
| Pairwise vs rubric | Pairwise for A/B testing; rubric for multi-dimensional absolute quality |
| Bias mitigation | Randomize response order (position bias), control for length (verbosity bias), cross-check across model families |
| Scoring | Require chain-of-thought before score. **Prefer binary (pass/fail) per criterion**: easier to define, lower variance than Likert scales. Specific, measurable criteria. Temperature \= 0\. |
| Validation threshold | Human-labeled representative set. Strong agreement (Cohen’s Kappa) → scale automated eval. |

### **Common Pitfalls**

| Symptom | Root Cause | Fix |
| :---- | :---- | :---- |
| Generation quality low despite tuning (RAG) | Retrieval is broken | Evaluate retrieval and generation independently; fix retrieval first. |
| LLM-as-judge disagrees with humans | No validation against human labels | Calibrate with a representative human-labeled set; require strong agreement (Cohen’s Kappa). |
| Using LLM-as-judge for verifiable outputs | Wrong eval method | Use deterministic evals (tests, exact match, schema validation) for verifiable dimensions. |
| Evals pass but production quality is poor | Golden dataset not representative | Log production queries; add failing cases to eval set continuously. |

# **7\. How Should I Optimize for Production?**

### **How Should I Use Caching?**

| Provider | How It Works | Cost Reduction |
| :---- | :---- | :---- |
| OpenAI | Automatic for prompts ≥1,024 tokens. No code changes. Exact prefix match. | Cached tokens: 50% discount (up to 90% cost, 80% latency reduction) |
| Anthropic | Explicit cache\_control breakpoints (up to 4). Exact prefix match. | Cache reads: 0.1× base input. Up to 90% cost, 85% latency reduction. |
| Google Gemini | Implicit (auto) or explicit. Min 2,048 tokens. Supports multimodal. | Cached tokens: 10% of standard cost (90% discount) |

| Best practice: Static content (system prompt, tools, examples) at prompt beginning. Variable content (user input) at end. Cache hits require exact prefix matches. |
| :---- |

### **How Do I Reduce Costs?**

| If… | → Then… |
| :---- | :---- |
| High cost, haven’t right-sized models | Use cheap models for routing/classification, frontier only for synthesis |
| High input token cost | Reduce top\_k, tighten metadata filters, reduce context length |
| Repetitive queries | Implement prompt caching \+ semantic caching |
| Batch-compatible workload | Use batch APIs for offline processing |

### **How Should I Quantize Self-Hosted Models?**

| Constraint | → Format |
| :---- | :---- |
| Minimal quality loss acceptable | FP16/BF16 (\~50% size reduction) |
| Default for self-hosting | INT8 (\~75% size reduction, negligible quality loss) |
| Hardware-constrained / edge | INT4 (\~87.5% size reduction, noticeable quality loss) |
| Using llama.cpp | GGUF (Q5\_K\_M or Q8\_0 recommended; Q4\_K\_M if hardware-constrained) |

| Start with 8-bit. Drop to 4-bit only if hardware constraints require it. Always evaluate quantized models on your tasks before production. |
| :---- |

### **Why Is Observability Non-Negotiable?**

Trace prompts, tool calls, token usage, latency breakdown, and quality scores per request. Tools: LangSmith, Langfuse (MIT), Opik (Apache 2.0), Phoenix, or structured logs. **No observability \= no reliable improvement.**

### **How Do I Keep Improving After Launch?**

User feedback (explicit \+ implicit) → Monitor metrics (latency, cost, quality) → Log and review (low scores → test cases; high scores → few-shot/fine-tuning data) → A/B test all changes before full deployment.

### **Common Pitfalls**

| Symptom | Root Cause | Fix |
| :---- | :---- | :---- |
| High latency | Multiple bottlenecks | (1) Streaming. (2) Prompt caching. (3) Reduce top\_k. (4) Smaller model for simple queries. (5) Semantic caching. |
| Cache hit rate low | Prompt not cache-friendly | (1) Static content at beginning. (2) Variable content at end. (3) Exact prefix matches. |
| Can’t reproduce issues | No observability | Implement tracing: log prompts, tools, tokens, latency per request. |

This decision framework (and the engineering patterns behind it) is exactly what we teach in the Towards AI courses.  
**Learn more in our full course:** [Full Stack AI Engineering](https://academy.towardsai.net/courses/beginner-to-advanced-llm-dev?ref=1f9b29)