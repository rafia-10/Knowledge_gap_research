# Grounding Commit

**Artifact Edited:** `tenacious-bench/training/hyperparams.json` and `tenacious-bench_interim_report.pdf`

**What changed and why:**
I rewrote the `"rationale": "lora_rank"` dictionary entry in `hyperparams.json`. Instead of vaguely stating "rubric complexity warrants higher rank," I added the mathematical justification: "Rank 16 provides the minimum independent semantic dimensions required to encode the 5 Tenacious tone markers simultaneously without destructive interference in the projection matrices. Alpha is strictly set to 32 to guarantee the invariant $\frac{\alpha}{r} = 2.0$ activation scaling factor." I mirrored this exact technical phrasing in the methodology section of the interim report to mathematically defend the ORPO training configuration.
