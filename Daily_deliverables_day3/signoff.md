# Asker Sign-off

**Status: Closed**

This explainer completely closed the gap. I used to think of `lora_alpha` as a learning rate multiplier and `rank` as just a memory switch. Seeing the actual equation $h = Wx + \frac{\alpha}{r} \Delta Wx$ makes it undeniably clear why setting $\alpha = 2r$ creates a stable, invariant scaling factor of 2.0. This mathematical grounding means I no longer have to blindly guess hyperparameters; I can intentionally design the informational bottleneck based on how many semantic concepts (like the 5 Tenacious tone markers) the adapter needs to memorize.
