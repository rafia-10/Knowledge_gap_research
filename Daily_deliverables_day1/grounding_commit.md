# Grounding Commit

**Artifact Edited:** `agent/conversion_engine.py` and `README.md`

**What changed and why:**
In `README.md §Known Limitations §3`, I updated the latency guidance. Instead of just suggesting dropping the model, I added an analytical note differentiating prefill vs. decode latency, documenting that "while prompt restructuring yields high prefix cache hit rates for the static tone markers, the 46s p50 is gated by generation length, necessitating smaller non-thinking models for batch scale." 
Additionally, I refactored the prompt construction in `conversion_engine.py` (lines ~196-209) to physically place all static tone constraints and rules at the absolute start of the `messages` array, pushing the dynamic JSON serialization of the user's lead strictly to the tail end. This guarantees the first ~80 tokens successfully match the vLLM Radix tree across all sequential calls.

