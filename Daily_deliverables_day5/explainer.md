# explainer.md

# Does My Benchmark Accidentally Favor PageIndex?

## The Question

My peer asked an important evaluation-design question:

> If the benchmark topics and relevance labels are derived from the same PageIndex structure used by the retrieval system, does that unfairly bias the measured improvement over baseline vector search?

Short answer: yes, it potentially can.

The concern is not that the benchmark is fake or useless. The concern is that the benchmark may partially reward the system for matching the same structure that generated the evaluation labels in the first place.

This is a classic evaluation leakage problem.

---

## What Is Happening Mechanically?

In the current setup, the evaluation topics and relevance judgments are derived from PageIndex sections.

That means:
- the retrieval system organizes information according to PageIndex structure
- and the benchmark also defines “correctness” using that same structure

This creates coupling between:
1. the retrieval representation
2. the evaluation criteria

When those two are derived from the same source, the benchmark can become structurally favorable to the indexed system.

The problem is subtle.

Even if the user-style demo queries are realistic, the formal `precision@3` metric may still reward PageIndex because the “ground truth” itself was shaped by PageIndex assumptions.

In other words:

> the benchmark may be testing alignment with PageIndex organization rather than true retrieval usefulness.

---

## Why Vector Search May Look Worse Than It Actually Is

Traditional vector retrieval works through semantic similarity.

It does not necessarily care about:
- document hierarchy
- section boundaries
- explicit organizational structure

Meanwhile, PageIndex retrieval explicitly encodes those structures.

So if relevance labels are defined around PageIndex sections, then:
- PageIndex retrieval naturally aligns with the benchmark
- vector search may retrieve semantically correct passages that are scored as “less relevant” simply because they do not map cleanly onto the predefined section structure

This creates evaluation skew.

The benchmark begins rewarding:
- structural agreement

instead of purely:
- user usefulness

That distinction matters a lot.

---

## Why This Matters for FDE Systems

Forward-Deployed Engineers often build retrieval systems for:
- enterprise search
- internal copilots
- agent memory systems
- RAG pipelines

In all these systems, benchmark validity matters more than benchmark score.

A biased benchmark can create false confidence.

You may think:

> “PageIndex is objectively better.”

But the benchmark might actually mean:

> “PageIndex is better at matching labels derived from PageIndex.”

Those are not the same claim.

This is exactly why evaluation design is one of the hardest parts of applied AI systems.

---

## Would a Human-Labeled Held-Out Set Be Fairer?

Yes — and probably necessary.

A held-out human-labeled query set is the minimum stronger validation because it separates:
- the retrieval mechanism
- the query creation process
- and the relevance labels

In a cleaner setup:
- humans independently write realistic queries
- humans independently label relevant results
- without relying on PageIndex structure

That reduces structural leakage.

Now the retrieval systems compete on:
- usefulness
- relevance
- retrieval quality

instead of compatibility with the benchmark construction process.

---

## Important Nuance: Human Labels Are Not Perfect Either

Human evaluation also introduces noise:
- annotator disagreement
- inconsistent relevance judgments
- varying domain understanding

But despite this, independent human labels are still far more reliable than labels derived from the same retrieval structure being evaluated.

The key principle is independence.

A benchmark is strongest when:
- retrieval generation
- query creation
- and relevance labeling

are separated as much as possible.

---

## What the Current Benchmark Still Tells Us

This does not mean the current benchmark is invalid.

It still measures something useful:
- whether PageIndex improves retrieval under a structure-aware evaluation setting

That is still valuable engineering evidence.

The danger is overclaiming.

The benchmark supports:

> “PageIndex performs well under this evaluation setup.”

It does not fully support:

> “PageIndex universally improves retrieval quality.”

That stronger claim requires held-out human evaluation.

---

## Final Takeaway

The core issue is evaluation coupling.

When the retrieval system and benchmark labels originate from the same structure, measured gains can become partially self-reinforcing.

A held-out human-labeled query set is therefore the minimum fair validation because it introduces independence between:
- the retrieval architecture
- and the evaluation criteria

Without that separation, improvements in `precision@3` may reflect benchmark alignment as much as real-world retrieval quality.