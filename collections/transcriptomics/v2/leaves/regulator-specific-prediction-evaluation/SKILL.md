---
name: regulator-specific-prediction-evaluation
description: Use when overall precision-recall evaluation of a GRN method looks acceptable
  and you need to check whether performance is uneven across individual regulators,
  using a gold standard with per-regulator target sets.
when_to_use_negative:
- You only need an overall precision-recall comparison of GRN methods and are not
  concerned with per-regulator variability
- The gold standard does not identify individual regulators and their target genes,
  so outgoing edge counts per regulator cannot be computed
- You have raw expression data and have not yet run the full SINGE pipeline (SINGE
  GLG then SINGE Aggregate) to obtain ranked edges
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: SINGE
  role: Produces the aggregated Borda-ranked edge list whose per-regulator edge ranks
    are evaluated against the ESCAPE gold standard
  repo: https://github.com/gitter-lab/SINGE
- name: MATLAB R2018a
  role: MATLAB environment in which SINGE edge-rank outputs are generated and in which
    per-regulator boxplots can be rendered
  repo: https://github.com/gitter-lab/SINGE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:cdf189dd1a6803141047f92b82b293b25f76e5d219f8bd595b182e7524a56fb7
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# regulator-specific-prediction-evaluation

## Summary

A regulator-specific evaluation that disaggregates GRN inference precision-recall performance by individual regulator. It sorts evaluated regulators (e.g., the 12 ESCAPE gold-standard regulators) by their outgoing gold-standard edge count and plots per-regulator distributions of predicted edge ranks to expose poor per-regulator performance hidden by overall metrics.

## When to use

Use when overall precision-recall evaluation of a GRN method looks acceptable and you need to check whether performance is uneven across individual regulators, using a gold standard with per-regulator target sets.

## When NOT to use

- You only need an overall precision-recall comparison of GRN methods and are not concerned with per-regulator variability
- The gold standard does not identify individual regulators and their target genes, so outgoing edge counts per regulator cannot be computed
- You have raw expression data and have not yet run the full SINGE pipeline (SINGE GLG then SINGE Aggregate) to obtain ranked edges

## Inputs

- Aggregated SINGE edge-rank predictions (Borda-ranked SINGE_Ranked_Edge_List.txt) for the retinoic acid mESC dataset
- ESCAPE gold-standard regulatory network with 12 evaluated regulators (e.g., Dab2, Esrrb) and their documented target genes

## Outputs

- Per-regulator boxplots of SINGE-predicted edge ranks, one boxplot per regulator (12 total), ordered by gold-standard outgoing-edge count, saved as regulator_edge_rank_boxplots.png

## How to apply

Load the aggregated Borda-ranked edge-rank predictions (e.g., SINGE's SINGE_Ranked_Edge_List.txt produced by the SINGE GLG + SINGE Aggregate workflow on the retinoic acid mESC dataset). Load the gold-standard network — here the ESCAPE regulatory network — and extract the 12 evaluated regulators (e.g., Dab2, Esrrb) with their documented target genes. Count each regulator's outgoing gold-standard edges and sort the 12 regulators by this degree in ascending order. For each regulator, collect the predicted ranks of its gold-standard edges and render per-regulator boxplots of edge-rank distributions on a shared rank axis (matplotlib in Python or equivalent MATLAB plotting), saved as regulator_edge_rank_boxplots.png. The rationale is that a good aggregate precision-recall curve can be driven by a few well-predicted high-degree regulators while others perform poorly, so the boxplots reveal whether per-regulator edge-rank distributions are poor despite acceptable overall performance.

## Related tools

- **SINGE** (Produces the aggregated Borda-ranked edge list whose per-regulator edge ranks are evaluated against the ESCAPE gold standard) — https://github.com/gitter-lab/SINGE
- **MATLAB R2018a** (MATLAB environment in which SINGE edge-rank outputs are generated and in which per-regulator boxplots can be rendered) — https://github.com/gitter-lab/SINGE

## Evaluation signals

- regulator_edge_rank_boxplots.png exists and contains exactly one boxplot per regulator (12 total)
- Boxplots are ordered by the regulators' number of outgoing edges in the ESCAPE gold standard
- All boxplots share a common rank axis so edge-rank distributions are directly comparable across regulators
- The plot visually exposes regulators whose edge-rank distributions are poor despite acceptable overall precision-recall performance
- Each boxplot contains only the ranks of gold-standard edges outgoing from that specific regulator

## Limitations

- This evaluation only partially explains overall precision-recall performance — it does not fully account for it
- A favorable overall precision-recall evaluation can be somewhat misleading when a few regulators dominate the gold-standard edge counts
- The analysis is limited to the regulators included in the gold standard (12 in the ESCAPE retinoic acid mESC evaluation) and their documented targets

## Evidence

- [abstract] However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes.: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
- [other] Count each regulator's outgoing gold-standard edges and sort the 12 regulators in ascending order of this degree.: [evidence span withheld — anchor status: not_found]
- [other] render per-regulator boxplots of edge-rank distributions (matplotlib in Python or equivalent MATLAB plotting), sharing a common rank axis: [evidence span withheld — anchor status: not_found]
- [readme] SINGE_Ranked_Edge_List.txt - File with list of ranked edges according to their SINGE scores: [evidence span withheld — anchor status: not_found]
- [methods] the complete SINGE workflow, which includes the SINGE GLG and SINGE Aggregate modes.: "the complete SINGE workflow, which includes the SINGE GLG and SINGE Aggregate modes."
