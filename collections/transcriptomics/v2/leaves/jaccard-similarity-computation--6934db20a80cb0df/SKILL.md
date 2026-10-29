---
name: jaccard-similarity-computation
description: Use when comparing which interactions are shared versus distinct across
  CCC inference methods, resources, or datasets — e.g., to measure how much top-ranked
  predictions from CellChat vs NATMI (or CellChatDB vs Ramilowski) overlap.
when_to_use_negative:
- When a gold-standard comparison of prediction accuracy is needed — no gold standard
  for CCC exists, so Jaccard overlap only measures agreement, not correctness
- When methods infer intracellular processes (NicheNet, Cytotalk, SoptSC) that are
  not directly comparable to ligand-receptor methods
- When interactions should not be decomplexified (comparing complex-level rather than
  subunit-level resource contents)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3170
tools:
- name: LIANA
  role: Runs the 7 CCC methods decoupled from their resources, ranks interactions
    by magnitude rank, and provides select_resource to load the 16 ligand-receptor
    resources
  repo: https://github.com/saezlab/liana
- name: OmnipathR
  role: Backs the loading of the 16 ligand-receptor resources used for the overlap
    statistics
  repo: https://github.com/saezlab/OmnipathR
- name: CellChat
  role: One of the 7 methods (probability score) whose top-1000 predictions are compared
    via Jaccard indices
- name: NATMI
  role: One of the 7 methods (mean-expression edge weight) whose top-1000 predictions
    are compared via Jaccard indices
  repo: https://github.com/saezlab/NATMI
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:dab61c19992fc83c507f24d9721824b312839052791f51dfd6eaa337206f8523
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - Connectome
  - Cytotalk
  - LIANA
  - NATMI
  - OmnipathR
  - SingleCellSignalR
  - iTALK
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Jaccard similarity computation

## Summary

Quantifies overlap between sets of cell-cell communication interactions (or resource contents) by building presence-absence matrices and computing pairwise Jaccard indices, used in Dimitrov et al. 2022 to show that both CCC methods and prior-knowledge resources strongly influence predictions.

## When to use

Use when comparing which interactions are shared versus distinct across CCC inference methods, resources, or datasets — e.g., to measure how much top-ranked predictions from CellChat vs NATMI (or CellChatDB vs Ramilowski) overlap.

## When NOT to use

- When a gold-standard comparison of prediction accuracy is needed — no gold standard for CCC exists, so Jaccard overlap only measures agreement, not correctness
- When methods infer intracellular processes (NicheNet, Cytotalk, SoptSC) that are not directly comparable to ligand-receptor methods
- When interactions should not be decomplexified (comparing complex-level rather than subunit-level resource contents)

## Inputs

- Ranked CCC interaction predictions (ligand-receptor pairs with magnitude rank scores) per method-resource-dataset combination, e.g., top 1000 interactions from LIANA runs of 7 methods x 16 resources on 6 scRNA-seq datasets
- Ligand-receptor resources loaded via liana::select_resource (backed by OmnipathR), decomplexified to subunit level
- Presence-absence matrices of predicted interactions per method and per resource

## Outputs

- Pairwise Jaccard similarity matrix across methods and resources (Fig 5-style heatmap)
- Median Jaccard values across datasets (expected ~0.080 for different methods, ~0.119 for different resources), written to median_jaccard_fig5.csv
- Per-resource summary statistics of unique interactions, unique receivers, and unique transmitters (Supplementary Table 1, supplementary_table_1.csv)

## How to apply

First, rank interactions per method-resource-dataset combination by each method's magnitude rank (e.g., CellChat probability, CellPhoneDBv2 truncated mean, NATMI mean-expression edge weight, SingleCellSignalR LRscore) and retain the top 1000 highest-ranked interactions. Build a presence-absence matrix indicating which interactions are predicted by each method and each resource. Compute pairwise Jaccard indices (intersection over union of interaction sets) from these matrices and take the median across the six scRNA-seq datasets separately for method-method and resource-resource comparisons. For resource overlap, decomplexify heteromeric complexes to subunit level before computing per-resource unique interactions/receivers/transmitters and the pairwise Jaccard matrix across all 16 resources. The paper expects consistently low overlap: median Jaccard ~0.080 when varying methods and ~0.119 when varying resources.

## Related tools

- **LIANA** (Runs the 7 CCC methods decoupled from their resources, ranks interactions by magnitude rank, and provides select_resource to load the 16 ligand-receptor resources) — https://github.com/saezlab/liana
- **OmnipathR** (Backs the loading of the 16 ligand-receptor resources used for the overlap statistics) — https://github.com/saezlab/OmnipathR
- **CellChat** (One of the 7 methods (probability score) whose top-1000 predictions are compared via Jaccard indices)
- **NATMI** (One of the 7 methods (mean-expression edge weight) whose top-1000 predictions are compared via Jaccard indices) — https://github.com/saezlab/NATMI

## Examples

```
R: library(liana); res <- select_resource('CellChatDB'); jaccard <- dist(presence_absence_matrix, method='binary')  # Jaccard = 1 - binary dist
```

## Evaluation signals

- Median pairwise Jaccard across datasets should be low: ~0.080 for method-method comparisons and ~0.119 for resource-resource comparisons (top-1000 interactions)
- Per-resource uniqueness should average ~6.4% unique receivers, ~5.7% unique transmitters, and ~10.4% unique interactions across the 16 resources
- Jaccard matrix must be symmetric with diagonal = 1.0 and all off-diagonal values in [0, 1]
- Presence-absence matrices must contain exactly 1000 positive entries per method-resource-dataset combination (top-1000 retained)
- Complex-mediated interactions should be decomplexified to constituent subunits before resource-level overlap computation

## Limitations

- Low Jaccard overlap does not imply one method or resource is best — no gold standard for CCC exists and biological ground truth is largely unknown
- Jaccard on top-ranked predictions cannot distinguish simple co-expression from actual CCC events
- Overlap statistics depend on the ranking cutoff (top 1000) and on decomplexification choices for heteromeric complexes

## Evidence

- [results] Consistently low overlap when varying methods or resources: [evidence span withheld — anchor status: not_found]
- [results] Limited resource uniqueness: [evidence span withheld — anchor status: not_found]
- [abstract] Method-resource influence: "We then examine all possible combinations of methods and resources and show that both strongly influence the predicted intercellular interactions."
- [abstract] Few unique interactions among resources: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [intro] LIANA as framework: "All results were generated using LIANA—a LIgand-receptor ANalysis frAmework (Fig. 1; available at https://github.com/saezlab/liana)"
