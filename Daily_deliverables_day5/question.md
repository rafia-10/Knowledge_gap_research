# Day 5
# Final Sharpened Question

**Topic:** Evaluation and Statistics

**Diagnostic Gap:**
In my Tenacious-Bench v0.1 ablation audit (`ablations/run_ablations.py`), I use a paired bootstrap test (`paired_bootstrap`) to compare three conditions — baseline scoring_evaluator, trained LoRA judge, and prompt-only Qwen judge — and report Delta A (trained vs. baseline) as p=0.189, which is not statistically significant. I recorded this as a "DO NOT DEPLOY" signal in my audit memo. But I do not actually understand *why* the paired bootstrap is the correct statistical test here rather than a paired t-test or a Wilcoxon signed-rank test, nor do I understand what the specific p=0.189 result means mathematically about the null distribution of score differences across the 59 held-out tasks. Specifically: how does the bootstrap resample procedure construct its null distribution from the paired score differences, what sample size (n=59) implies about statistical power for the effect sizes we observed, and how does the 60/40 blended scoring formula (`0.6 * judge_score + 0.4 * machine_score`) in `condition_trained_judge` introduce a systematic variance reduction that may be artificially deflating our test's sensitivity?

**Connection to Grounded Work:**
Knowing this would let me revise the "Statistical Significance" section of `audit_memo.md` and the ablation results table in my interim report. Currently I state that "Delta A is not significant (p=0.189)" as a final verdict, but I am papering over a gap: I do not know whether p=0.189 reflects a genuine null effect, or whether the blended scoring formula introduced correlated variance between conditions that reduced power below the threshold needed to detect the real effect size at n=59. If the 60/40 blend is the culprit, the DO NOT DEPLOY conclusion may be based on a misconfigured evaluation, not a real absence of signal from the trained judge.

**Generalizability:**
Any engineer deploying an LLM-as-judge evaluation pipeline (such as `tau2-bench` or a custom rubric evaluator) needs to understand how score-blending formulas, non-normal score distributions, and small held-out partition sizes interact with bootstrap test power — otherwise they will either ship a model that genuinely does not improve or incorrectly flag a good model as neutral.

**Resolvability:**
A colleague can resolve this by producing a 600–1,000 word explainer that: (1) derives the paired bootstrap null distribution construction step by step, (2) calculates the minimum detectable effect size at n=59 with 80% power under bootstrap assumptions, and (3) shows concretely how the 60/40 blending formula reduces the variance of the per-task score differences and whether that reduction is enough to explain the observed p=0.189.
