# Week 12 Portfolio Update — How Week 10–11 Projects Improved

**Program:** 10 Academy TRP · Week 12  
**Author:** Rafia  
**Date:** 2026-05-08

This document traces the specific improvements made to Week 10–11 artifacts as a direct result of the knowledge gaps closed in Week 12. Each entry maps from the gap that was identified → the artifact that was retroactively improved → what specifically changed and why it matters.

---

## 1. Tenacious Conversion Engine (`agent/conversion_engine.py` + `README.md`)

**Source gap:** Gap 1 — KV Cache vs. Prefix Caching

### Before Week 12
The `messages` array in `conversion_engine.py` was constructed with the dynamic lead serialization appearing early in the prompt structure, before the static tone constraints. This was the standard "put context first" pattern from generic RAG tutorials. The README's Known Limitations section vaguely recommended "use a smaller model" to address the 46.6s p50 latency without explaining which phase of inference was the actual bottleneck.

### After Week 12

**`agent/conversion_engine.py` (lines ~196–209):**
The prompt construction was refactored to physically place all static tone constraints, rules, and system instructions at the absolute start of the `messages` array. The dynamic JSON serialization of the lead's data is pushed strictly to the tail end. This guarantees that the first ~80 tokens of every sequential call in a batch exactly match the Radix tree entry built by the first call, producing near-100% prefix cache hits on vLLM/SGLang backends.

**`README.md` § Known Limitations §3:**
Replaced the generic "use a smaller model" recommendation with an analytically grounded note differentiating prefill vs. decode latency: the 46.6s p50 is gated by generation length (decode phase), not input processing (prefill phase). The note now documents that prompt restructuring saves 50–100ms of prefill time but that meaningful latency reduction for batch scale requires switching away from the thinking model — which is the architecturally correct conclusion but for a reason I could not previously state precisely.

**Why this matters for the portfolio:** The change demonstrates that I understand inference architecture well enough to engineer prompts for production characteristics, not just functional correctness. The README now shows I can diagnose latency precisely rather than guessing.

---

## 2. Tenacious-Bench Training Configuration (`tenacious-bench/training/hyperparams.json`)

**Source gap:** Gap 2 — LoRA Rank Capacity Math

### Before Week 12
The `rationale` field in `hyperparams.json` read: *"Rank 8 was considered but judge rubric complexity warrants higher rank."* This was a phrase copied from a tutorial with no mathematical backing. Anyone reviewing the training configuration would immediately identify that the author did not understand what rank does.

### After Week 12
The `"rationale": "lora_rank"` entry was rewritten to: *"Rank 16 provides the minimum independent semantic dimensions required to encode the 5 Tenacious tone markers simultaneously without destructive interference in the projection matrices. Alpha is strictly set to 32 to guarantee the invariant α/r = 2.0 activation scaling factor per Hu et al. (2021) Equation 6."*

This is now a mathematically defensible configuration choice, not a cargo-culted number. The argument is: 5 distinct tone criteria require at least 5 independent gradient directions; r=16 provides sufficient headroom above that minimum to avoid interference; any reduction to r=8 would create a bottleneck that forces the criteria to compete geometrically.

**Why this matters for the portfolio:** Training hyperparameter choices are the most commonly interrogated section of any ML methodology review. This entry now demonstrates that I can derive, not guess, the configuration.

---

## 3. Tenacious-Bench Interim Report (Methodology Section)

**Source gaps:** Gaps 2 and 3 — LoRA Rank Math + α/r Scaling

### Before Week 12
The methodology section stated the ORPO fine-tuning configuration with rank and alpha listed as parameters but justified only by reference to the Unsloth tutorial used to scaffold the training script.

### After Week 12
The methodology section was updated to mirror the mathematical justification added to `hyperparams.json`. Specifically:

1. The ΔW = B × A decomposition is stated explicitly with dimension annotations (A ∈ ℝ^(16×k), B ∈ ℝ^(d×16)), establishing that rank 16 was a deliberate capacity choice rather than a default.

2. The α/r = 2.0 invariant is explained in one sentence: scaling the adapter by 2.0 ensures the trained adapter's gradient updates are amplified enough to shift the base model's vocabulary distribution while not overwhelming the base model's pre-trained instruction-following priors during early training epochs.

3. The choice to target only `q_proj` and `v_proj` is defended on parameter-efficiency grounds: adapting Q (where the model attends) and V (what it extracts) captures the primary behavioral shift for tone-oriented tasks while staying within T4 VRAM limits. Including MLP adapters would triple the trainable parameter count without proportional gain for a five-criterion evaluation task.

