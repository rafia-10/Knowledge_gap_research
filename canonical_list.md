# Week 12 Canonical List — Papers, Tools, and Patterns to Keep

**Program:** 10 Academy TRP · Week 12  
**Author:** Rafia  
**Date:** 2026-05-08

This is the reference shortlist produced by Week 12's knowledge gap research. Each entry is annotated with why it is load-bearing for the Tenacious project work and what specific mechanism it resolves.

---

## Canonical Papers

### 1. LoRA: Low-Rank Adaptation of Large Language Models
**Citation:** Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., & Chen, W. (2021). *LoRA: Low-Rank Adaptation of Large Language Models.* arXiv:2106.09685.

**Why it's load-bearing:**
This paper defines the ΔW = B × A decomposition, the α/r scaling invariant, and the empirical recommendation to target q_proj and v_proj. It is the mathematical ground truth behind every LoRA configuration in the Tenacious-Bench training pipeline. Gaps 2 and 3 are both resolved by reading this paper carefully rather than copying tutorial defaults.

**Specific mechanisms it covers:**
- The rank-r bottleneck and why adaptation has low intrinsic rank
- The (α/r) forward-pass scaling factor and why α = 2r is the stable convention
- Why q_proj and v_proj are the efficient targets vs. full-layer adaptation
- Parameter count math for any architecture (Equation 2)

---

### 2. Efficient Memory Management for Large Language Model Serving with PagedAttention
**Citation:** Kwon, W., Li, Z., Zhuang, S., Sheng, Y., Zheng, L., Yu, C. H., Gonzalez, J. E., Zhang, H., & Stoica, I. (2023). *Efficient Memory Management for Large Language Model Serving with PagedAttention.* SOSP 2023.

**Why it's load-bearing:**
The foundational vLLM paper. It explains how KV cache tensors are chunked and managed across requests in paged VRAM blocks — the mechanism that makes prefix caching possible. Understanding the paging model is necessary to reason about what "cache hit" and "cache miss" actually mean at the GPU memory level, which directly informs how to structure prompts in `conversion_engine.py`.

**Specific mechanisms it covers:**
- KV cache block structure and VRAM paging
- Copy-on-write for parallel sampling
- Why KV cache memory fragmentation dominated inference cost before PagedAttention

---

### 3. SGLang: Efficient Execution of Structured Language Model Programs
**Citation:** Zheng, L., Yin, L., Xie, Z., Huang, J., Sun, C., Yu, C. H., Cao, S., Kozyrakis, C., Stoica, I., Gonzalez, J. E., Barrett, C., & Sheng, Y. (2024). *SGLang: Efficient Execution of Structured Language Model Programs.* arXiv:2312.07104.

**Why it's load-bearing:**
This paper introduces RadixAttention — the Radix tree data structure that implements exact-prefix cache matching across requests. The invalidation rule (any diverging token invalidates the entire downstream cache) is specified here. This is the precise mechanism behind Gap 1's core insight about prompt ordering.

**Specific mechanisms it covers:**
- RadixAttention Radix tree construction and lookup
- Exact-match prefix invalidation rule
- Why the static-first, variable-last prompt pattern is required for high hit rates

---

### 4. ORPO: Monolithic Preference Optimization without Reference Model
**Citation:** Hong, J., Lee, N., & Thorne, J. (2024). *ORPO: Monolithic Preference Optimization without Reference Model.* arXiv:2403.07691.

**Why it's load-bearing:**
ORPO is the training objective used for Tenacious-Bench judge training. Understanding ORPO's beta parameter and the odds-ratio loss function is the complement to understanding LoRA's rank/alpha configuration. The two papers together fully define the training setup in `train_judge.py`. The beta configuration decisions in `hyperparams.json` are justified by this paper.

**Specific mechanisms it covers:**
- Preference optimization without a separate reference model
- The monolithic SFT + alignment loss formulation
- How beta controls the weight of the preference alignment term

---

### 5. LIMA: Less Is More for Alignment
**Citation:** Zhou, C., Liu, P., Xu, P., Iyer, S., Sun, J., Mao, Y., Ma, X., Efrat, A., Yu, P., Yu, L., Zhang, S., Ghosh, G., Lewis, M., Zettlemoyer, L., & Levy, O. (2023). *LIMA: Less Is More for Alignment.* NeurIPS 2023.

**Why it's load-bearing:**
LIMA provides the theoretical backing for why fine-tuning shifts style and alignment distributions at the weight level in ways that zero-shot or few-shot prompting fails to replicate. This is the supporting literature for Gap 4's Walrus Effect argument — that prompt engineering cannot override a strong pre-training prior on "professional tone" while a small, high-quality fine-tuning set can permanently reweight the distribution.

