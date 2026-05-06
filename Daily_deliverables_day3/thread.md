# Thread

**Tweet 1:**
Stop guessing your LoRA hyperparameters! 🛑

While training the `Tenacious-Bench` AI judge on Qwen2.5, we set rank $r=16$ and $\alpha=32$. But what do those numbers actually mean mathematically? And why can a tiny adapter fundamentally alter an LLM's behavior? 🧵👇

**Tweet 2:**
LoRA (Low-Rank Adaptation) works by freezing a massive base weight matrix (e.g., $1536 \times 1536$) and training two tiny decomposition matrices $A$ and $B$. Instead of updating 2.3M parameters, you update just 49k. 

The rank ($r$) is the narrow "bottleneck" dimension connecting $A$ and $B$.

**Tweet 3:**
What happens if you increase rank from 8 to 16? 
Rank defines the number of linearly independent semantic directions the adapter can learn. We moved to $r=16$ because evaluating 5 complex B2B tone markers simultaneously required more geometric capacity than 8 vectors could hold.

**Tweet 4:**
What about Alpha ($\alpha$)? 
The adapter output is scaled by $(\alpha / r)$ before being added to the base network. Setting $\alpha = 2r$ makes the scaling factor 2.0—a proven sweet spot for task-adaptation. It ensures the adapter "speaks loudly" over the base model.

**Tweet 5:**
By defining $\alpha$ as a ratio of $r$, the scaling factor remains constant even if you change the rank later. This means you don't have to throw away and retune your learning rate when experimenting with wider adapters! 🧠

**Tweet 6:**
Why target only `q_proj` and `v_proj`? 
Adjusting the Query (where the model looks) and Value (what it extracts) is the most VRAM-efficient way to change behavior. Changing everything risks an OOM error on a T4 GPU. 

Learn the linear algebra behind the code: [Link to Blog]
