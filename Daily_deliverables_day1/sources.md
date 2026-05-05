# Sources

**Canonical Papers/Documentation:**
1. *Efficient Memory Management for Large Language Model Serving with PagedAttention* (Kwon et al., SOSP 2023) - The foundational vLLM paper explaining how KV Cache tensors are chunked, stored, and managed in paged VRAM across requests.
2. *SGLang: Efficient Execution of Structured Language Model Programs* (Zheng et al., 2024) - Documentation on RadixAttention prefix caching and exact-match token invalidation behavior.

**Tool / Pattern Used:**
- Modifying the array order in the API request JSON passed to OpenRouter, isolating `system` role text at index 0 and observing the structural difference it makes for downstream Radix tree matching. Checked the OpenRouter API documentation for `prompt_tokens_details` to verify reporting.