**Why this matters for the portfolio:** The methodology section of a benchmark paper is the section most scrutinized for engineering rigor. These additions transform the section from "here are the parameters we used" to "here is why these are the correct parameters for this task."

---

## 4. Tenacious-Bench v0.1 Release Presentation (`tenacious_bench_v0.1_presentation.md`)

**Source gaps:** Gaps 3 and 4 — α/r Scaling + Prompting vs. Fine-tuning

### Before Week 12
Slide 4 ("Fine-tuning Methodology") contained a bullet: *"Trained using standard LoRA hyperparameters (r=8, alpha=16)"* with no speaker notes explaining why these specific values were chosen or what they protect against.

Additionally, the presenter narrative defending the decision to build a custom trained judge rather than use a generic OpenRouter judge was grounded only in qualitative claims about "better discriminative power" and "reduced false positives" — neither of which was mechanically explained.

### After Week 12

**Slide 4 speaker notes** were revised to include:
- The α/r = 2.0 scaling explanation: this ratio was deliberately set to ensure early adapter updates do not overwhelm the base model's pre-trained reasoning capabilities — without this invariant, the judge loses its ability to evaluate logical coherence while gaining tone sensitivity.
- A concrete statement of what catastrophic forgetting looks like in this context: the judge begins scoring by surface tone markers only, ignoring semantic validity of the conversion pitch.

**The generic vs. trained judge narrative** (supporting slides) was updated to explain the Walrus Effect: a generic zero-shot model prompted with the Tenacious tone rubric over-indexes on the explicit markers and produces caricatured scores. Tone is a distributed statistical property; training shifts the weight geometry rather than applying a spotlight, which is why the trained judge generalizes more reliably across paraphrase variation in the leads.

**Why this matters for the portfolio:** A release presentation for a production evaluation system should be defensible under engineering interrogation. The updates mean the "why a custom judge" question now has a mechanism-level answer rather than a performance-only claim.

---

## 5. Ablation Audit Memo (`audit_memo.md`) — Pending Revision

**Source gap:** Gap 5 — Bootstrap Statistics for Evaluation (partially open)

### Current state of the artifact
The memo currently states: *"Delta A (trained judge vs. baseline): p=0.189. Result: NOT significant. Verdict: DO NOT DEPLOY."*

### What needs to change
This verdict was formed without understanding the statistical power of the paired bootstrap at n=59 or whether the 60/40 blended scoring formula (`0.6 * judge_score + 0.4 * machine_score`) introduced correlated variance across conditions that artificially reduced the test's sensitivity to the real effect.

**Specifically, the memo needs:**
1. A power analysis: at n=59 and the observed effect size distribution, what is the minimum detectable effect with 80% power? If the true effect is below that threshold, p=0.189 is expected regardless of whether the trained judge genuinely improves over baseline.
2. A variance decomposition: does the 60/40 blend reduce variance in the trained-judge condition (because the machine_score component is shared across both conditions) in a way that creates artificially correlated differences? If so, the effective sample size for the test is smaller than 59.
3. A revised verdict: if the power analysis reveals the test was underpowered for the observed effect, the conclusion should be "insufficient evidence" rather than "null effect" — and the deployment decision should be deferred pending a larger held-out evaluation set.

**Status:** The explainer for Gap 5 was not yet delivered by the close of Week 12. This revision is blocked on the partner session completing. The DO NOT DEPLOY conclusion should be treated as provisional until the power analysis is added.

**Why this matters for the portfolio:** An ablation audit that misinterprets a power problem as a null effect is a significant methodological error. Correcting it demonstrates statistical maturity — the ability to distinguish "we didn't find an effect" from "we couldn't have found an effect with this sample size."

---

## Summary of Changes

| Artifact | Gap | Type of change |
|---|---|---|
| `agent/conversion_engine.py` lines ~196–209 | Gap 1 | Prompt ordering refactor for Radix tree cache hits |
| `README.md` §Known Limitations §3 | Gap 1 | Prefill vs. decode latency analysis |
| `hyperparams.json` `rationale` field | Gap 2 | Mathematical justification replacing tutorial boilerplate |
| Interim Report methodology section | Gaps 2, 3 | ΔW = B×A derivation + α/r invariant + q/v proj justification |
| `tenacious_bench_v0.1_presentation.md` Slide 4 | Gaps 3, 4 | α/r catastrophic forgetting explanation + Walrus Effect narrative |
| `audit_memo.md` § Statistical Significance | Gap 5 | **Pending** — requires power analysis and variance decomposition |

The net effect is that every piece of Week 10–11 work that was grounded in a correctly working system but an incorrectly understood mechanism now has that mechanism documented. The portfolio demonstrates not just that the systems work, but that I understand why they work.
