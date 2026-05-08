# Week 12 Knowledge Gap Formulation — Synthesis

**Program:** 10 Academy TRP · Week 12  
**Author:** Rafia  
**Date:** 2026-05-08  
**Scope:** Daily_deliverables_day1 · day3 · day4 · day5

---

## Overview

Week 12 ran a structured research loop: each day I identified a specific gap in my own understanding of a mechanism I had already shipped in Week 10–11 work, sharpened a diagnostic question with a partner, received a peer-written explainer, grounded the closure in an artifact commit, and signed off. Four active days produced five substantive gaps. Day 5 produced a well-sharpened question but the explainer session had not yet occurred by the time of this submission.

The overarching theme connecting all five gaps is the same: **I had shipped working code and defensible results without being able to mathematically defend the mechanisms underneath them.** Every gap was discovered by asking "could I explain why this works to an engineering team?" and discovering the honest answer was no.

---

## Gap-by-Gap Summary

### Gap 1 — KV Cache vs. Prefix Caching (Day 1)

**The gap:** I knew our Tenacious Conversion Engine was slow for sequential lead processing on `qwen3-80b` but could not explain the difference between intra-request KV caching and inter-request prefix caching, could not calculate the GPU memory cost of a cached token, and did not know what invalidates the Radix tree.

**Resolution:** The KV cache stores Key and Value projection tensors per token per layer to avoid O(N²) attention recomputation within a single request. Prefix caching (RadixAttention in vLLM/SGLang) extends this across requests by storing completed sequences in a Radix tree mapped by token IDs. For Qwen3-80B, one cached token costs approximately 328 KB of VRAM. The Radix tree enforces exact prefix matching: the moment a single token diverges, the entire downstream cache is invalidated. This means placing variable user data before the static system prompt gives 0% cache hits, while inverting the order gives near-100% hits across sequential calls. The critical secondary insight: for a 46.6s p50 latency, the bottleneck is the decode phase (reasoning token generation), not the prefill phase. Prefix caching saves 50–100ms of prefill but does nothing to the generation cost of the thinking model.

**Mental model shift:** Latency has two structurally distinct phases. Caching optimizes one. I was confusing them.

---

### Gap 2 — LoRA Rank Capacity Math (Day 2 / Day 3 folder)

**The gap:** In `tenacious-bench/training/hyperparams.json` I had written `r=16, lora_alpha=32` and justified it as "rubric complexity warrants higher rank" — a phrase I could not defend algebraically. I treated rank and alpha as numbers copied from an Unsloth tutorial.

**Resolution:** LoRA freezes the base weight matrix W and instead learns ΔW = B × A, where A ∈ ℝ^(r×k) and B ∈ ℝ^(d×r). The rank r is the inner bottleneck dimension. For a 1536×1536 projection layer, r=16 reduces trainable parameters from 2.36M to 49K — a 98% reduction. Rank defines the maximum number of linearly independent semantic directions the adapter can learn. At r=8 the adapter cannot simultaneously encode gradient signals for all five Tenacious tone markers (Direct, Grounded, Honest, Professional, Non-condescending) without destructive interference; r=16 provides enough geometric width. The alpha term enters the forward pass as h = Wx + (α/r)·ΔWx. Setting α = 2r fixes the scaling factor at 2.0, which (a) empirically produces stable task-specific fine-tuning per the original LoRA paper, and (b) decouples the effective learning magnitude from the rank dimension so that changing r later does not require retuning the learning rate.

**Mental model shift:** Rank is not a memory dial. It is the dimensional capacity of the adapter's semantic vocabulary.

---

### Gap 3 — α/r Scaling and Catastrophic Forgetting (Day 4 — Rafia's question)

**The gap:** I had shipped `r=8, α=16` in the Tenacious-Bench judge training but could not explain what would happen if α were set much higher relative to r — specifically whether and how that causes the base model to lose its instruction-following capability.

**Resolution:** The adapter output (α/r)·ΔWx is added to the base model's layer output at every forward pass. During early training, the random initialization of A and B produces noisy gradient updates. If α/r >> 1, those noisy early updates are amplified and can dominate the base model's well-calibrated logits, overwriting the pre-trained instruction-following priors before the adapter has learned anything meaningful. This is catastrophic forgetting: the adapter noise overwhelms the base signal. Holding α/r = 2.0 is a principled tradeoff — the adapter speaks loudly enough to shift the distribution but not so loudly that it destroys base capabilities in early epochs. The mathematical guarantee is that if you increase r to 32 and correspondingly increase α to 64, the scaling factor remains 2.0 and the learning rate stays valid.