**Specific mechanisms it covers:**
- How 1,000 carefully curated fine-tuning examples shifts alignment behavior durably
- Why alignment is a surface-level distribution shift on top of pre-trained capabilities
- The implication that prompt engineering fights the model's gravity while fine-tuning moves it

---

## Canonical Tools and Patterns

### Pattern 1: Static-First Prompt Ordering for Radix Tree Hits
**What it is:** Always place the static system prompt tokens at the absolute start of the messages array, followed by the variable user/lead data at the tail.

**Why to keep it:** Guarantees maximum prefix cache hit rates on vLLM/SGLang backends. A single variable token at position 3 invalidates the entire downstream cache. This pattern was validated in `conversion_engine.py` and reduces redundant prefill computation across sequential lead processing.

**How to apply:** Check every API request builder in the codebase. If any variable interpolation appears before the static system instructions, invert the order.

---

### Pattern 2: OpenRouter Cache Hit Monitoring
**What it is:** Read `response.usage.prompt_tokens_details.cached_tokens` from every OpenRouter API response.

**Why to keep it:** The only reliable way to verify that the Radix tree is actually hitting in production. If `cached_tokens` is 0 across sequential calls with identical system prompts, the prompt order is wrong or the downstream provider does not support prefix caching. This monitoring should be part of any latency investigation in the pipeline.

**How to apply:** Add logging to the API response handler in `agent/llm.py` to surface this value in the benchmark output.

---

### Pattern 3: PEFT Configuration Template (q_proj + v_proj, r=16, α=32)
**What it is:** The validated LoRA configuration for adapter training on instruction-following models for multi-criterion evaluation tasks.

```json
{
  "r": 16,
  "lora_alpha": 32,
  "target_modules": ["q_proj", "v_proj"],
  "lora_dropout": 0.05,
  "bias": "none"
}
```

**Why to keep it:** This configuration provides exactly 16 independent semantic dimensions — sufficient for encoding 5 simultaneous tone markers without destructive interference — while staying within T4 GPU VRAM limits by excluding MLP layers. The α/r = 2.0 invariant is the empirically validated scaling factor from Hu et al. Do not change r without changing α proportionally.

**When to change r:** Increase r only if the adapter shows underfitting on a task with more than ~8 simultaneous evaluation dimensions. Increase α proportionally to maintain the 2.0 scaling factor.

---

### Pattern 4: Logit Distribution Inspection for Prompt vs. Fine-tune Comparison
**What it is:** A diagnostic script using the `transformers` library to print the top-k next-token logit distributions from a base model and a fine-tuned model under identical prompts.

**Why to keep it:** Provides empirical evidence for the Walrus Effect — whether a prompt is merely routing attention to pre-trained clichés vs. whether fine-tuning has genuinely reweighted the vocabulary distribution. Used in Day 4 to verify that the LoRA-trained Tenacious judge actually surfaces "Your" (direct, grounded) as the top token instead of "I" (boilerplate) for a cold outreach prompt.

**How to apply:** Run the comparison whenever a new style-transfer fine-tune is completed to verify weight-level behavioral shift rather than just qualitative review.

---

### Pattern 5: Paired Bootstrap Test for LLM Evaluation Significance
**What it is:** The `paired_bootstrap` function in `ablations/run_ablations.py` that compares per-task score differences between two evaluation conditions.

**Why to keep it:** The correct statistical test for comparing two LLM-judge conditions on the same held-out task set. Unlike a t-test, it makes no normality assumptions — important because LLM rubric scores are typically bounded (0–5) and non-normally distributed. Unlike Wilcoxon, it directly handles paired continuous scores.

**Pending action (Gap 5):** Before the next audit report, validate that the 60/40 blending formula is not introducing correlated variance that artificially reduces test power at n=59. The p=0.189 result should be re-evaluated with a power analysis before being used as a final deployment verdict.

---

## Quick Reference Matrix

| Resource | Gaps it closes | Where it applies |
|---|---|---|
| LoRA (Hu et al., 2021) | Gaps 2, 3 | hyperparams.json, train_judge.py |
| PagedAttention (Kwon et al., 2023) | Gap 1 | conversion_engine.py, README |
| SGLang/RadixAttention (Zheng et al., 2024) | Gap 1 | conversion_engine.py |
| ORPO (Hong et al., 2024) | Gap 2 | train_judge.py, hyperparams.json |
| LIMA (Zhou et al., 2023) | Gap 4 | interim report, presentation |
| Static-first prompt ordering | Gap 1 | conversion_engine.py |
| OpenRouter cache monitoring | Gap 1 | agent/llm.py |
| PEFT config template | Gaps 2, 3 | hyperparams.json |
| Logit inspection script | Gap 4 | evaluation diagnostics |
| Paired bootstrap + power analysis | Gap 5 | ablations/run_ablations.py, audit_memo.md |
