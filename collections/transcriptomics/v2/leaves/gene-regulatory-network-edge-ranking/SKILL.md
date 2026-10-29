---
name: gene-regulatory-network-edge-ranking
description: Use when an overall precision-recall evaluation of a GRN inference method
  (e.g., SINGE Borda-aggregated edge ranks) looks acceptable but you need to check
  whether performance is driven or hidden by the prevalence of individual regulators
  in the gold standard.
when_to_use_negative:
- No gold-standard regulator-target annotation is available for the dataset, so per-regulator
  stratification of true edges cannot be computed
- You only need the headline precision-recall comparison across GRN methods, not a
  diagnosis of regulator-level performance heterogeneity
- The edge-rank output is not from an ensemble/aggregation stage (e.g., raw single-hyperparameter
  GLG output rather than the Borda-aggregated SINGE ranked edge list), since the evaluation
  targets the final aggregated ranking
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: SINGE
  role: Produces the aggregated Borda-ranked edge-rank predictions whose per-regulator
    rank distributions are evaluated
  repo: https://github.com/gitter-lab/SINGE
- name: MATLAB R2018a
  role: Environment for running the SINGE GLG + SINGE Aggregate workflow and equivalent
    MATLAB plotting of regulator-specific edge-rank boxplots
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:31aaf5615d18332b8d8e0bf7897587c2fca7065ae86bedcbdf79100e69347098
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-regulatory-network-edge-ranking

## Summary

A regulator-specific diagnostic evaluation of GRN inference edge rankings: SINGE-predicted ranks of gold-standard edges are stratified per regulator (12 ESCAPE regulators) and visualized as boxplots ordered by each regulator's gold-standard outgoing-edge degree, exposing regulators whose predictions are poor despite acceptable overall precision-recall performance.

## When to use

Use when an overall precision-recall evaluation of a GRN inference method (e.g., SINGE Borda-aggregated edge ranks) looks acceptable but you need to check whether performance is driven or hidden by the prevalence of individual regulators in the gold standard.

## When NOT to use

- No gold-standard regulator-target annotation is available for the dataset, so per-regulator stratification of true edges cannot be computed
- You only need the headline precision-recall comparison across GRN methods, not a diagnosis of regulator-level performance heterogeneity
- The edge-rank output is not from an ensemble/aggregation stage (e.g., raw single-hyperparameter GLG output rather than the Borda-aggregated SINGE ranked edge list), since the evaluation targets the final aggregated ranking

## Inputs

- SINGE aggregated (Borda-ranked) edge-rank prediction file, e.g. SINGE_Ranked_Edge_List.txt from the SINGE GLG + SINGE Aggregate workflow on the retinoic acid mESC dataset
- ESCAPE gold-standard regulatory network with 12 evaluated regulators (e.g., Dab2, Esrrb) and their documented target genes

## Outputs

- regulator_edge_rank_boxplots.png: 12 per-regulator boxplots of SINGE edge ranks, ordered by gold-standard outgoing-edge count, sharing a common rank axis
- Per-regulator outgoing-edge degree counts from the ESCAPE gold standard (sorting key)

## How to apply

Load the SINGE aggregated (Borda-ranked) edge-rank predictions, e.g. the SINGE_Ranked_Edge_List.txt output from the SINGE GLG + SINGE Aggregate workflow on the retinoic acid mESC dataset. Load the ESCAPE gold-standard regulatory network and extract the 12 evaluated regulators (e.g., Dab2, Esrrb) with their documented target genes. Count each regulator's outgoing gold-standard edges and sort the 12 regulators by this degree. For each regulator, collect the SINGE-predicted ranks of its gold-standard edges and render per-regulator boxplots of edge-rank distributions with a shared rank axis (matplotlib in Python or equivalent MATLAB plotting), so regulators with few but well-ranked outgoing edges can be distinguished from those with many poorly ranked edges. The rationale is that the paper's 'more detailed examination reveals caveats about poor performance for individual regulators', so a regulator-stratified view partially explains the aggregate precision-recall result and shows when it is misleading.

## Related tools

- **SINGE** (Produces the aggregated Borda-ranked edge-rank predictions whose per-regulator rank distributions are evaluated) — https://github.com/gitter-lab/SINGE
- **MATLAB R2018a** (Environment for running the SINGE GLG + SINGE Aggregate workflow and equivalent MATLAB plotting of regulator-specific edge-rank boxplots)

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- regulator_edge_rank_boxplots.png exists and contains exactly one boxplot per regulator (12 total)
- Boxplots are ordered by each regulator's gold-standard outgoing-edge count (ascending degree sort)
- All boxplots share a common edge-rank axis so distributions are directly comparable across regulators
- The figure visually exposes regulators whose edge-rank distributions are poor even though overall precision-recall performance is acceptable
- Ranks plotted for each regulator correspond to the gold-standard edges of that regulator in the SINGE ranked edge list (cross-check regulator-target pairs against SINGE_Ranked_Edge_List.txt)

## Limitations

- Regulator-specific evaluation only partially explains the overall precision-recall performance; it does not fully account for method differences
- Results depend on the composition and prevalence of regulators in the ESCAPE gold standard (only 12 evaluated regulators), so conclusions are gold-standard-dependent
- SINGE output reproducibility requires identical hyperparameters including the date seed; the master branch may be unstable, so versioned releases should be used

## Evidence

- [abstract] Aggregation produces the ranked list evaluated per regulator: "It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes."
- [abstract] Motivation for regulator-level caveats: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
- [other] Finding on misleading overall PR evaluation: "partially explains the overall precision-recall performance of the GRN methods and demonstrates that it can be somewhat misleading."
- [readme] SINGE output file containing the ranked edges analyzed by this skill: [evidence span withheld — anchor status: not_found]
- [readme] Standalone Docker invocation of the SINGE workflow producing the ranked edge list: [evidence span withheld — anchor status: external_doc]
