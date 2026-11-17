# 07 — Example Output

This document embeds the prompt templates and the complete generated research gaps and hypotheses from two supplied runs. The prompt function definitions are reproduced from each run's saved `prompt_snapshot.py`; they are not reconstructed summaries.

The prompt functions contain runtime placeholders such as `{topic}`, `{cards}`, and `{records}`. At execution, the code fills these with the configured topic, paper summaries, and current records.

---

# A. Single-agent run — `1794725368`

**Prompt snapshot:** `run_1794725368_prompt_snapshot.py`  
**Output artifacts:** `run_1794725368_identified_research_gaps.txt` and `run_1794725368_proposed_hypothesis.txt`

## A1. Actual prompt templates

### `_reader_prompt`

```python
def _reader_prompt(topic, additional, paper_id, path):
    return f"""
ROLE: Literature Reader
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}
PAPER ID: {paper_id}
PAPER FILE: {Path(path).name}
Read only the attached paper. Separate explicit statements from inference.
Capture section/page references wherever possible. A limitation is not
automatically a research gap. Return only valid JSON:
{{
 "paper_title":"","authors_year":"","research_question":"","context":"",
 "methods":[],"main_findings":[],"assumptions":[],"limitations_explicit":[],
 "potential_open_questions":[{{"question":"","why_unresolved":"","evidence":"","evidence_excerpt":"","sections":[]}}],
 "candidate_hypotheses":[{{"hypothesis":"","rationale":"","evidence":"","evidence_excerpt":"","sections":[]}}]
}}
For each potential_open_question and candidate_hypothesis, provide evidence_excerpt
as a CONTEXTUAL VERBATIM EXCERPT, not merely the single sentence that states the
limitation or future direction. Copy the continuous passage from the PDF, preserving
the original wording. Include a few complete sentences immediately BEFORE the key
evidence and a few complete sentences immediately AFTER it, so the excerpt captures
the preceding discussion, reasoning, qualifications, results, or follow-up discussion.
Aim for roughly 5-10 sentences or about 100-180 words per excerpt when the source
contains that much relevant context. Include the full relevant paragraph(s) when
shorter; do not add unrelated text just to meet a length target. Do not stitch
non-contiguous sentences together. Never invent, paraphrase, or silently correct
quoted text. Include the printed page number and section heading in sections when
identifiable. If no directly relevant passage exists, use an empty string for
evidence_excerpt. Keep evidence as a concise explanation separate from the verbatim
excerpt. Do not claim novelty beyond this paper."""
```

### `_synthesis_prompt`

```python
def _synthesis_prompt(topic, additional, cards):
    return f"""
You are a single scientific researcher synthesizing a supplied literature corpus.
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}

Use ONLY the paper summaries below as evidence. Do not imply that this corpus
represents all published literature. Distinguish explicit findings, reasonable
inferences, and speculation.

Produce a concise historical account and specific research gaps numbered G001,
G002, ... in order. There is no maximum number of gaps: identify every distinct,
well-supported gap that the corpus justifies. Ten is a target, not a cap or a
requirement to invent gaps. Each gap must explain what is known, what remains
unknown and why it matters. Cite direct evidence and supporting papers using only
supplied paper IDs. Do not generate hypotheses at this stage.

Do not call something a research gap merely because one paper does not discuss it.
A limitation is not automatically a research gap. Avoid duplicate or overlapping gaps.
Do not claim novelty beyond the supplied papers; a broader literature review would
be required to establish novelty.

Return ONLY valid JSON, no Markdown:
{{"historical_analysis":"",
"research_gaps":[{{"id":"G001","gap":"","why_it_matters":"","what_is_known":"",
"what_remains_unknown":"","gap_type":"explicit|inferred","confidence":"low|medium|high",
"supporting_papers":[{{"paper_id":"P001","sections":[]}}],
"evidence":[{{"paper_id":"P001","section":"","evidence":""}}]}}]}}

PAPER SUMMARIES:
{_dump(cards)}
"""
```

### `_gap_generation_prompt`

