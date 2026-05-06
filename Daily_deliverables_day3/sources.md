# Sources

**Canonical Papers/Documentation:**
1. *LoRA: Low-Rank Adaptation of Large Language Models* (Hu et al., 2021) - The foundational paper defining the $B \times A$ decomposition and the $\frac{\alpha}{r}$ scaling invariant.
2. *ORPO: Monolithic Preference Optimization without Reference Model* (Hong et al., 2024) - Justifies the beta learning configurations used in conjunction with the LoRA adapter.

**Tool / Pattern Used:**
- Inspecting the `train_judge.py` PEFT configuration, and manually writing out the shape sizes of the Qwen2.5 projection layers to calculate the exact parameter reduction ratio of 98%.
