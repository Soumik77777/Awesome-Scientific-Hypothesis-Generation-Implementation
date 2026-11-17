# 00 --- Research Problem

## Project title

**Single-LLM Versus Multi-Agent Debate Systems for Scientific Hypothesis
Generation**

## Research topic

**Status and direction of outlier trimming in K-means and similar
algorithms.**

## Research problem

Large language models (LLMs) can analyse scientific literature, identify
potential research gaps, and propose testable hypotheses. Two broad ways
to organise this work are:

-   **Single-LLM refinement:** one model performs literature analysis,
    generation, self-critique, and revision.
-   **Multi-agent debate:** multiple role-specialised agents generate,
    critique, challenge, and refine candidate ideas.

Multi-agent systems introduce additional calls, coordination, and token
costs. It is therefore not self-evident that distributing the work among
agents produces sufficiently better scientific ideas to justify the
added computational expense.

## Aim

To experimentally compare a single-LLM iterative-refinement pipeline
with a multi-agent debate pipeline for literature-grounded research-gap
identification and hypothesis generation, while controlling the
underlying model, topic, and literature corpus.

## Research question

**Under a shared LLM, research topic, and literature corpus, does a
multi-agent debate architecture produce higher-quality research gaps and
hypotheses than iterative refinement by a single LLM, and how do any
quality differences compare with the additional computational cost?**

## Objectives

1.  Implement a single-LLM research-gap and hypothesis-generation
    pipeline.
2.  Implement a topic-adaptive multi-agent debate pipeline.
3.  Keep the model, topic, and corpus common across comparable runs.
4.  Evaluate generated research gaps and hypotheses on novelty,
    significance, feasibility, and testability.
5.  Compare token usage, API calls, runtime, and output quality.
6.  Examine whether architectural changes and role differentiation
    affect the quality-cost trade-off.

## Scope and delimitations

-   The scientific domain is outlier trimming in K-means and related
    algorithms.
-   Generation and critique are grounded in the supplied corpus;
    external web grounding and automatic literature verification are
    disabled.
-   The experiment evaluates generated ideas, not the eventual
    scientific validity of implementing every proposed method.
-   The supplied corpus is a bounded collection, not an exhaustive
    representation of all published research.

## Main comparison

  Aspect             Single-LLM            Multi-agent
  ------------------ --------------------- ---------------------------------
  Generation         One researcher        Perspective-specific generators
  Criticism          Self-critique         Independent perspective critics
  Revision           Same researcher       Centralised adjuster
  Final curation     No separate curator   Dedicated final curator
  Refinement limit   Up to 3 rounds        Up to 3 debate rounds

## Expected contribution

The study provides an empirical comparison of two LLM orchestration
strategies for scientific ideation, considering both evaluated output
quality and computational efficiency. It also examines whether role
design and interaction architecture matter more than simply increasing
the number of agents.
