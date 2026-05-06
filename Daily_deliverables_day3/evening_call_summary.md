# Evening Call Summary

In the evening call, I walked through the $\Delta W = B \times A$ matrix equation. My partner noted that the analogy of geometric "width" for the 5 tone markers perfectly landed the concept of structural rank capacity. She requested one modification: explicitly clarifying *why* we targeted `q_proj` and `v_proj` instead of all linear layers. I added a section to the explainer detailing the parameter-efficiency tradeoff of excluding the MLP layers to prevent Out of Memory (OOM) errors on the T4 GPU constraint.