```python
def _gap_generation_prompt(topic, additional, cards, target_count):
    return f"""
You are a single scientific researcher conducting a comprehensive literature-gap
analysis of the supplied corpus in ONE attempt.
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}
TARGET: identify at least {target_count} distinct, substantive research gaps if the
evidence supports them.

Examine the entire corpus systematically: methods, assumptions, datasets,
evaluation choices, theoretical claims, explicit limitations, conflicting findings,
and underexplored connections. Aim for at least {target_count} gaps in this one
response. Do not stop after finding only a few obvious gaps. However, stop when no
further NEW, solid, scientifically meaningful gap can be supported by the corpus.
The target is not permission to invent weak, speculative, duplicate, or unsupported
gaps. It is acceptable to return fewer than {target_count} only when the corpus
genuinely supports no more distinct gaps.

Use ONLY the supplied paper summaries. Do not imply that this corpus represents all
published literature. Distinguish explicit statements from reasonable inferences.
A limitation or a paper's silence is not automatically a research gap. Do not split,
rephrase, or lightly modify one issue to inflate the count.

For every gap, explain what is known, what remains unknown, and why it matters.
Cite direct evidence and supporting papers using only supplied paper IDs. Use
specific section references wherever available. Keep gaps distinct and grounded.
Assign IDs sequentially as G001, G002, ... in the order returned.

Return ONLY valid JSON, no Markdown, with exactly this structure:
{{"historical_analysis":"",
"research_gaps":[{{"id":"G001","gap":"","why_it_matters":"","what_is_known":"",
"what_remains_unknown":"","gap_type":"explicit|inferred","confidence":"low|medium|high",
"supporting_papers":[{{"paper_id":"P001","sections":[]}}],
"related_papers":[{{"paper_id":"P001","sections":[],"relation":"addresses|contradicts|extends|mentions","explanation":""}}],
"evidence":[{{"paper_id":"P001","section":"","evidence":""}}]}}]}}

PAPER SUMMARIES:
{_dump(cards)}
"""
```

### `_hypothesis_prompt`

```python
def _hypothesis_prompt(topic, additional, gaps, cards):
    return f"""
You are the same single scientific researcher. Generate testable hypotheses only
AFTER the research gaps have been critiqued and finalised.
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}

Use the final gaps and the supplied paper summaries. Generate as many or as few
hypotheses as the gaps and scientific reasoning justify. There is no numerical
minimum, maximum, or target average. Do not force one hypothesis per gap. Every
hypothesis must be specific and falsifiable, cite one or more valid originating gap
IDs, and include supporting papers/sections, evidence, rationale, prediction and test.
Do not claim novelty beyond this corpus.

Return ONLY valid JSON:
{{"hypotheses":[{{"id":"H001","hypothesis":"","rationale":"",
"testable_prediction":"","proposed_test":"","originating_gap_ids":["G001"],
"supporting_papers":[{{"paper_id":"P001","sections":[]}}],
"evidence":[{{"paper_id":"P001","section":"","evidence":""}}]}}]}}

FINAL RESEARCH GAPS:
{_dump(gaps)}
PAPER SUMMARIES:
{_dump(cards)}
"""
```

### `_critique_prompt`

```python
def _critique_prompt(topic, additional, rnd, max_rounds, records, cards, subject):
    return f"""
You are the SAME single researcher who produced the records below.
Perform ONE self-critique (round {rnd}/{max_rounds}) of EVERY {subject} record
in this batch. Evaluate only the records supplied in this batch. Do not simulate other agents, personas, reviewers, or a debate.
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}
It is not mandatory to propose changes. Only suggest changes when necessary; do not
manufacture problems. Categories: unsupported_by_cited_papers, already_addressed,
infeasible, vague, non_falsifiable. Treat a gap that rests only on a paper's silence as
unsupported_by_cited_papers, and duplicate or overlapping records as a merge. Cite
corpus paper IDs and sections. Corpus-only, no web search or outside knowledge.
Return only JSON:
{{"proposed_changes":[{{"target_id":"","action":"revise|reject|merge|retain",
"category":"unsupported_by_cited_papers|already_addressed|infeasible|vague|non_falsifiable",
"argument":"","evidence":[],"paper_references":[],
"proposed_revision":"","merge_with":[]}}],"no_change_reason":""}}
RECORDS: {_dump(records)}
Use the paper summaries' evidence_excerpt fields as the primary source passages.
Check whether each excerpt directly supports the associated claim. Quote or cite
these exact passages in proposed changes where useful; do not fabricate quotations.
Do not expect or refer to attached PDFs in this critique request.

PAPER SUMMARIES: {_dump(cards)}
"""
```

### `_revision_prompt`

