# Research Decisions

This document consolidates the major research and implementation decisions recorded in the AI-assisted research and implementation log. It is intended as a concise reference for the ATR paper, not as a chronological prompt history.

Decisions that changed during development are described in their final or latest recorded form. Earlier settings are mentioned only where needed to explain the evolution.

## 1. Research objective: compare single-LLM and multi-agent systems

The study will compare a single-LLM workflow with a multi-agent debate workflow for scientific research-gap identification and hypothesis generation. The central question is whether structured multi-agent interaction improves the quality of generated outputs, rather than whether multi-agent systems are universally superior.

## 2. Use a literature-based, historically bounded input corpus

Both systems will work from a curated collection of research papers organised by publication year. A configurable cutoff determines which papers are available to a run. This supports controlled experiments in which the systems receive a defined literature snapshot.

## 3. Treat temporal validation as retrospective evidence, not ground truth

Generated directions may be compared with research published after the selected cutoff. However, later publication is only an indicator of future alignment: it does not establish that a hypothesis is scientifically good, and absence of a later matching paper does not establish that it is poor. Potential model knowledge leakage across the cutoff must also be acknowledged.

## 4. Use Ueda et al. (2025) as the principal architecture reference

Ueda et al. (2025) was selected as the main reference for the implementation design. The goal is selective adaptation of useful mechanisms, not a complete reproduction of that system or of the other reviewed systems.

## 5. Prioritise a working single-agent baseline before multi-agent implementation

Development began with a minimal PDF-based single-agent pipeline. The multi-agent implementation was deferred until the single-agent workflow, input handling, outputs, and usage tracking were functional.

## 6. Keep the initial document-ingestion scope to PDF

The first working prototype supports PDF papers only. TXT and Markdown ingestion, as well as additional convenience features, were deferred to avoid delaying a functional end-to-end system.

## 7. Make the workflow configurable through a single entry point

`main.py` is the entry point for selecting the execution mode, research topic, model, and publication-year cutoff, and for invoking the relevant agent pipeline. Single-agent and multi-agent logic are maintained in separate modules.

## 8. Record usage and preserve run-level provenance

Each run should retain the information needed to inspect and reproduce its processing: timestamped outputs, prompt snapshots, API usage, and inference history. Usage records include token counts, runtime, and errors where available. File writing is managed by the main execution layer rather than being scattered across agent modules.

## 9. Separate research gaps from proposed hypotheses

Research gaps and proposed hypotheses are distinct output types and should be stored separately. The outputs should retain their source-paper references and other provenance needed to trace claims back to the input corpus.

## 10. Align the single-agent and multi-agent output schemas

Both systems should produce compatible structured fields and human-readable inference histories. This allows the same downstream processing and evaluation pipeline to be applied to both systems and avoids giving either system an advantage through incompatible output formats.

## 11. Use a generate–critique–revise single-agent workflow

The single-agent baseline follows an iterative generation, critique, and refinement pattern. Prompt improvements drawn from the reference literature include awareness of previously proposed ideas, multiple critique perspectives where appropriate, and more concrete experimental plans. Optional variations were adopted selectively rather than all at once.

## 12. Make multi-agent perspectives topic-adaptive

The multi-agent system uses a perspective-finder stage to identify relevant, minimally overlapping perspectives for the selected research topic. Theoretical, computational, observational, and experimental perspectives are examples, not a mandatory fixed set for every topic. Each perspective should have a clear scope.

## 13. Distinguish curation, critique, defence, and adjudication

The debate workflow separates the management of candidate records from substantive debate. Curation handles duplicate or overlapping candidates; perspective agents critique candidates; challengers propose changes; defenders may respond; and an adjudicator makes the final decision. The analyser role was later merged into the curator in the revised design.

## 14. Preserve traceability throughout debate

Candidate identifiers and paper references must remain traceable across stages. Debate history should make it possible to see which agent raised a criticism, how it was answered, what the adjudicator decided, and how a candidate changed or was removed. Rejected candidates and decision events should remain available in the history rather than disappearing silently.

## 15. Bound debate iterations and avoid forced revisions

The revised architecture uses a maximum of three debate rounds. Agents are not required to propose changes when none are warranted. Debate should be driven by substantive criticism rather than by an assumption that every round must alter every candidate. Independent reasoning is required; simultaneous API execution is not.

## 16. Keep external web verification disabled in the initial implementation

Web-search grounding was deliberately commented out for the initial implementation. The debate could later use the model's internal knowledge, but mandatory external retrieval was not part of the initial workflow. This keeps the first comparison focused on the supplied corpus and the defined agent interaction.

## 17. Evaluate individual gaps and hypotheses on four dimensions

The evaluation uses four dimensions: **novelty, significance, feasibility, and testability**. Research gaps and hypotheses are evaluated as individual items. Feasibility and testability remain separate dimensions: an idea may be practical to pursue but insufficiently specified to test, or testable but difficult to execute.

## 18. Use structured evaluation outputs and independent evaluators

Evaluation results are stored in structured tables with mappings between runs, datasets, and architectures, alongside run-level metadata and item-level scores. Multiple independent LLM evaluators are used. Estimated API cost and a separate evidence-strength score were excluded from the final scoring dimensions recorded in the log.

## 19. Strengthen critique with explicit, category-wise checks

The single-agent critique must explicitly assess each candidate for:
- unsupported claims relative to cited papers;
- whether the problem has already been addressed;
- infeasibility;
- vagueness;
- lack of falsifiability; and
- duplication or substantial overlap.

A general, unstructured critique is insufficient. Each category should receive an explicit status and explanation.

## 20. Control large requests using token estimates and evidence-based batching

PDF-containing requests are monitored using token estimates, with Gemini's `count_tokens` preferred for measuring request size. The implementation initially used a 150K-token application-level ceiling and later revised the PDF-token threshold to 125K for critique batching. Batches should include the active records and relevant paper summaries/references while respecting the applicable limit. Cooldowns are conditional on recent large requests, not applied after every call.

Large-request failures, including HTTP 503 responses, were not treated as proof that the architecture or quota was inherently unsuitable. The log records that backend capacity was a plausible explanation, but not a confirmed cause; limits and batching should therefore be changed in response to measured evidence rather than speculation.

## 21. Preserve recoverable outputs and make parsing failures visible

Pipeline failures must not silently discard model outputs. Raw responses should be retained where possible, parsing and validation failures should be exposed in the run history, and recovery fixes should be applied locally when the output can be repaired without repeating expensive generation calls.

---

## Notes on decision status

- These are consolidated decisions, not a verbatim reproduction of every prompt or response.
- Some implementation parameters evolved. In particular, the critique PDF-token threshold changed from 150K to 125K in a later revision.
- Suggestions recorded as deferred in the log (for example, additional input formats, external benchmark datasets, and mandatory web verification) are not presented as implemented features.
- The log does not establish that every proposed implementation detail was independently validated in a completed experiment. Claims about experimental outcomes should be supported by run evidence elsewhere in the paper.
