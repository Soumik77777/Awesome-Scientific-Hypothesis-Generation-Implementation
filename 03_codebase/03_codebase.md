# 03 — Codebase

## Project files

The code in this directory is linked directly below. These are the supplied code snapshots dated 17 November 2026.

| File | Role |
|---|---|
| [`main.py`](./main.py) | Configuration, model initialisation, mode selection, run execution, diagnostics, and artifact writing |
| [`single_agent.py`](./single_agent.py) | Single-researcher literature synthesis, self-critique, and revision |
| [`multi_agent_debate.py`](./multi_agent_debate.py) | Topic-adaptive perspectives, multi-agent generation, critique, adjustment, and curation |
| [`evaluate_runs.py`](./evaluate_runs.py) | Run/dataset/architecture mappings, metadata extraction, and evaluation-sheet generation |
| [`requirements.txt`](./requirements.txt) | Pinned Python dependencies for the current Gemini/LangChain integration |

## Runtime configuration

The supplied `main.py` snapshot currently sets:

```python
MODE = "multi"
TOPIC = "Status and direction of outlier trimming in K-means and similar algorithms"
MODEL_NAME = "gemini-3.5-flash-lite"
TEMPERATURE = 0.2
KNOWLEDGE_CUTOFF_YEAR = 2026
MAX_ROUNDS = 3
PAUSE_SEC = 10
MAX_RETRIES = 3
```

The model allowlist does not pass `temperature` for `gemini-3.5-flash-lite`; therefore, `0.2` is a requested setting, not an applied setting for this model. The run metadata correctly records the applied temperature as unavailable (`NA`).

## Module responsibilities

### `main.py`

- Loads environment variables using `python-dotenv`.
- Selects `single` or `multi` mode.
- Initialises `ChatGoogleGenerativeAI`.
- Wraps structured-output calls to log token usage and inference events.
- Captures a run-specific prompt/code snapshot.
- Records package/environment diagnostics.
- Writes research gaps, hypotheses, API usage, inference history, and debug logs.

### `single_agent.py`

- Reads each paper individually.
- Produces structured paper summaries with contextual evidence excerpts.
- Generates and synthesises research gaps.
- Applies self-critique followed by revision.
- Generates hypotheses from finalised gaps.
- Applies hypothesis self-critique and revision.
- Preserves record IDs, evidence, paper references, and round history.

### `multi_agent_debate.py`

- Discovers topic-adaptive research perspectives.
- Generates perspective-specific gaps and hypotheses.
- Curates candidate gaps and checks candidates against the full corpus.
- Runs independent perspective-based critiques.
- Uses a central scientific adjuster to make record-level decisions.
- Performs final curation while preserving IDs and debate history.

### `evaluate_runs.py`

Reads the configured run folders without modifying them. It creates run-ID, dataset-ID, and architecture-ID mappings, run metadata, and a blank evaluation sheet. Architecture identity is based on a SHA-256 hash of the prompt snapshot; dataset identity is based on a fingerprint of the paper paths.

## Dependencies and installation

The code uses Python 3.11 or later. The requirements file pins the LangChain Google GenAI integration, its core abstraction package, Google's GenAI SDK, Pydantic, and `python-dotenv`.

From this directory, install with:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Set `GOOGLE_API_KEY` in the environment (or in a local `.env` file loaded by `main.py`). Do not commit credentials.

## Reproducibility notes

- Keep each run's `prompt_snapshot.py`, `api_usage.json`, and `inference_history.json`.
- Distinguish requested model parameters from parameters actually applied.
- Preserve the exact corpus and prompt snapshot used by each run.
- Do not treat D02–D04 as equivalent to D01.
- The current runner configuration may differ from the configuration used in historical runs.

## Version note

The pinned dependency set is selected for Python 3.11+ and the `langchain-google-genai` 4.5.0 integration, which uses the consolidated `google-genai` SDK. The pins satisfy the integration's declared minimums for `langchain-core` and `google-genai`. See the [langchain-google-genai release](https://pypi.org/project/langchain-google-genai/4.5.0/), [google-genai release](https://pypi.org/project/google-genai/3.2.0/), and [langchain-core release](https://pypi.org/project/langchain-core/1.8.1/). The project code should be smoke-tested in a clean virtual environment before a long experiment run.