**Mental model shift:** α/r is a signal-to-noise ratio control for the adapter's influence on the base model, not merely a "learning rate modifier."

---

### Gap 4 — Prompting vs. Fine-tuning at the Weight Level (Day 4 — Ephrata's question)

**The gap:** Ephrata had written in his Week 11 blog that "prompt engineering alone can't capture the Tenacious tone — fine-tuning is required." The claim was correct but undefended at the mechanism level.

**Resolution:** A prompt does not alter model weights. It modifies transient activation states — the X in Y = WX — for one inference call. The attention mechanism acts as a spotlight routing information from instruction tokens into the generated output, but it can only illuminate what the base model already associates with the instruction. Since the strongest pre-training association for "professional sales tone" is generic corporate boilerplate, prompting reliably produces boilerplate. This is the Walrus Effect: blunt stylistic constraints produce caricatures (every sentence ends with "Arrr!" when you prompt "act like a pirate"). Tone is a distributed statistical property involving micro-decisions at every token step. Fine-tuning permanently updates the projection matrices — it moves the center of probability mass in the vocabulary distribution rather than shining a spotlight over the existing distribution. After ORPO fine-tuning on Tenacious examples, the model generates "Your recent Q3 filing..." as the top token for a cold outreach prompt (p=0.55) without any explicit tone instruction, because the weight geometry now places that token highest by default.

**Mental model shift:** Prompts rent the model's attention. Fine-tuning buys the weights.

---

### Gap 5 — Bootstrap Statistics for Evaluation (Day 5 — question formulated, explainer pending)

**The gap:** My ablation audit reports p=0.189 (not significant) for the trained LoRA judge vs. baseline, and I used this as a DO NOT DEPLOY signal. I do not understand whether p=0.189 reflects a genuine null effect or whether the 60/40 blended scoring formula (`0.6 * judge_score + 0.4 * machine_score`) introduced correlated variance across conditions that artificially deflated the test's statistical power at n=59.

**Status:** Question sharpened and grounded. The diagnostic formulation identifies three specific sub-questions: (1) how the paired bootstrap constructs its null distribution from resampled score differences, (2) what the minimum detectable effect size is at n=59 with 80% power, and (3) whether the variance reduction from the 60/40 blend is sufficient to explain the observed p-value without a genuine null effect. The explainer session was scheduled but not yet conducted. The DO NOT DEPLOY verdict in `audit_memo.md` should be treated as provisional until this gap is closed.

---

## Cross-Cutting Themes

**1. Mechanism ownership is the difference between shipping and understanding.**
Every gap this week was discovered by the same test: "Could I defend this to an engineering team?" I had correct output but hollow reasoning. The grounding commits show the tangible difference: `hyperparams.json` went from "rubric complexity warrants higher rank" to a precise algebraic justification. That is the difference between a practitioner and an engineer.

**2. The two-phase inference model recurred across multiple gaps.**
Prefill vs. decode (Gap 1), activation state vs. weight state (Gap 4), and early-epoch vs. converged adapter behavior (Gap 3) all share the same structure: a fast, shallow phase and a slow, deep phase, and conflating them leads to wrong diagnoses.

**3. Architectural choices encode constraints, not magic numbers.**
Rank 16, alpha 32, q_proj + v_proj, 60/40 score blend — each of these was copied from a tutorial without understanding. Week 12 established that every hyperparameter in a production ML system should be derivable from first principles or explicitly flagged as an empirical choice pending analysis.

**4. Statistical significance is not the same as a deployment verdict.**
Gap 5 reveals that p=0.189 at n=59 with a blended scoring formula may be a power problem, not a null result. Treating a significance test as a binary ship/don't-ship gate — without understanding the power properties of the test — is a systematic risk in any LLM evaluation pipeline.

---

## What Changed in My Mental Model

| Before Week 12 | After Week 12 |
|---|---|
| KV cache = fast memory, works automatically | KV cache = intra-request; prefix caching = inter-request via Radix tree; prompt order determines cache hit rate |
| LoRA rank = memory knob | Rank = dimensional capacity of the adapter's semantic vocabulary; specific to the complexity of the target task |
| α = learning rate multiplier I copy from tutorials | α/r = invariant scaling ratio; decouples adapter magnitude from rank; prevents catastrophic forgetting if held at 2.0 |
| Fine-tuning is "better" than prompting | Prompts change activations (transient); fine-tuning changes weights (permanent); tone transfer requires weight-level distribution shifts |
| p < 0.05 = ship, p > 0.05 = don't ship | p-value interpretation requires knowing test power, effect size, and whether the evaluation formula introduced correlated variance |