```python
def _revision_prompt(topic, additional, rnd, max_rounds, records, changes, cards):
    return f"""
You are the SAME single researcher who produced the records and wrote the critique below.
Perform ONE revision (round {rnd}/{max_rounds}). Do not simulate other agents,
personas, reviewers, or a debate, and do not critique the revised version again.
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}
For every critique, accept it (revise, merge or reject the record) or decline it
(retain). Do not accept merely to comply and do not decline merely to keep the draft.
For a revision, rewrite the COMPLETE record including supporting_papers, related_papers
and evidence. Never change or renumber IDs. A merge keeps one existing ID and lists
absorbed IDs. Do not silently drop records. Preserve uncertainties. Cite corpus paper IDs
and sections.
Return only JSON:
{{"decisions":[{{"target_id":"","decision":"retain|revise|merge|reject",
"reason":"","merged_ids":[],"revised_record":{{"id":"",
"gap_or_hypothesis_text":"","why_it_matters":"","what_is_known":"",
"what_remains_unknown":"","rationale":"","testable_prediction":"",
"proposed_test":"","supporting_papers":[],"related_papers":[],"evidence":[]}},
"remaining_uncertainties":[]}}],"round_summary":""}}
RECORDS: {_dump(records)}
CRITIQUE: {_dump(changes)}
PAPER SUMMARIES: {_dump(cards)}
"""
```

## A2. Generated research gaps

```text
- G001: Lack of deterministic, non-exhaustive algorithms for identifying optimal trimming regions without enforcing strict spherical symmetry and equal dispersion radii across clusters. [Papers: P002 (4. Application); P003 (4. MIXTURES OF ELLIPTICAL DISTRIBUTIONS)]

- G002: Absence of principled, automated approaches for estimating data-dependent trimming parameters (alpha) and minimum membership thresholds (s) in relaxed robust clustering frameworks. [Papers: P009 (6. Conclusions and future work)]

- G003: The mismatch between worst-case universal breakdown point vulnerability (O(1/n)) and robust practical performance under well-structured data assumptions. [Papers: P005 (3.2 Unbiased proximal maps: the case of non-convex f\lambda, 3.3 Restricted robustness of robust k-means for well-clustered data)]

- G004: Lack of parametric closed-form probability distributions for pseudo-isolation and distance-based outlier scores in nonparametric trimming frameworks. [Papers: P015 (8 Discussion)]

- G005: Vulnerability of federated outlier-aware K-means calibration to commissioning window contamination and non-robust sample covariance estimators. [Papers: P017 (Section 6.7. Contamination of the Trusted Anchor, Section 7.4. Future Work)]

- G006: Unresolved exponential dependence on the outlier count m in FPT approximation algorithms for metric clustering with outliers. [Papers: P013 (5. Concluding Remarks)]

- G007: Computational trade-offs and scalability bottlenecks of recursive, attribute-wise robust statistical rules (such as Tukey's fence or median absolute deviation) when applied to large-scale high-dimensional data streams. [Papers: P010 (7. Conclusion and future work)]

- G008: Lack of streaming concept drift and adaptive refresh mechanisms for benign anchors in federated and distributed outlier-detection K-means deployments. [Papers: P017 (Section 7.4. Future Work)]
```

## A3. Generated hypotheses

