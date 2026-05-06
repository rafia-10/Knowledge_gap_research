# Day 2
# Final Sharpened Question

**Diagnostic Gap:**
When training our Tenacious-Bench custom evaluator with ORPO on `Qwen2.5-1.5B-Instruct`, we targeted only `q_proj` and `v_proj` with a LoRA rank ($r$) of 16 and an alpha ($\alpha$) of 32. 
How do the low-rank matrices $A$ and $B$ actually bottleneck the gradient updates to the attention projections, what specifically changes in the parameter dimensions and representational capacity when increasing $r$ from 8 to 16, and why is $\alpha$ hardcoded to exactly $2r$ in the weight scaling math?

**Connection to Grounded Work:**
Knowing this would allow me to revise the `rationale: lora_rank` section in my `tenacious-bench/training/hyperparams.json` artifact, and the corresponding paragraph in the `tenacious-bench_interim_report.pdf`. Currently, I claim that "Rank 8 was considered but judge rubric complexity warrants higher rank," but I am treating rank and alpha purely as numbers I copied from an unsloth tutorial. I could not mathematically defend what information capacity a rank 16 bottleneck actually provides over a rank 8 bottleneck, nor why the scaling invariant requires $\alpha=32$.

**Generalizability:**
Every Forward-Deployed Engineer using PEFT (Parameter-Efficient Fine-Tuning) to adapt models for specific reasoning or evaluation tasks must configure rank and alpha. Blindly copying $r=16, \alpha=32$ without understanding the $A \times B$ decomposition leads to VRAM wastage or catastrophic underfitting when the rank becomes a rigid bottleneck.

**Resolvability:**
A colleague can resolve this by producing a 600-1,000 word explainer that writes out the actual matrix multiplication $\Delta W = B \times A$, visualizes how rank determines the inner dimension, and algebraically shows how the $\Delta W$ matrix is scaled by $\frac{\alpha}{r}$ before being added to the frozen base weights during inference.
