# 03 --- Codebase

## 1. Overview

The project implements two alternative pipelines behind a shared runner:

-   `single_agent.py` --- single-LLM literature synthesis,
    self-critique, and revision.
-   `multi_agent_debate.py` --- adaptive multi-agent generation,
    perspective-based criticism, centralised adjustment, and final
    curation.
-   `main.py` --- configuration, model initialisation, execution,
    diagnostics, and output persistence.

The code snapshot supplied for this update is dated **2026-11-17**.

## 2. Project structure

The following is the logical structure reflected by the current code and
configured paths. It is not a claim that every directory/file has been
uploaded in this conversation.

``` text
ATR/
└── agents/
    ├── main.py
    ├── single_agent.py
    ├── multi_agent_debate.py
    ├── dataset/
    │   └── papers/
    │       └── trimmed_k_means/
    │           └── <publication_year>/
    │               └── *.pdf
    ├── output/
    │   └── run_<timestamp>/
    │       ├── run_<timestamp>_identified_research_gaps.txt
    │       ├── run_<timestamp>_proposed_hypothesis.txt
    │       ├── run_<timestamp>_prompt_snapshot.py
    │       ├── run_<timestamp>_api_usage.json
    │       ├── run_<timestamp>_inference_history.json
    │       ├── run_<timestamp>_inference_history.txt
    │       └── run_<timestamp>_debug_<timestamp>.jsonl
    └── analysis/
        ├── 01_run_id_mapping.csv
        ├── 02_dataset_id_mapping.csv
        ├── 03_architecture_id_mapping.csv
        ├── 04_run_metadata.csv
        └── 05_evaluation_scores.csv
```

## 3. `main.py`

Responsibilities:

-   Load environment variables using `python-dotenv`.
-   Configure the mode, topic, model, cutoff year, paths, and run
    controls.
-   Select the appropriate pipeline function.
-   Initialise `ChatGoogleGenerativeAI`.
-   Wrap the model with structured-output logging.
-   Snapshot the complete selected agent module for each run.
-   Record environment/package diagnostics.
-   Save readable gap and hypothesis outputs, API usage, inference
    history, and debug logs.

Important configuration values in the supplied snapshot:

``` python
MODE = "multi"
TOPIC = "Status and direction of outlier trimming in K-means and similar algorithms"
MODEL_NAME = "gemini-3.5-flash-lite"
TEMPERATURE = 0.2
KNOWLEDGE_CUTOFF_YEAR = 2026
MAX_ROUNDS = 3
PAUSE_SEC = 10
MAX_RETRIES = 3
```

The model allowlist intentionally does not pass temperature to
`gemini-3.5-flash-lite`; consequently, the requested temperature is not
applied in the recorded configuration.

## 4. `single_agent.py`

The single-agent module:

-   Reads PDFs individually using a common structured literature-reader
    prompt.
-   Synthesises paper summaries into research gaps.
-   Performs self-critique followed by revision.
-   Generates hypotheses only after gap refinement.
-   Independently critiques and revises hypotheses.
-   Preserves evidence, paper references, IDs, and per-round history.
-   Uses structured JSON parsing and records API usage and inference
    events.

One self-critique round consists of two calls: critique and revision.
The same researcher performs both.

## 5. `multi_agent_debate.py`

The multi-agent module:

-   Discovers topic-adaptive perspectives.
-   Generates perspective-specific research gaps and hypotheses.
-   Uses structured JSON Schema output.
-   Curates initial candidate gaps and checks them against the corpus.
-   Runs independent perspective critiques.
-   Uses a centralised adjuster to make record-level decisions.
-   Performs final curation.
-   Tracks evidence, supporting papers, IDs, decisions, and inference
    history.

The supplied architecture snapshot defines theoretical, computational,
observational, and experimental perspectives as examples; the
perspective finder can adapt the actual set to the topic.

## 6. Run outputs and logging

Each run folder stores:

  -----------------------------------------------------------------------
  Artifact                            Purpose
  ----------------------------------- -----------------------------------
  `identified_research_gaps.txt`      Human-readable gap list

  `proposed_hypothesis.txt`           Hypotheses, rationales,
                                      predictions, and references

  `prompt_snapshot.py`                Snapshot of the complete selected
                                      agent module

  `api_usage.json`                    Per-call token usage, runtime,
                                      status, and usage summary

  `inference_history.json`            Agent parameters, events, final
                                      result, and artifact paths

  `inference_history.txt`             Readable inference history

  `debug_*.jsonl`                     Structured-output request and
                                      environment diagnostics
  -----------------------------------------------------------------------

## 7. Evaluation utility

`evaluate_runs.py` reads run folders and creates run, dataset, and
architecture mappings, metadata, and a blank evaluation sheet. It
identifies architectures using SHA-256 hashes of prompt snapshots and
datasets using fingerprints of paper paths. It does not modify the
source run folders.

The supplied filled evaluation CSV and derived average-score CSVs are
analysis outputs, not generated directly by the blank-sheet creation
step.

## 8. Dependencies and configuration

The supplied `main.py` imports:

-   `langchain-google-genai`
-   `python-dotenv`
-   `langchain-core` (used by the agent modules)
-   `google-genai` (used by the Gemini integration)
-   `pydantic` (used by structured-output schemas)

The runner records the installed versions of `langchain-core`,
`langchain-google-genai`, `google-genai`, and `pydantic` in the debug
log. Exact package versions should be taken from a run's environment
diagnostics rather than hard-coded here.

Set the Google API key in the environment (typically through `.env`)
before execution. Do not commit API keys or other secrets.

## 9. Reproducibility notes

-   Keep the run-specific prompt snapshot with every result.
-   Preserve the complete inference history and API usage records.
-   Distinguish requested model parameters from parameters actually
    applied.
-   Do not treat D02--D04 as equivalent to D01.
-   The current `main.py` configuration is not necessarily identical to
    the settings used by every historical run.