```text
- H001: Formulating optimal trimming regions as non-spherical geometric unions of dispersion-parameterized scatter matrices rather than equal-radius balls enables robust recovery of heterogeneous, nearby, or elongated clusters.
Rationale: Standard trimmed K-means theoretical characterizations restrict optimal trimming sets and cluster boundaries to unions of spherical balls with equal radii, causing the algorithm to fail or badly detect nonspherical or differentially dispersed clusters.
Testable prediction: On synthetic and real datasets with elongated or heterogeneous elliptical dispersions, an elliptical-dispersion-based trimming formulation will achieve higher adjusted Rand index (ARI) and better cluster recovery compared to spherical trimmed K-means.
References: P002, P003

- H002: Coupling relaxed continuous simplex clustering objectives with adaptive, objective-driven model selection criteria (such as adapted X-means/G-means splitting rules) enables automated, data-driven estimation of trimming proportions (alpha) and minimum memberships (s).
Rationale: Current relaxed robust clustering frameworks rely on manual tuning or arbitrary parameter sweeps over alpha and s, limiting their deployment in fully automated, large-scale data pipelines.
Testable prediction: An automated parameter selection pipeline that optimizes RTKM objective bounds during iterative cluster splitting will estimate alpha and s within 5% of empirically optimal grid-search values while matching F1-score performance.
References: P009

- H003: Establishing restricted non-asymptotic breakdown metrics under (rho_1, rho_2)-balanced inlier separation bounds eliminates the worst-case O(1/n) vulnerability paradox for robust K-means algorithms.
Rationale: Universal breakdown point analyses dictate that robust K-means objectives fail under arbitrary contamination with as few as one or two outliers, creating a false impression of fragility that contradicts their strong empirical success on well-structured data.
Testable prediction: Under bounded well-separated inlier ball conditions, the maximum tolerable contamination fraction before cluster center perturbation exceeds delta scales independently of total sample size n for unbiased proximal maps.
References: P005

- H004: Assuming a parametric underlying generative model for Mahalanobis pseudo-isolation scores yields an analytical closed-form probability distribution, permitting exact p-value significance testing for individual edge observations.
Rationale: Non-parametric frameworks rely on conservative concentration bounds (such as Cantelli's inequality) because exact analytical distributions for pseudo-isolation scores are unknown, preventing exact hypothesis testing.
Testable prediction: A parametric closed-form density function fitted to pseudo-isolation scores will approximate empirical tail probabilities with a Kolmogorov-Smirnov test statistic p-value > 0.05 on synthetic Gaussian mixture data.
References: P015

- H005: Replacing non-robust sample mean and covariance estimators with the Minimum Covariance Determinant (MCD) estimator in federated baseline K-means calibration preserves outlier detection recall under dirty commissioning windows with up to 10% attack contamination.
Rationale: In federated and edge IoT deployments, if initial baseline commissioning windows contain modest amounts of attack traffic, non-robust sample covariance estimators inflate along attack directions, shrinking anomaly distances and destroying detection recall.
Testable prediction: Integrating MCD into federated baseline K-means maintains detection recall above 85% when commissioning windows contain 5% attack contamination, compared to a drop below 30% for standard sample covariance estimators.
References: P017

- H006: Coreset condensation and ring-partitioning reductions can eliminate the exponential dependence on the outlier count m in FPT approximation algorithms for metric clustering with outliers, achieving running times FPT in k and epsilon alone.
Rationale: State-of-the-art parameterized approximation algorithms exhibit running times with an exponential factor ((k+m)/epsilon)^{O(m)} in terms of outliers m, creating scalability bottlenecks for large contamination regimes.
Testable prediction: A refined coreset sampling construction can bound the size of the enumerated outlier subset independently of m, reducing runtime complexity from ((k+m)/epsilon)^{O(m)} to FPT in k and epsilon while preserving an (1+epsilon)-approximation.
References: P013

- H007: Approximating recursive attribute-wise robust statistical rules using online streaming sketching (such as distributed quantile sketches) reduces the time complexity of outlier removal to sublinear time in data stream applications.
Rationale: Recursive attribute-wise robust statistical rules like Tukey's fence or median absolute deviation introduce significant computational overhead when computed iteratively across high-dimensional features in big data streams.
Testable prediction: Integrating quantile sketching into attribute-wise robust K-means reduces per-iteration runtime by at least 70% on streaming datasets of size n > 1,000,000 while maintaining within 1% of the baseline clustering accuracy and outlier F1-score.
References: P010

- H008: Equipping federated K-means deployments with lightweight streaming drift detectors (such as Page-Hinkley tests) and dynamic anchor-refresh protocols maintains false-positive rate stability under non-IID concept drift.
Rationale: IoT and networked systems experience continuous environmental changes, making fixed one-time commissioning anchors stale over extended operational periods and degrading false-positive guarantees.
Testable prediction: A federated K-means framework augmented with adaptive anchor refresh maintains a stable false-positive rate (FPR variance < 0.005) under continuous concept drift, whereas a static anchor protocol experiences an FPR increase exceeding 15%.
References: P017
```

---

# B. Multi-agent debate run — `1794730841`

**Prompt snapshot:** `run_1794730841_prompt_snapshot.py`  
**Output artifacts:** `run_1794730841_identified_research_gaps.txt` and `run_1794730841_proposed_hypothesis.txt`

## B1. Actual prompt templates

### `_reader_prompt`

```python
def _reader_prompt(topic, additional, paper_id, path):
    return f"""
ROLE: Literature Reader
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}
PAPER ID: {paper_id}
PAPER FILE: {Path(path).name}
Read only the attached paper. Separate explicit statements from inference.
Capture section/page references wherever possible. A limitation is not
automatically a research gap. Return only valid JSON:
{{
 "paper_title":"","authors_year":"","research_question":"","context":"",
 "methods":[],"main_findings":[],"assumptions":[],"limitations_explicit":[],
 "potential_open_questions":[{{"question":"","why_unresolved":"","evidence":"","evidence_excerpt":"","sections":[]}}],
 "candidate_hypotheses":[{{"hypothesis":"","rationale":"","evidence":"","evidence_excerpt":"","sections":[]}}]
}}
For every potential_open_question and candidate_hypothesis, provide an evidence_excerpt
as a contextual, verbatim quotation copied continuously from the attached paper.
Include several complete sentences before and after the key evidence when relevant,
aiming for 5–10 sentences or roughly 100–180 words. Include the full relevant
paragraph(s) when the discussion is shorter; do not add unrelated text merely to
reach a length target. Do not stitch together non-contiguous passages, paraphrase,
invent, or silently correct the quotation. Keep "evidence" as a concise explanation
of why the excerpt supports the item; it is separate from the verbatim excerpt.
Record the printed page number and section heading in "sections" when identifiable.
Do not claim novelty beyond this paper.
"""
```

