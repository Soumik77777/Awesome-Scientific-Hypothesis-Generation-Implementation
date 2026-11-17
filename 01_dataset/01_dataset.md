# 01 --- Dataset

## Overview

The experiment uses a curated literature corpus on K-means clustering,
outlier detection, robust clustering, and related methods.

-   **Original corpus:** 17 research papers.
-   **Paper format:** PDF.
-   **Publication range:** 1967--2026.
-   **Paper identifiers:** P001--P017.
-   **Knowledge cutoff configured in the runner:** 2026.
-   **Dataset root in the code:**
    `ATR/agents/dataset/papers/trimmed_k_means/`.

The 17-paper corpus is referred to as **D01** in the experiment
analysis.

## Corpus preparation

The runner searches year-named subdirectories under the dataset root and
includes PDF files whose directory year is no later than the configured
knowledge-cutoff year. The selected paths are sorted before processing.

Each paper is analysed separately by the LLM. The structured paper
analysis captures:

-   Bibliographic information and research question
-   Context and methods
-   Main findings and assumptions
-   Explicit limitations
-   Potential open questions
-   Candidate hypotheses
-   Evidence excerpts and identifiable section/page references

Paper IDs are used to preserve traceability from generated gaps and
hypotheses back to the source corpus.

## Dataset variants in the experiment

  ------------------------------------------------------------------------
  Dataset ID                        Number of papers Use
  --------------------- ---------------------------- ---------------------
  D01                                             17 Original corpus;
                                                     primary comparison

  D02                                             15 Incomplete subset
                                                     from a run with
                                                     paper-processing
                                                     failures

  D03                                             13 Incomplete subset
                                                     from a run with
                                                     paper-processing
                                                     failures

  D04                                             16 Incomplete subset
                                                     from a run with
                                                     paper-processing
                                                     failures
  ------------------------------------------------------------------------

D02--D04 are subsets of the original corpus caused by unsuccessful paper
processing in particular runs. They are **not directly comparable** with
D01 and are excluded from the primary single-versus-multi comparison.

## Dataset identity

The evaluation script fingerprints the sorted, normalised paper paths to
assign dataset IDs. It also writes a dataset mapping CSV containing the
dataset ID, paper count, and paths.

## Limitations

-   The corpus is deliberately focused and relatively small.
-   Corpus-only generation can miss relevant work outside the supplied
    papers.
-   A paper's silence or an explicitly stated limitation is not, by
    itself, sufficient evidence of a research gap.
-   Incomplete datasets caused by API or processing failures must not be
    pooled with D01 when comparing output quality.

