---
name: rank-distribution-boxplot-visualization
description: Use when evaluating GRN inference on the SINGE retinoic acid mESC dataset
  and the overall precision-recall evaluation needs to be broken down per regulator
  against the ESCAPE gold standard.
when_to_use_negative:
- The goal is an overall method-level precision-recall comparison across GRN methods
  rather than a regulator-level diagnosis of a single method's predictions
- The input predictions are not aggregated ranked edge lists (no Borda rank or edge-rank
  ordering) from the SINGE GLG + SINGE Aggregate workflow
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINGE
  role: Produces the Borda-aggregated ranked edge list (SINGE GLG + SINGE Aggregate
    workflow) whose per-edge ranks are visualized per regulator
  repo: https://github.com/gitter-lab/SINGE
- name: MATLAB R2018a
  role: Runtime for running the SINGE workflow that generates the ranked edge list;
    equivalent MATLAB plotting can render the per-regulator boxplots
  repo: https://github.com/gitter-lab/SINGE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:0668e8b68f2f42c6ad8ebbeda19e1e2927ae76861ec2db14b497e3599c15ea45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rank-distribution-boxplot-visualization

## Summary

A regulator-stratified diagnostic that renders per-regulator boxplots of predicted edge ranks (from SINGE Borda-aggregated output) so that poor performance for individual regulators is exposed even when the overall precision-recall curve of the GRN method looks acceptable.

## When to use

Use when evaluating GRN inference on the SINGE retinoic acid mESC dataset and the overall precision-recall evaluation needs to be broken down per regulator against the ESCAPE gold standard.

## When NOT to use

- The goal is an overall method-level precision-recall comparison across GRN methods rather than a regulator-level diagnosis of a single method's predictions
- The input predictions are not aggregated ranked edge lists (no Borda rank or edge-rank ordering) from the SINGE GLG + SINGE Aggregate workflow

## Inputs

- SINGE aggregated (Borda-ranked) edge-rank predictions for the retinoic acid mESC dataset (e.g., SINGE_Ranked_Edge_List.txt with ranked edges according to SINGE scores)
- ESCAPE gold-standard regulatory network with the 12 evaluated regulators (e.g., Dab2, Esrrb) and their documented target genes

## Outputs

- regulator_edge_rank_boxplots.png with one boxplot per regulator (12 total) ordered by gold-standard outgoing-edge count on a common rank axis

## How to apply

Load the SINGE aggregated (Borda-ranked) edge-rank predictions produced by the SINGE GLG + SINGE Aggregate workflow, together with the SINGE_Ranked_Edge_List.txt output. Load the ESCAPE gold-standard regulatory network and extract the 12 evaluated regulators (e.g., Dab2, Esrrb) with their documented target genes. Count each regulator's outgoing gold-standard edges and sort the 12 regulators by this degree. For each regulator, collect the SINGE-predicted ranks of its gold-standard edges and render per-regulator boxplots of the edge-rank distributions (matplotlib in Python or equivalent MATLAB plotting) on a shared rank axis so regulators are comparable. The rationale, per the paper, is that aggregate precision-recall performance 'can be somewhat misleading': a more detailed examination reveals caveats about poor performance for individual regulators.

## Related tools

- **SINGE** (Produces the Borda-aggregated ranked edge list (SINGE GLG + SINGE Aggregate workflow) whose per-edge ranks are visualized per regulator) — https://github.com/gitter-lab/SINGE
- **MATLAB R2018a** (Runtime for running the SINGE workflow that generates the ranked edge list; equivalent MATLAB plotting can render the per-regulator boxplots) — https://github.com/gitter-lab/SINGE

## Examples

```
E = readtable('SINGE_Ranked_Edge_List.txt'); boxplot(reshape(E.Rank, [], 12), 'Labels', regulators)
```

## Evaluation signals

- regulator_edge_rank_boxplots.png exists and is a valid image
- The figure contains exactly one boxplot per regulator — 12 total, matching the number of ESCAPE-evaluated regulators
- The 12 boxplots are ordered by each regulator's number of outgoing edges in the ESCAPE gold standard
- All boxplots share a common edge-rank axis so rank distributions are directly comparable across regulators
- Regulators whose gold-standard edge-rank distributions are poor are visually identifiable despite acceptable overall precision-recall performance

## Limitations

- The regulator-specific evaluation only partially explains the overall precision-recall performance of the GRN methods; it demonstrates that overall evaluation can be somewhat misleading rather than fully accounting for it
- The diagnosis is limited to the 12 regulators evaluated in the ESCAPE gold standard; unranked/unevaluated regulators are not covered
- SINGE results depend on hyperparameters, subsampling, and the date-based random seed; use a versioned release for stable, reproducible ranked edge lists

## Evidence

- [abstract] caveats about individual-regulator performance: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
- [methods] Borda aggregation produces the ranked candidate interactions that are visualized: "It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes."
- [readme] Ranked edge list output file consumed by the boxplot workflow: [evidence span withheld — anchor status: not_found]
- [readme] Use a versioned release for reproducible aggregated predictions: [evidence span withheld — anchor status: external_doc]
