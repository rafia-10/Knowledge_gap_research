# Thread

**Tweet 1:**
Are you paying 20x for the exact same LLM system prompt? 💸

A colleague noticed terrible latency when running 20 leads sequentially through an 80B model using the identical 60-token system prompt. Is there a way to avoid redundant computation? Yes: Prefix Caching. 🧵👇

**Tweet 2:**
First, what is the KV Cache?
Instead of recalculating attention over the entire prompt for every single generated token (which is O(N^2)), LLMs store the Key/Value matrices of past tokens in GPU memory. For an 80B model (like Qwen3), one token costs ~328 KB of VRAM.

**Tweet 3:**
Standard KV caching dies when the request finishes. 
*Prefix caching* (via Radix Trees in vLLM/SGLang) keeps the Key/Values in GPU memory across requests. If a new request shares the exact same starting tokens, it skips the expensive prefill phase! ⚡

**Tweet 4:**
But there's an exact matching rule 🛑.
The cache matches sequentially. If you put a user's name at the very beginning of the prompt, the cache diverges at token 3. The entire rest of the prompt must be recomputed. 

Always format: `[Static System Prompt] -> [Variable Data]`

**Tweet 5:**
Does OpenRouter do this automatically? 
Yes and No. OpenRouter proxy-routes to providers (Fireworks, vLLM servers) that often run prefix caching globally to save *themselves* compute. If supported, they return `cached_tokens` counts in the usage response object.

**Tweet 6:**
The reality check: Prefix caching eliminates *Prefill* time (input processing). But for heavy reasoning models, 99% of your latency is *Decode* time (output generation). Caching is magical for massive inbound RAG contexts, but won't solve heavy output latency bottlenecks.

Read my full token math breakdown: [Link to Blog Post]
