# Question

**Topic:** Training and post-training mechanics

**My Question:**
In my Tenacious-Bench v0.1 release presentation, I rely on a LoRA evaluator that I trained to act as our domain-specific proxy judge. I state that we trained a low-rank adapter on the base LLM, using a rank ($r$) of 8 and an $\alpha$ of 16. What actually happens to the gradient updates and the learned weight matrices when $\alpha$ is scaled relative to $r$? Specifically, how does this $\alpha/r$ scaling factor control the magnitude of the adapter's contribution to the forward pass, and why does getting this ratio wrong cause the model to either ignore the fine-tuning data entirely or catastrophically forget its base instruction-following capability when evaluating Tenacious tone markers?

**Grounding to my work:**
Knowing this would let me revise the fine-tuning methodology slide in my Tenacious-Bench release presentation. Currently, I treat the adapter hyperparameters as boilerplate numbers copied from a tutorial. Closing this gap would enable me to explain specifically why our LoRA-trained judge maintains its base reasoning capabilities while successfully adopting the Tenacious tone, defending the $r=8, \alpha=16$ choice to an engineering team.
