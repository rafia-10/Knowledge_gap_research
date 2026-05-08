# thread.md

# Thread: Can Your Benchmark Accidentally Favor Your Retrieval System?

1/  
A retrieval benchmark can accidentally favor the system it’s trying to evaluate.

That happens when the benchmark labels are derived from the same structure the retrieval system uses internally.

This is a subtle but important evaluation problem in RAG systems.

2/  
In our case, PageIndex retrieval organizes information using structured document sections.

But the benchmark relevance labels were also derived from those same PageIndex sections.

That creates “evaluation coupling.”

3/  
The issue is not that the benchmark is fake.

The issue is that the benchmark may partially reward:
- alignment with PageIndex structure

instead of purely measuring:
- actual retrieval usefulness

Those are not the same thing.

4/  
This can make baseline vector search look worse than it actually is.

Vector retrieval focuses on semantic similarity, not document hierarchy or section boundaries.

So semantically correct retrievals may still score lower under a structure-aware benchmark.

5/  
A stronger validation approach is a held-out human-labeled query set.

Why?

Because independent humans create:
- the queries
- and the relevance labels

without relying on the retrieval structure itself.

That breaks the evaluation coupling.

6/  
Big takeaway:

A benchmark can measure real improvement and still contain structural bias at the same time.

Good evaluation design is not just about metrics.

It’s about independence between:
- the system,
- the labels,
- and the benchmark itself.