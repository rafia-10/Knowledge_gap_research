# Unpacking the Black Box: KV Cache, Prefix Caching, and Inference Math

*An explainer for Forward-Deployed Engineers building LLM automation pipelines.*

When my partner Martha was benchmarking 20 leads sequentially through the Tenacious Conversion Engine, her p50 latency was 46.6 seconds for an identical 60-token system prompt sent to `qwen/qwen3-next-80b-a3b-thinking`. Her question cuts to the core of production inference: *What is the KV cache storing, how does prefix caching work, what invalidates it, and is OpenRouter doing this for me under the hood?*

As Forward-Deployed Engineers, we cannot treat inference as a monolithic HTTP request. If we do not understand the shape of the memory, we waste time, compute, and money. Knowing how the KV cache works is the difference between blindly over-provisioning servers and designing prompts that effortlessly slice latency in half.

## 1. What the KV Cache Actually Stores

To understand prefix caching, we must first understand why the KV cache exists.

In a dense Transformer model, generating each subsequent token ("autoregressive decoding") requires the model to compute attention scores between the new token and *every single token that preceded it*. The attention mechanism computes Query (Q), Key (K), and Value (V) projection vectors for every token at every layer.

If we did not use a KV cache, computing the 100th token would require calculating the K and V vectors for all 99 previous tokens from scratch. This would make text generation $O(N^2)$ in time. To solve this, inference engines store the intermediate K and V projection tensors in GPU memory—the **KV cache**.

When generating the 100th token, the model only computes the $Q$, $K$, and $V$ for token 100. It then compares its $Q$ against the $K$ vectors stored in the fast KV cache, avoiding recomputation.

## 2. Quantifying the Memory Footprint (Qwen3-80B)

How expensive is it to store this cache? We can calculate it mathematically. The formula for the size of a KV cache entry per token, per batch, in bytes is:

$$ 2 \times n_{\text{layers}} \times n_{\text{kv\_heads}} \times d_{\text{head}} \times \text{bytes\_per\_param} $$

Let's assume an 80B architecture similar to Qwen/Llama with Grouped-Query Attention (GQA). Approximating the dimensions:
- Layers ($n_{\text{layers}}$): 80
- KV Heads ($n_{\text{kv\_heads}}$): 8
- Head Dimension ($d_{\text{head}}$): 128
- Precision ($FP16/BF16$): 2 bytes

Plugging this in: $2 \times 80 \times 8 \times 128 \times 2 = 327,680$ bytes (roughly **328 KB context per token**).

So, for Martha's 60-token system prompt, the persistent memory cost in the KV cache is $60 \times 328 \text{ KB} \approx 19.6 \text{ MB}$. In the grand scheme of an 80B model that requires 160 GB of VRAM just to load its weights, 20 MB is absolutely trivial. 

## 3. Prefix Caching and Invalidation Mechanics

Standard KV caching only works *within a single request*. Once the request finishes, the GPU discards the cache to free up VRAM.

**Prefix caching** (often implemented via RadixAttention in vLLM or SGLang) fundamentally changes this. The inference engine keeps the KV caches of completed requests resident in GPU memory, structured natively as a Radix Tree mapping token IDs to KV tensors.

When a new API call arrives, the engine compares the token IDs sequentially starting from the beginning.
- If tokens `1..60` match exactly with a cached sequence, the engine simply maps those GPU memory blocks to the new request. This makes the **Prefill Phase** for those 60 tokens instantaneous.
- **Invalidation/Cache Miss:** The matching rule is strictly exact prefix matching. The moment a single token diverges (e.g., if you inject the user's name at token 12), *the entire remainder of the cache is invalidated for that request*. The model must compute full attention for token 12 and all subsequent tokens. 

This strict rule is why **prompt restructuring is critical**. If you map:
`[Variable User Data] -> [Static System Prompt]`
The Radix tree diverges immediately. You get 0% prefix cache hits.
If you map:
`[Static System Prompt] -> [Variable User Data]`
The first 60 tokens match the Radix tree perfectly.

## 4. Production Reality: OpenRouter and the API Layer

So, does this happen automatically when you call OpenRouter?

The answer is: **It depends on the backend provider being routed to.** OpenRouter is a proxy. When you request an OpenRouter model, OpenRouter is routing your request to a data center running an inference engine (like vLLM, Fireworks, or Anthropic).

- **Anthropic natively supports prompt caching,** but you must explicitly pass `cache_control` headers. OpenRouter passes these through.
- **DeepSeek and Qwen** models running on specialized high-throughput providers (like DeepSeek's native API or Fireworks) use SGLang or vLLM heavily. These providers generally have prefix caching turned **ON** globally for all users because it drastically reduces their compute costs. 
- OpenRouter returns `prompt_tokens_details: { cached_tokens: X }` in their API response object if the downstream provider supports reporting cache hits.

For Martha's 20-lead benchmark, assuming she structures her prompt so the 60 static tokens are first, the downstream provider will likely hit the prefix cache for the identical tokens across the requests. However, because she uses an 80B "thinking" model, the 46 seconds of latency is massively dominated by the **Decode Phase** (the generation of the reasoning trace), not the prefill phase. Eliminating 60 tokens of prefill (saving ~50-100 milliseconds) will not meaningfully dent a 46-second generation. 

Caching solves prefill boundaries, but it does not compress the generation time of reasoning tokens. For scale, switching the model—as she initially hypothesized—remains the architecturally correct choice to impact p50 latency.
