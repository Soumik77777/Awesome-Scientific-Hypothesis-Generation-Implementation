# 06 --- Results

## 1. Runs included

The experiment contains 14 runs in total. The primary comparison uses
the 11 runs on D01 (17 papers):

-   **Single-agent:** R04, R05, R06, R07, R10 (5 runs)
-   **Multi-agent:** R01, R02, R03, R11, R12, R13 (6 runs)

R08 (D02), R09 (D03), and R14 (D04) use incomplete corpus variants and
are excluded from the primary comparison.

## 2. Computational cost

The following values are recomputed from the supplied `run_table.csv`
for D01.

  Metric                             Single-agent (n=5)   Multi-agent (n=6)
  -------------------------------- -------------------- -------------------
  Mean total tokens per run                     327,650             738,215
  Mean API calls per run                           23.6                35.5
  Mean runtime per run (seconds)                  320.5               483.1

Multi-agent debate used approximately **2.25×** the total tokens of the
single-agent pipeline. The supplied cost figure rounds the token means
to approximately 328K and 738K and describes this as about 2.3×.

The multi-agent runs also made more API calls on average. Runtime is
more variable and should be interpreted cautiously.

## 3. Quality results

### Values shown in the supplied quality figure

The supplied quality-by-dimension figure reports the following group
means:

  Dimension        Single-agent   Multi-agent
  -------------- -------------- -------------
  Novelty                  2.81          2.69
  Significance             3.28          3.28
  Feasibility              3.35          3.60
  Testability              4.14          4.04
  Overall                  3.39          3.40

On the figure's values, multi-agent debate has a higher feasibility
score, while single-agent refinement has higher novelty and testability
scores. Significance is equal, and the overall means are nearly
identical.

### CSV recomputation check

The supplied `run_table.csv` contains a separate set of D01 run-level
means. Recomputing the mean across D01 runs gives:

  Dimension        Single-agent   Multi-agent
  -------------- -------------- -------------
  Novelty                  2.74          2.64
  Significance             3.28          3.32
  Feasibility              3.32          3.54
  Testability              4.06          3.99
  Overall                  3.35          3.37

These values do not match the group means printed in the supplied
quality figure. The difference may reflect a different aggregation
method or a different evaluation-data version. It should be resolved by
regenerating the figure from a single authoritative CSV and documenting
whether gaps and hypotheses are weighted equally or pooled at item
level.

## 4. Token use versus quality

The supplied cost-versus-quality plot reports a Spearman rank
correlation of approximately **−0.02** between total tokens and overall
quality for the 11 D01 runs. Recalculation from `run_table.csv` gives
−0.018 (approximately −0.02).

This indicates no meaningful monotonic association between spending more
tokens and receiving higher overall scores in this small set of runs. It
does not establish that token use has no causal effect.

## 5. Interpretation

The available results do not support the claim that multi-agent debate
inherently produces better overall research ideas. Its clearest
advantage in the supplied quality figure is feasibility, while the
single-agent pipeline is slightly stronger in novelty and testability.
The overall quality difference is negligible relative to the additional
token use.

The architecture comparisons also suggest that role differentiation and
interaction design may matter more than agent count alone. However, the
runs span multiple architecture snapshots, and the sample size is small;
these findings should be presented as exploratory.

## 6. Figures

The supplied figures correspond to:

1.  **Cost:** mean total tokens, API calls, and output tokens per run.
2.  **Quality:** group scores by evaluation dimension, separately for
    research gaps and hypotheses.
3.  **Cost versus quality:** individual run token usage against overall
    quality, with runs from D02--D04 marked as not directly comparable.

## 7. Reporting cautions

-   Use D01 only for the primary single-versus-multi comparison.
-   State the number of runs, not just the number of generated items.
-   Distinguish architecture comparisons from a pure agent-count
    comparison.
-   Resolve the quality-figure/CSV aggregation discrepancy before
    treating either set of group means as final.
-   Token usage is a proxy for computational cost; no monetary cost is
    available in the supplied configuration.
