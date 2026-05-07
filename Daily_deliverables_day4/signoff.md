# Sign-off

**Gap Status:** Closed

**What I understand now:**
Before this research loop, I simply copy-pasted $r=8, \alpha=16$ into my Tenacious-Bench training scripts because the tutorial said it worked. Thanks to Ephrata's explainer, I now understand the mathematical mechanics. The low-rank matrices $B$ and $A$ are multiplied by $\frac{\alpha}{r}$ before adding their output to the base layer. If $r$ increases while $\alpha$ remains fixed, the effective learning magnitude drops. By holding the $\alpha/r$ ratio steady, we decouple the learning rate from the rank dimension. Furthermore, I now grasp that if $\alpha$ is too high relative to $r$, the adapter’s early gradient steps dominate the base model's outputs, leading to catastrophic forgetting of the base instruction-following capabilities. I can confidently defend our hyperparameter choices now.
