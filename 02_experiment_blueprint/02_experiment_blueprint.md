# 02 --- Experiment Blueprint

## 1. Experimental question

Compare single-LLM iterative refinement with multi-agent debate for
generating literature-grounded research gaps and testable hypotheses,
measuring both quality and computational cost.

## 2. Shared experimental conditions

The two pipelines share:

-   Research topic: **Status and direction of outlier trimming in
    K-means and similar algorithms**
-   Underlying model in the recorded runs: `gemini-3.5-flash-lite`
-   Primary literature corpus: D01 (17 papers)
-   Paper-level reading procedure and structured paper summaries
-   Maximum refinement/debate limit: 3 rounds
-   Retry limit: 3 attempts for retryable API errors
-   Corpus-only generation and criticism; web grounding is disabled

The runner's current configuration sets `PAUSE_SEC = 10`. The supplied
run metadata records pauses of 3--5 seconds for D01 runs, so the actual
historical run settings should be taken from each run's logs rather than
inferred from the current script alone.

## 3. Single-LLM pipeline

One LLM acts as the same researcher throughout the pipeline.

1.  **Paper analysis:** read each PDF separately and produce a
    structured summary with evidence excerpts and references.
2.  **Research-gap synthesis:** synthesise the paper summaries and
    identify distinct, evidence-supported gaps.
3.  **Gap self-critique:** assess each gap for unsupported claims,
    already-addressed issues, infeasibility, vagueness, and overlap.
4.  **Gap revision:** retain, revise, merge, or reject records based on
    the critique.
5.  **Hypothesis generation:** generate falsifiable hypotheses from the
    finalised gaps.
6.  **Hypothesis self-critique and revision:** independently refine the
    hypotheses.

Each subject can undergo up to three critique--revision rounds. The
process can stop early when no changes are proposed. It does not
introduce separate personas, independent reviewers, or adversarial
debate.

## 4. Multi-agent debate pipeline

The multi-agent pipeline uses topic-adaptive perspectives rather than
fixing disciplinary roles in advance.

1.  **Perspective discovery:** identify a small set of relevant
    perspectives, including their scope, exclusions, ownership
    boundaries, and rationale.
2.  **Perspective-specific generation:** each perspective independently
    proposes research gaps.
3.  **Initial curation and corpus analysis:** a curator deduplicates and
    merges candidates and checks them against the complete set of paper
    summaries.
4.  **Perspective-based critique:** each perspective critiques the
    active records, considering novelty, scientific/theoretical
    soundness, feasibility, significance, and testability as applicable.
5.  **Centralised adjustment:** one adjuster considers the critiques and
    decides whether to retain, revise, merge, or reject each record.
6.  **Hypothesis generation:** perspective-specific generators propose
    hypotheses from the finalised gaps.
7.  **Hypothesis debate and adjustment:** hypotheses undergo the
    corresponding critique and adjustment process.
8.  **Final curation:** a dedicated curator produces the final gap and
    hypothesis collections and records merged or rejected items.

The maximum is three debate rounds per subject. Record IDs are preserved
through revisions and are not renumbered after merges or rejections.

## 5. Execution controls

  Control                 Current `main.py` setting
  ----------------------- --------------------------------
  Mode                    `multi` (editable to `single`)
  Model                   `gemini-3.5-flash-lite`
  Requested temperature   0.2
  Knowledge cutoff        2026
  Maximum rounds          3
  Pause between calls     10 seconds
  Maximum retries         3
  Structured output       Enabled, JSON Schema method

**Temperature note:** the current model parameter allowlist does not
pass `temperature` for `gemini-3.5-flash-lite`. The requested value of
0.2 is therefore not applied to that model. The evaluation metadata
records temperature as `NA`.

## 6. Experimental runs

Fourteen runs are represented in the supplied run metadata:

-   11 runs use D01 and form the primary comparable set: 5 single-agent
    and 6 multi-agent runs.
-   3 runs use D02, D03, or D04 and are excluded from the primary
    comparison.
-   Multiple architecture snapshots are represented, allowing the
    experiment to include changes in pipeline design.

The run IDs, architecture IDs, dataset IDs, token counts, API calls, and
runtimes are documented in the analysis CSVs.

## 7. Outcomes

### Quality

Research gaps and hypotheses are evaluated separately on:

1.  Novelty
2.  Significance
3.  Feasibility
4.  Testability

### Computational cost

-   Input tokens
-   Output tokens
-   Total tokens
-   API calls
-   Failed calls
-   Runtime

## 8. Interpretation

The experiment is a controlled implementation comparison, not a proof
that one orchestration strategy is universally superior. Results must be
interpreted in light of run-to-run variation, architecture changes, the
bounded literature corpus, and the additional cost of multi-agent
coordination.