### `_finder_prompt`

```python
def _finder_prompt(topic, additional, cards):
    return f"""
ROLE: Topic-Adaptive Perspective Finder
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}
Discover perspectives that collectively cover all plausible research-generation
areas for this topic, including areas likely to be found without assigned roles.
Minimize overlap. Examples below are illustrative, not fixed defaults:
{_dump(EXAMPLE_PERSPECTIVES)}
Return a compact set of distinct perspectives. For each provide name,
description, excludes, owned_by, boundary_rule and rationale. Explicitly state
which perspective owns overlapping/root-cause areas. Do not impose a fixed
number; prioritize broad coverage with minimum overlap.
Return only JSON:
{{"perspectives":[{{"name":"","description":"","excludes":[],"owned_by":[],
"boundary_rule":"","rationale":""}}],"coverage_check":"","overlap_check":""}}
PAPER SUMMARIES:
{_dump(cards)}
"""
```

### `_gap_prompt`

```python
def _gap_prompt(topic, additional, perspective, cards):
    return f"""
ROLE: Research-Gap Generator
TOPIC: {topic}
USER INSTRUCTIONS: {additional or "None"}
PERSPECTIVE: {_dump(perspective)}
Identify evidence-supported explicit or inferred gaps within this perspective.
Respect its exclusions and ownership boundaries. Each gap must explain what is
known, what remains unknown and why it matters. Cite direct evidence and
supporting papers using only supplied paper IDs. Use section references where
possible; sections may be empty but paper IDs must be valid.
Return only JSON:
{{"perspective":"{perspective['name']}","research_gaps":[{{
"gap":"","why_it_matters":"","what_is_known":"","what_remains_unknown":"",
"gap_type":"explicit|inferred","confidence":"low|medium|high",
"supporting_papers":[{{"paper_id":"P001","sections":[]}}],
"evidence":[{{"paper_id":"P001","section":"","evidence":""}}]
}}]}}
PAPER SUMMARIES:
{_dump(cards)}
"""
```

### `_curator_prompt`

```python
def _curator_prompt(topic, candidates, cards):
    return f"""
ROLE: Research-Gap Curator and Corpus Analyser
TOPIC: {topic}
First merge only genuinely repetitive gaps (same question and substantially
same resolution); do not create umbrella gaps. Reject duplicates/non-gaps.
Do not rank. Keep one existing ID for a merge and list absorbed IDs.
Then independently check EVERY candidate against EVERY paper summary,
including papers not cited by that candidate. Add related_papers with relation
addresses, contradicts, extends or mentions, plus sections and explanation.
Assess candidates using the supplied paper summaries together with sound scientific
judgement. Consider whether a gap is already known, implausible, poorly framed
or genuinely meaningful. Do not treat absence from these summaries as proof
of novelty; distinguish corpus-supported observations from broader scientific
assessments when relevant.
Every input ID must be retained, merged or rejected; never silently drop.
Preserve/union supporting_papers, evidence and origin perspectives.
Return only JSON:
{{"research_gaps":[{{"id":"G001","decision":"retain|merge|reject","merged_ids":[],
"reason":"","gap":"","why_it_matters":"","what_is_known":"",
"what_remains_unknown":"","gap_type":"","confidence":"",
"origin_perspectives":[],"supporting_papers":[],"related_papers":[],"evidence":[]}}],
"rejected":[{{"id":"G000","reason":""}}],"curation_summary":""}}
CANDIDATE GAPS:
{_dump(candidates)}
ALL PAPER SUMMARIES:
{_dump(cards)}
"""
```

### `_critique_prompt`

