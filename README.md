# ATR: Single-LLM Versus Multi-Agent Debate Systems for Scientific Hypothesis Generation

A research project investigating how single-LLM systems and multi-agent debate systems identify research gaps and generate scientific hypotheses. The repository contains the research specification, dataset documentation, experiment design, implementation, evaluation, results, and example outputs.

## Contents

- [Overview](#overview)
- [Research documentation](#research-documentation)
  - [Research problem](00_research_problem/00_research_problem.md)
  - [Dataset](01_dataset/01_dataset.md)
  - [Experiment blueprint](02_experiment_blueprint/02_experiment_blueprint.md)
  - [Codebase](03_codebase/03_codebase.md)
  - [Evaluation](04_evaluation/04_evaluation.md)
  - [Results](06_results/06_results.md)
  - [Example output](07_example_output/07_example_output.md)
  - [Research decisions](08_research_decisions/08_research_decisions.md)
- [Implementation paper](#implementation-paper)
- [Implementation prompt history](#implementation-prompt-history)
- [Code and execution](#code-and-execution)
- [Results and figures](#results-and-figures)
- [License](#license)

---

## Overview

The project compares two approaches to scientific hypothesis generation:

- **Single-LLM:** a single language model analyses the literature and generates, critiques, and refines research gaps and hypotheses.
- **Multi-agent debate:** multiple specialised agents contribute to the generation, critique, challenge, defence, and adjudication of candidate research gaps and hypotheses.

The comparison considers the quality of generated outputs alongside the computational cost of each approach. Evaluation dimensions include novelty, significance, feasibility, and testability.

## Research documentation

The numbered folders organise the project documentation by research stage:

| Stage | Documentation |
|---|---|
| Research problem | [00_research_problem.md](00_research_problem/00_research_problem.md) |
| Dataset and corpus | [01_dataset.md](01_dataset/01_dataset.md) |
| Experiment design | [02_experiment_blueprint.md](02_experiment_blueprint/02_experiment_blueprint.md) |
| Implementation and codebase | [03_codebase.md](03_codebase/03_codebase.md) |
| Evaluation methodology | [04_evaluation.md](04_evaluation/04_evaluation.md) |
| Results | [06_results.md](06_results/06_results.md) |
| Example output | [07_example_output.md](07_example_output/07_example_output.md) |
| Research decisions | [08_research_decisions.md](08_research_decisions/08_research_decisions.md) |

## Implementation paper

The implementation paper documents the implemented system and experimental work.

[View Implementation Paper](Implementation_Paper.pdf)

## Implementation prompt history

The prompt-history document records the prompts used during the development and refinement of the implementation.

[View Implementation Prompt History](Implementation_Prompt_History.pdf)

## Code and execution

The implementation is in [`03_codebase/`](03_codebase/):

- `main.py` — main entry point for running experiments.
- `single_agent.py` — single-LLM workflow.
- `multi_agent_debate.py` — multi-agent debate workflow.
- `evaluate_runs.py` — evaluation of generated runs.
- `requirements.txt` — Python dependencies.

The codebase documentation provides further implementation details and usage guidance.

## Results and figures

The [`06_results/`](06_results/) directory contains run and dataset mappings, architecture mappings, run metadata, evaluation scores, aggregated run-type scores, and result figures.

The [`07_example_output/`](07_example_output/) directory documents an example output from the system.

## License

See [LICENSE](LICENSE).
