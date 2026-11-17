# 01 — Dataset

## Corpus

The experiment's literature corpus is a curated collection of **17 research papers** on K-means clustering, outlier detection, robust clustering, and related methods.

- Corpus identifier in the primary experiment: **D01**
- Paper identifiers: **P001–P017**
- File format: PDF
- Configured knowledge cutoff: 2026
- Dataset directory configured in `main.py`: `ATR/agents/dataset/papers/trimmed_k_means/`

### Bibliography

The corpus bibliography is maintained in [`corpus.bib`](./corpus.bib). It contains the BibTeX citation records used to identify and cite the papers.

## Corpus processing

The runner selects PDFs from year-named subdirectories under the configured dataset root, subject to the configured knowledge-cutoff year. Papers are processed individually. Each paper receives a stable paper ID and a structured analysis containing its research question, context, methods, findings, assumptions, explicit limitations, potential open questions, candidate hypotheses, evidence excerpts, and section/page references where available.

The paper IDs are used by both pipelines to trace generated gaps and hypotheses back to the supplied literature.

## Dataset variants in the experiment

| Dataset ID | Paper count | Interpretation |
|---|---:|---|
| D01 | 17 | Complete corpus; primary comparison |
| D02 | 15 | Incomplete subset |
| D03 | 13 | Incomplete subset |
| D04 | 16 | Incomplete subset |

D02–D04 arose from runs in which some papers were not successfully processed. They are not pooled with D01 for the primary single-agent versus multi-agent comparison.

## Dataset identity and reproducibility

`evaluate_runs.py` fingerprints the sorted, normalised paper paths to assign dataset IDs. It writes a dataset mapping CSV containing the dataset ID, paper count, and source paths. Each run's inference history also retains the paper paths and analysis records used for that run.

## Limitations

- The corpus is bounded and is not an exhaustive survey of all published research.
- Corpus-only generation may miss relevant work outside the supplied papers.
- A limitation or omission in one paper is not automatically a research gap.
- A claim of novelty beyond this corpus requires a broader literature review.