```python
def _critique_prompt(topic, kind, rnd, max_rounds, perspective, records, cards):
    return f"""
ROLE: Independent Scientific Critic
TOPIC: {topic}; SUBJECT: {kind}; ROUND: {rnd}/{max_rounds}
PERSPECTIVE NAME: {perspective.get("name", "")}
PERSPECTIVE DESCRIPTION: {perspective.get("description", "")}
SCOPE EXCLUSIONS: {_dump(perspective.get("excludes", []))}
SCOPE OWNERSHIP: {_dump(perspective.get("owned_by", []))}
BOUNDARY RULE: {perspective.get("boundary_rule", "")}

Critically evaluate EVERY supplied record from this perspective's domain.
The description and boundary rule define the scope of your expertise; do not
give generic criticism detached from this scope. Judge each record's novelty,
scientific/theoretical soundness, feasibility, significance and testability
where those dimensions apply. For example, a theoretical perspective should
assess whether the reasoning, assumptions, mechanisms and expected implications
are theoretically coherent and defensible, and whether the idea is meaningfully
new within that domain. Other perspectives should apply equivalent standards
appropriate to their own scope.

Identify substantive weaknesses, hidden assumptions, missing controls or
comparisons, implausible claims, overlooked prior ideas, and ways to strengthen
the record. A record may be strong; do not manufacture criticism. Use the
supplied paper summaries and relevant scientific reasoning. You may draw on
your existing scientific knowledge where useful, but do not invent citations
or present unsupported claims as established facts. Clearly distinguish
paper-supported evidence from your own reasoning. Cite paper IDs/sections
when referring to supplied papers. Do not limit your assessment to what is
explicitly stated in the supplied corpus.

For every record, return a concise but substantive assessment. Set
recommendation to retain, revise, merge or reject. Use revise only when you
can explain a concrete improvement; use merge only when another supplied
record is substantially equivalent. If no change is warranted, recommend
retain and explain why. Keep IDs unchanged.

Return only JSON:
{{"perspective":"{perspective.get("name", "")}","critiques":[{{
"target_id":"","assessment":"","novelty":"","scientific_or_theoretical_soundness":"",
"feasibility":"","significance":"","testability":"","argument":"",
"evidence":[],"paper_references":[],"recommendation":"retain|revise|merge|reject",
"proposed_revision":""}}],"overall_assessment":""}}
RECORDS:
{_dump(records)}
PAPER SUMMARIES:
{_dump(cards)}
"""
```

### `_adjuster_prompt`

```python
def _adjuster_prompt(topic, kind, rnd, max_rounds, records, critiques, cards):
    return f"""
ROLE: Scientific Adjuster
TOPIC: {topic}; SUBJECT: {kind}; ROUND: {rnd}/{max_rounds}

You are the final decision-maker for this debate round. You receive every
active record, every perspective's critique as a separate assessment, and
the COMPLETE paper cards for the entire corpus.

Independently inspect the paper cards when evaluating the critiques. Check
supporting AND contradictory findings, methods, assumptions, limitations and
open questions across the full corpus. Do not rely only on evidence excerpts
or on the critics' summaries.

Evaluate EVERY individual critique separately, including critiques that
disagree with one another. Do not use majority voting, consensus, or the number
of perspectives supporting a position as a reason to accept or reject it.
Assess each critique on its own merits, considering the perspective's stated
expertise and scope, the record being assessed, and the paper-card evidence.
A single well-supported critique may justify a change; several weak critiques
do not become strong through agreement.

For EACH critique, explicitly record:
- whether it is accepted, partially accepted, or rejected;
- why that specific criticism is accepted or rejected;
- which parts are incorporated into the final decision;
- which points are overlooked or not acted upon, and the reason.
No critique may be silently ignored. If a critique is irrelevant, out of scope,
unsupported, mistaken, or redundant, state that explicitly and explain why.
If it is valid but does not warrant changing the record, explain why it is
not acted upon. Distinguish paper-card evidence from scientific reasoning.

After evaluating all critiques individually, make ONE final decision per
record: retain, revise, merge or reject. The final decision must be consistent
with the individual critique assessments, or explain any apparent difference.
For a revision, rewrite the COMPLETE record, preserving its scientific meaning
where possible and incorporating only justified improvements. Include
supporting_papers, related_papers and evidence. Never change or renumber IDs.
A merge keeps one existing ID and lists absorbed IDs. Do not silently drop
records. Preserve uncertainties. If no improvement is justified, retain.

Return only JSON:
{{"critique_assessments":[{{
"perspective":"","target_id":"","critique_summary":"",
"disposition":"accepted|partially_accepted|rejected",
"reason":"","incorporated_points":[],"overlooked_points":[],
"evidence_checked":[],"paper_references":[]
}}],
"decisions":[{{"target_id":"","decision":"retain|revise|merge|reject",
"reason":"","merged_ids":[],"revised_record":{{"id":"",
"gap_or_hypothesis_text":"","why_it_matters":"","what_is_known":"",
"what_remains_unknown":"","rationale":"","testable_prediction":"",
"proposed_test":"","supporting_papers":[],"related_papers":[],"evidence":[]}},
"remaining_uncertainties":[]}}],
"round_summary":""}}

RECORDS:
{_dump(records)}
INDIVIDUAL PERSPECTIVE CRITIQUES:
{_dump(critiques)}
COMPLETE PAPER CARDS:
{_dump(cards)}
"""
```

