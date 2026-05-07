# Sources

The explainer and thread I wrote for Ephrata were grounded in the following canonical sources and patterns:

**Canonical Sources:**
1. **LoRA: Low-Rank Adaptation of Large Language Models** (Hu et al., 2021) - Used to verify the exact mathematical mechanism of weight updates ($W = W_0 + \Delta W$) versus transient state generation in attention processing.
2. **LIMA: Less Is More for Alignment** (Zhou et al., 2023) - Used as the theoretical backing for how fine-tuning shifts style/alignment distributions structurally in ways that zero-shot or few-shot prompting often fails to replicate smoothly.

**Tools & Patterns:**
- I ran a quick comparative script using the `transformers` library on a small base model (Qwen 1.5B), printing the actual next-token logit distributions with and without a style-transfer prompt to observe "The Walrus Effect" experimentally, confirming that the prompt merely routes attention to clichés rather than rewriting stylistic priors.
