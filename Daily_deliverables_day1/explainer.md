# The Statistics of Sycophancy: Why Generic LLMs Make Terrible Judges

*An explainer for Forward-Deployed Engineers building LLM-as-a-judge evaluation pipelines.*

When building the `tau2` simulation framework and scoring tasks for `Tenacious-Bench`, we hit a wall. We were using OpenRouter’s generic `deepseek-chat-v3` as a zero-shot judge to grade model completions against strict tone markers (Direct, Grounded, Honest, Professional, Non-condescending). But the judge kept misbehaving. It systematically rewarded long, sycophantic, flowery responses—even when they violated our "Direct" and "Non-condescending" markers—while penalizing concise, ruthlessly accurate completions that hit the rubrics perfectly. 

My partner asked a critical question: *How do position bias (ordering of the prompt context) and length bias manifest mathematically at the token-probability level in generic LLM-as-a-judge models, and how does this statistical skew physically justify our architectural decision to build a custom trained LoRA judge?*

We often wave our hands and say "generic judges lack discriminative power" or "they hallucinate." But as Forward-Deployed Engineers, we cannot paper over the mechanics. If we are using LLMs to score other LLMs, we must understand the math of the judge's blind spots.

## The Load-Bearing Mechanism: Length Bias and Token Probability Chains

Pre-trained LLMs, at their core, are next-token predictors optimized to minimize cross-entropy loss over massive internet datasets. When acting as a judge, an LLM generates a chain of tokens that culminate in a final score or pass/fail judgment.

The manifestation of length bias is not a psychological preference; it is a statistical artifact of autoregressive generation. 

Consider how an LLM scores a completion. The joint probability of a judgment sequence $Y = (y_1, y_2, \ldots, y_n)$ given the context $X$ (the rubric and the completion) is the product of conditional probabilities:

$$ P(Y \mid X) = \prod_{i=1}^n P(y_i \mid y_{<i}, X) $$

In a typical zero-shot prompt, the judge outputs a reasoning trace ("The response addresses the customer's query by...") before outputting the final score. Because human-written training data on the internet heavily correlates "long, detailed responses" with "high-quality, polite answers," the attention mechanism naturally assigns higher logits to positive evaluative tokens when attending to long completions.

If you profile the logits of `deepseek-chat-v3` scoring a 500-word sycophantic email versus a 50-word direct email, the cross-attention weights strongly fire on the volume of tokens in the 500-word email. The model mathematically drifted toward generating tokens like "comprehensive" and "polite", leading to a final pass. For the 50-word email, the scarcity of tokens depressed the attention activation patterns associated with "goodness," pushing the model to generate words like "abrupt," violating our tone markers artificially.

## The Load-Bearing Mechanism: Position Bias (Lost in the Middle)

Position bias is the second pillar of evaluator failure. When we feed an LLM a prompt structured as `[Rubric] -> [Prompt] -> [Model Completion]`, the distance between the rubric constraints and the final judgment token can be thousands of tokens.

LLMs exhibit a "U-shaped" attention curve. They perfectly attend to the very beginning of the context window (due to attention sinks—the first few tokens act as key-value anchors) and the very end of the context window (due to recency bias in relative positional embeddings like RoPE). 

If a completion is long, the [Rubric] gets pushed out of the high-attention zone into the middle of the context. The judge "forgets" the strict tone markers ("Must be direct", "Must not be condescending") because the Query and Key matrices ($Q$ and $K$) for those tokens yield low dot-products compared to the recent tokens of the flowery completion. We proved this by flipping the context order to `[Model Completion] -> [Prompt] -> [Rubric]`. Suddenly, the model became hypersensitive to the rubric and heavily penalized the same flowery completion it just approved. This asymmetry mathematically invalidates zero-shot `pass^k` metrics because the score is a function of string ordering, not string quality.

## The Solution: Why LoRA Adapters Fix the Math

To solve this for `Tenacious-Bench`, we didn't tweak the prompt. We trained a custom LoRA (Low-Rank Adaptation) adapter. But *why* does this fix the bias?

When we train a LoRA adapter, we freeze the base model weights $W \in \mathbb{R}^{d \times k}$ and inject trainable rank decomposition matrices $A \in \mathbb{R}^{r \times k}$ and $B \in \mathbb{R}^{d \times r}$. During the backward pass, the LoRA adapter is trained strictly on pairs of `(Completion, Binary Score)`.

By doing this, we explicitly reshape the loss landscape. We break the internet-learned heuristic that *length = quality*. The gradient updates to the $A$ and $B$ matrices forcibly reweigh the attention mechanism to attend directly to structural constraint violations (like condescending tone markers) regardless of where they appear in the text, and regardless of how long the text is. It zeroes out the sycophancy bias because the low-rank updates act as a highly specialized filter that overrides the base model's default attention distribution. 

If you want an objective evaluator, you cannot rely on generic next-token probabilities. You must physically warp the attention weights to respect your specific constraints.

## Connect the Dots

**1. Attention Sinks:** The concept that the first few tokens in a context window mathematically anchor the KV cache, absorbing massive attention scores. If your rubric isn't in those first tokens, its influence degrades rapidly.
**2. RoPE (Rotary Position Embedding):** The mechanism that injects relative positional information into the attention calculation. RoPE heavily biases dot products toward tokens that are positionally close to the current generation step, causing the "recency bias" layer of position bias.
**3. SimPO / DPO:** Post-training techniques designed to align models with human preferences. Often, these very techniques are what instill the length and sycophancy bias in base models like DeepSeek in the first place, optimizing for what human raters *perceive* as polite rather than what is strictly required by B2B sales markers. 

To build reliable data engines, we have to look past the API abstraction. When you understand the token probabilities, the need for custom LoRA judges isn't just an optimization—it is an architectural necessity.