### `_hypothesis_prompt`

```python
def _hypothesis_prompt(topic, additional, perspective, gaps, cards):
    return f"""
ROLE: Scientific Hypothesis Generator
TOPIC: {topic}; USER INSTRUCTIONS: {additional or "None"}
PERSPECTIVE: {_dump(perspective)}
Generate testable hypotheses grounded in the final gaps. Each must cite one or
more originating gap IDs, and attach supporting paper IDs/sections and direct
evidence. Do not generate unrelated hypotheses or claim proof.
Return only JSON:
{{"perspective":"{perspective['name']}","hypotheses":[{{
"hypothesis":"","rationale":"","testable_prediction":"","proposed_test":"",
"originating_gap_ids":["G001"],"supporting_papers":[],"related_papers":[],"evidence":[]
}}]}}
FINAL GAPS: {_dump(gaps)}
PAPER SUMMARIES: {_dump(cards)}
"""
```

### `_final_prompt`

```python
def _final_prompt(topic, gaps, hypotheses):
    return f"""
ROLE: Final Output Curator
TOPIC: {topic}
Prepare final lists; do not rank. Remove only substantive duplicates. On merge,
keep an existing ID and list absorbed IDs. Never renumber IDs. Preserve
originating gap IDs, paper/section references, evidence and debate history.
Every input ID must be retained or explicitly listed as merged/rejected.
Return only JSON:
{{"research_gaps":[],"hypotheses":[],"merged_or_rejected":[],"curation_summary":""}}
GAPS: {_dump(gaps)}
HYPOTHESES: {_dump(hypotheses)}
"""
```

## B2. Generated research gaps

```text
- G001: Universal Breakdown Bounds and Worst-Case Robustness Limits in Arbitrary versus Well-Structured Metric Spaces [Papers: P002 (5. Conclusions); P005 (3.3 Restricted robustness of robust k-means for well-clustered data, 5)]

- G002: FPT Dependency Limits and Parameter Exponential Scaling in Clustering with Outliers [Papers: P013 (5. Concluding Remarks)]

- G003: Mathematical Limits and Closed-Form Analytical Distribution of Nonparametric Pseudo-Isolation Scores [Papers: P015 (8 Discussion)]

- G004: Generalization of trimmed k-means and robust distance metrics beyond mixtures of spherical or isotropic distributions [Papers: P003 (4. MIXTURES OF ELLIPTICAL DISTRIBUTIONS, Page 15); P015 (8 Discussion)]

- G005: Principled, automated parameter estimation for outlier proportions and cluster cardinality in robust k-means variants [Papers: P002 (5. Conclusions, Page 16); P009 (6. Conclusions and future work, Page 15); P014 (Section II, Section II-D, Page 3)]

- G007: Integration of robust covariance estimators and streaming concept-drift mechanisms into federated and distributed outlier-aware k-means architectures [Papers: P017 (4. The AF-BKM Method, 6. Results and Analysis, 6.7. Contamination of the Trusted Anchor)]

- G008: The empirical and trade-off sensitivity of robust thresholding coefficients (such as q standard deviations or trimming proportions alpha) across diverse non-Gaussian, high-dimensional, and large-scale data distributions remains underexplored beyond basic low-dimensional benchmarks. [Papers: P006 (VI. CONCLUSION); P007 (I. INTRODUCTION); P011 (3 Experiment, 3.1 Data Set)]

- G010: Integration of real-world qualitative and behavioral factors into robust clustering pipelines for predictive applications such as educational employability and career linearity. [Papers: P016 (4. CONCLUSION, Page 7)]

- G011: Handling streaming concept drift and automated dynamic anchor refreshing in resource-constrained federated outlier-aware clustering environments. [Papers: P017 (4. The AF-BKM Method, 4.2. Threat Model)]
```

## B3. Generated hypotheses

