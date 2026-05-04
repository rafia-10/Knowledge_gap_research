# Day 1
# Final Sharpened Question 

**Diagnostic Gap:**
How do position bias (ordering of the prompt context) and length bias manifest mathematically at the token-probability level in generic LLM-as-a-judge models like `deepseek/deepseek-chat-v3` (OpenRouter), and how does this statistical skew precisely justify the shift to building a custom trained LoRA judge for the Tenacious-Bench ablation audit?

**Connection to Grounded Work:**
Knowing this would allow me to confidently revise the 'Generic vs. Trained Judges' section in my **Tenacious-Bench v0.1 Interim Report** and my **Evaluation Pipeline Presentation**. Currently, I claim that our custom trained judge has "better discriminative power" and "reduces false positives." However, I use these phrases to paper over the truth: I do not actually understand the underlying token-level mechanics of why our generic OpenRouter implementation (`agent/llm.py`) systematically fails to penalize longer, sycophantic responses while misclassifying shorter, more direct texts that successfully hit our production tone markers.

**Generalizability:**
Any Forward-Deployed Engineer building an LLM evaluator (like the `tau2` framework) needs to understand how zero-shot position and length bias structurally invalidates their evaluation metrics and why a generic model cannot reliably act as an objective judge on structural constraints.

**Resolvability:**
A colleague can resolve this by producing a 600-1,000 word explainer that traces the log-probabilities for a flipped-context evaluation, isolating the position bias mechanism, and demonstrating the attention distribution that causes pre-trained models to intrinsically reward verbosity.
