---
name: early-precision-ranking-evaluation
description: Use when comparing GRN inference methods that each output a ranked edge
  list over the same gene set and a gold-standard regulatory network is available
  to compute average precision and early precision for benchmarking.
when_to_use_negative:
- The candidate methods output binary adjacency matrices rather than ranked edge lists,
  since no precision-recall ranking can be computed
- No gold-standard regulatory network (e.g., ESCAPE) exists for the genes in the predicted
  edge lists
- The methods were run on different datasets or gene sets, making ranked edge lists
  incomparable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Produces the ranked candidate regulator-target interaction list (via modified
    Borda aggregation over the GLG ensemble) that is scored by average precision and
    early precision
  repo: https://github.com/gitter-lab/SINGE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:815d26d41ff48689762beb3b1eb208b5a9f68c09039113edc694e9e86295cf87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# early-precision-ranking-evaluation

## Summary

Scores each method's ranked list of candidate regulator-target interactions against a gold-standard regulatory network (e.g., the ESCAPE gold standard) using average precision and early precision to benchmark GRN inference methods like SINGE against baselines such as GENIE3, SCODE, SINCERITIES, and Jump3.

## When to use

Use when comparing GRN inference methods that each output a ranked edge list over the same gene set and a gold-standard regulatory network is available to compute average precision and early precision for benchmarking.

## When NOT to use

- The candidate methods output binary adjacency matrices rather than ranked edge lists, since no precision-recall ranking can be computed
- No gold-standard regulatory network (e.g., ESCAPE) exists for the genes in the predicted edge lists
- The methods were run on different datasets or gene sets, making ranked edge lists incomparable

## Inputs

- Ranked edge list per method (e.g., SINGE_Ranked_Edge_List.txt with edges ranked by SINGE scores)
- Gold-standard regulatory network (ESCAPE gold standard) for the same gene set
- Per-method precision-recall results compiled into benchmark_precision_recall.csv

## Outputs

- Average precision per method against the gold-standard network
- Early precision per method against the gold-standard network
- Per-method precision-recall benchmark table (benchmark_precision_recall.csv)

## How to apply

Collect each method's predictions as a ranked list of candidate regulator-target interactions, with SINGE's ranking produced by modified Borda aggregation over the ensemble of partial GLG networks. Score every ranked edge list against the ESCAPE gold-standard regulatory network by computing average precision and early precision, so that performance at the top of the ranking (the edges an experimenter would actually validate) is captured alongside overall ranking quality. Compile the per-method precision-recall results into a table such as benchmark_precision_recall.csv and compare methods on the same pseudotime-ordered expression data. On the untuned retinoic acid mESC differentiation case study, the acceptance criterion is that SINGE's average precision and early precision reproduce the paper's reported benchmark values and exceed all four baselines (GENIE3, SCODE, SINCERITIES, Jump3).

## Related tools

- **SINGE** (Produces the ranked candidate regulator-target interaction list (via modified Borda aggregation over the GLG ensemble) that is scored by average precision and early precision) — https://github.com/gitter-lab/SINGE

## Evaluation signals

- Each method's predictions are aggregated into a ranked list of candidate regulator-target interactions before scoring
- Every ranked edge list is scored against the ESCAPE gold-standard network using both average precision and early precision
- SINGE's average precision and early precision reproduce the paper's reported benchmark values on the retinoic acid dataset
- SINGE outperforms all four baseline methods (GENIE3, SCODE, SINCERITIES, Jump3) on the retinoic acid mESC differentiation dataset
- Per-method precision-recall results are compiled into benchmark_precision_recall.csv

## Limitations

- Performance is measured against a gold standard whose coverage is imperfect, so rankings reflect agreement with the ESCAPE network rather than true regulatory accuracy
- SINGE's paper notes caveats about poor performance for individual regulators and uninformative pseudotimes, which would degrade ranking quality regardless of the evaluation metric
- For reproducibility, SINGE rankings depend on pseudo-random subsampling and zero-removal seeded by the hyperparameters including the date input; the same inputs and date must be supplied to reproduce a ranking

## Evidence

- [abstract] Aggregation and ranking of interactions: "It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes."
- [abstract] SINGE outperforms other contemporary algorithms on two mouse embryonic stem cell differentiation datasets: "In two mouse embryonic stem cell differentiation datasets, SINGE outperforms other contemporary algorithms."
- [readme] Ranked edge list output format: [evidence span withheld — anchor status: not_found]
- [readme] Reproducibility of ranked predictions: [evidence span withheld — anchor status: external_doc]