```text
- H001: Alternative penalty functions, non-convex proximal maps, or modified objective formulations cannot fundamentally elevate universal breakdown bounds beyond O(1/n) on arbitrary, unconstrained datasets without requiring prior geometric restrictions.
Rationale: Supported by variational analysis establishing O(1/n) universal breakdown limits on arbitrary metric spaces.
Testable prediction: Constructing adversarial contaminations exceeding 2/n will break unconstrained trimmed k-means center estimates.
References: P002, P005

- H002: The exponential FPT dependence on the number of outliers m is a fundamental computational barrier that cannot be eliminated or relaxed to depend solely on k and epsilon.
Rationale: Based on parameterized complexity reductions and coreset enumeration limits in outlier-aware clustering.
Testable prediction: Any exact reduction removing FPT dependence on m will violate conditional lower bounds unless P=NP.
References: P013

- H003: A fully parametric extension and closed-form analytical distribution for pseudo-isolation scores can be derived to enable exact hypothesis testing.
Rationale: Required for advancing nonparametric anomaly detection into exact parametric inference.
Testable prediction: Deriving score density under Gaussian assumptions will match empirical pseudo-isolation histograms asymptotically.
References: P015

- H004: Replacing non-robust sample covariance estimators with Minimum Covariance Determinant (MCD) estimators and robust dispersion updates in robust trimmed k-means optimization loops and federated architectures will maintain high anomaly recall and prevent cluster center degradation under directional adversarial contamination.
Rationale: Classical robust and trimmed k-means variants suffer from covariance inflation under attack vectors. Integrating robust dispersion matrices into alternating minimization updates addresses this vulnerability.
Testable prediction: Substituting standard sample covariance calculations with robust MCD updates in iterative trimmed clustering will maintain outlier detection recall and prevent cluster center breakdown under unconstrained adversarial contamination.
References: P005, P017

- H005: Embedding automated information-theoretic criteria (such as Minimum Description Length or Bayesian Information Criterion penalties) into the objective functions of robust trimmed k-means will enable simultaneous joint optimization of cluster count k, minimum cluster size s, and trimming proportion alpha without grid searching.
Rationale: Most robust k-means frameworks achieve strong performance given fixed hyperparameters but lack closed-form update rules or automated mechanisms for tuning alpha, s, and k, creating a major operational bottleneck.
Testable prediction: BIC-penalized robust trimmed k-means formulations will automatically recover the true underlying number of clusters and contamination fraction within 5% error on synthetic benchmark suites.
References: P009, P002

- H006: Replacing spherical distance metrics in trimmed k-means with cluster-specific Mahalanobis distances derived from iterative scatter estimation matrices will improve clustering accuracy and outlier separation on elliptical and non-spherical mixtures.
Rationale: Standard trimmed k-means and robust k-means rely heavily on spherical distance assumptions and ball intersections, causing them to fail when data exhibits complex elliptical or overlapping geometries.
Testable prediction: Covariance-adjusted trimmed k-means utilizing scatter estimation matrices will achieve higher Adjusted Rand Index (ARI) scores than spherical trimmed k-means on synthetic mixtures of elongated elliptical distributions.
References: P003

- H007: Systematic parameter sweeps of threshold coefficients (q) and trimming proportions (alpha) across diverse non-Gaussian and high-dimensional ODDS benchmark datasets will demonstrate distinct Pareto frontiers where low threshold coefficients maximize cluster purity while higher coefficients optimize precision at the expense of recall.
Rationale: Prior studies on low-dimensional datasets have shown that varying q or alpha trades off clustering MSE against false-positive rates and recall; validating this behavior across high-dimensional and non-Gaussian manifolds establishes the operational limits of robust thresholding.
Testable prediction: Varying q from 2 to 6 across high-dimensional ODDS benchmarks will reveal a statistically significant decrease in outlier recall paired with a corresponding asymptotic increase in precision exceeding 95%.
References: P006, P007, P011

- H010: Mechanisms for streaming concept-drift adaptation and automated anchor refreshing will keep benign calibration valid over extended operational lifespans without manual intervention in federated IoT intrusion detection systems.
Rationale: IoT and distributed monitoring infrastructures experience constantly evolving network traffic profiles, making fixed commissioning anchors and static thresholds vulnerable to performance degradation over extended operational lifespans.
Testable prediction: Dynamic anchor sliding windows will prevent false-positive inflation under gradual concept drift compared to fixed commissioning anchors.
References: P017
```

---

## Notes on interpretation

- The prompt templates above are source-exact function definitions from the saved snapshots. Dynamic content (paper cards, topic, records, critiques, and perspective definitions) is inserted at runtime and is retained in the corresponding inference-history artifacts, not embedded as static text in these template functions.
- The two examples are separate runs and may use different architecture snapshots. They demonstrate actual output formats and content, not a controlled pairwise comparison.
- Gap and hypothesis IDs are reproduced exactly as present in the supplied output text files.
