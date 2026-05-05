# Knowledge Gap Research

This directory contains the Week 12 knowledge gap research deliverables for the **Tenacious Conversion Engine** project. 

The goal of this week is research and pedagogy: shifting from building systems to auditing the systems already built, finding the places where the engineer does not actually know how something works, closing those gaps through deep pair research, and producing artifacts that teach the concept to the community.

## Structure

Each day produces a set of deliverables (housed in `Daily_deliverables_day[N]`), which include:
- `question.md`: The sharpened diagnostic question, grounded in existing portfolio work.
- `morning_call_summary.md`: Summary of the questioning phase.
- `explainer.md`: A 600-1000 word blog post closing the gap.
- `thread.md`: A 4-6 tweet synthesis of the explainer.
- `evening_call_summary.md`: Feedback and revision notes.
- `signoff.md`: Confirmation that the gap was closed.
- `grounding_commit.md`: Pointer to the concrete edit made to existing Week 10/11 artifacts based on the research.
- `sources.md`: The canonical papers and tools used.

Additionally, the root folder holds cumulative artifacts:
- `synthesis.md`: Week-end synthesis of the 10 gaps closed.
- `canonical_list.md`: Annotated contribution to the cohort canon.
- `portfolio_update.md`: Summary of how the grounding commits improve the portfolio.

## Tech Stack Context
All questions are formulated rigorously from the practical systems implemented for the Conversion Engine, specifically targeting:
* **LLM Inference Mechanics:** KV caching, prefix caching across sequential requests, memory footprint, OpenRouter API proxy behavior, and decoding vs prefill latency.
* **Agent Architecture:** Converting leads using system prompts tied to the specific pipeline, and assessing structural formatting for downstream pipeline tools.
