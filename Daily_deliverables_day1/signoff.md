# Asker Sign-off

**Status: Closed**

I finally understand the distinction between the KV cache (intra-request) and prefix caching (inter-request). Knowing that the Radix tree invalidates the moment a single token diverges completely changes how I will construct standard templates in the Tenacious Conversion Engine. More importantly, understanding that prefix caching only saves *prefill* latency proved my original hypothesis right but for the opposite reason: restructuring the prompt will save compute, but it will not meaningfully dent the 46.6s latency bottleneck, which is strictly governed by the *decode* phase of the thinking model.

