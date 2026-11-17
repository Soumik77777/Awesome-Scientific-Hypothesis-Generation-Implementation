# 04 --- Evaluation

## 1. Purpose

Evaluate the scientific quality of every generated research gap and
hypothesis and compare the computational resources used by the two
pipelines.

## 2. Evaluation dimensions

Each item is rated on a 1--5 scale across four dimensions.

  -----------------------------------------------------------------------
  Dimension                           Evaluation question
  ----------------------------------- -----------------------------------
  Novelty                             How novel is the idea relative to
                                      the available literature?

  Significance                        How important would addressing the
                                      gap or testing the hypothesis be?

  Feasibility                         Can the proposed research
                                      realistically be carried out with
                                      available tools, resources, and
                                      knowledge?

  Testability                         Can the claim or hypothesis be
                                      tested with sufficient
                                      concreteness?
  -----------------------------------------------------------------------

A higher score indicates a stronger assessment on that dimension.

## 3. Evaluators

The supplied evaluation sheet contains five LLM evaluators:

-   GPT
-   Gemini
-   Claude
-   Grok
-   DeepSeek

Each evaluator assigns a score for every item and dimension. The sheet
also contains an average across the five evaluators for each dimension.

## 4. Evaluation unit and data layout

The evaluation unit is an individual research gap or hypothesis from a
particular run.

The supplied `05_evaluation_scores_filled.csv` contains:

-   Run ID
-   Gap/hypothesis ID
-   Item type
-   Item text
-   Five evaluator scores and an average for each of the four dimensions

The file contains 234 rows including two header rows, corresponding to
232 evaluated item instances. The score matrix has 28 columns.

`05_run_type_average_scores.csv` aggregates dimension scores by run and
item type (gaps or hypotheses). `run_table.csv` contains combined
run-level metrics and quality summaries.

## 5. Computational-efficiency measures

The run metadata tracks:

-   Total API calls
-   Failed API calls
-   Input tokens
-   Output tokens
-   Total tokens
-   Runtime in seconds
-   Maximum refinement/debate rounds
-   Number of generated gaps and hypotheses

Token counts are used as a cost proxy. Monetary cost is not calculated
because the supplied model pricing configuration has no input/output
prices set.

## 6. Comparability rules

-   Primary quality and cost comparisons use D01 only.
-   D02, D03, and D04 are incomplete corpus variants and are excluded
    from the primary comparison.
-   The D01 comparison contains 5 single-agent runs and 6 multi-agent
    runs.
-   Run-level means and variation should be reported; individual items
    within a run are not independent experimental runs.

## 7. Aggregation and reporting

Report the mean score for each dimension and the overall mean across the
four dimensions. Keep gaps and hypotheses distinguishable when analysing
them separately. For computational cost, report run-level means and,
where useful, run-to-run spread.

The charts supplied with the project show individual run observations
alongside group summaries. Their captions should state the number of
runs and the aggregation procedure.

## 8. Evaluation limitations

-   LLM evaluators are not independent human domain experts.
-   Scores are subjective model judgements and may share biases.
-   The five evaluators' outputs are not a substitute for inter-rater
    validation with domain experts.
-   A mean score can conceal disagreement between evaluators and
    variation between runs.
-   Quality scores do not establish that an idea is genuinely novel
    across the full literature.

## 9. Data-integrity note

The supplied quality-by-dimension plot reports group means that do not
exactly match the means recomputed from the supplied `run_table.csv` and
`05_run_type_average_scores.csv`. The discrepancy is recorded in
`06_results.md`; the aggregation and source version should be reconciled
before the plotted values are used as final manuscript statistics.
