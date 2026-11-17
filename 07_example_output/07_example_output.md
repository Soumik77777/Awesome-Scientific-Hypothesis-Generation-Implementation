# 07 --- Example Output

## 1. Purpose

This document records representative outputs and the corresponding
prompt snapshots from the supplied experiment runs. The examples
illustrate the output format and differences between the two pipeline
architectures; they are not intended as a scientific validation of the
proposed ideas.

## 2. Single-agent example

**Run:** `1794725368`

**Artifacts supplied:**

-   `run_1794725368_prompt_snapshot.py`
-   `run_1794725368_identified_research_gaps.txt`
-   `run_1794725368_proposed_hypothesis.txt`

The run produced 8 research gaps and 9 hypotheses.

### Example research gap

> G001: Lack of deterministic, non-exhaustive algorithms for identifying
> optimal trimming regions without enforcing strict spherical symmetry
> and equal dispersion radii across clusters.

The output includes supporting paper IDs and section references.

### Example hypothesis

> H001: Formulating optimal trimming regions as non-spherical geometric
> unions of dispersion-parameterized scatter matrices rather than
> equal-radius balls enables robust recovery of heterogeneous, nearby,
> or elongated clusters.

The hypothesis output also records a rationale, a testable prediction,
and supporting paper references.

## 3. Multi-agent example

**Run:** `1794730841`

**Artifacts supplied:**

-   `run_1794730841_prompt_snapshot.py`
-   `run_1794730841_identified_research_gaps.txt`
-   `run_1794730841_proposed_hypothesis.txt`

The run produced 9 research gaps and 8 hypotheses.

### Example research gap

> G001: Universal Breakdown Bounds and Worst-Case Robustness Limits in
> Arbitrary versus Well-Structured Metric Spaces.

The output links the gap to supporting papers and specific sections.

### Example hypothesis

> H001: Alternative penalty functions, non-convex proximal maps, or
> modified objective formulations cannot fundamentally elevate universal
> breakdown bounds beyond O(1/n) on arbitrary, unconstrained datasets
> without requiring prior geometric restrictions.

The hypothesis output includes a rationale, a testable prediction, and
paper references.

## 4. Prompt snapshots

Each run stores a snapshot of the complete selected agent module,
including prompts, helper functions, and orchestration logic. The
supplied snapshots are therefore the authoritative record of the
architecture used in the corresponding run.

The two examples above illustrate different pipeline architectures. The
single-agent snapshot contains the single-researcher synthesis and
self-critique/revision workflow. The multi-agent snapshot contains
perspective discovery, perspective-specific generation and critique,
centralised adjustment, and final curation.

## 5. Output structure

The current runner writes:

-   `identified_research_gaps.txt`
-   `proposed_hypothesis.txt`
-   `prompt_snapshot.py`
-   `api_usage.json`
-   `inference_history.json`
-   `inference_history.txt`
-   `debug_*.jsonl`

The text files are readable summaries. The inference-history JSON
retains structured records and the sequence of events; the prompt
snapshot preserves the implementation used for that run.

## 6. Interpretation caution

The two examples come from different run IDs and architecture snapshots.
They should be used to demonstrate output structure, not as a controlled
item-by-item comparison. The quantitative comparison belongs in
`06_results.md`.
