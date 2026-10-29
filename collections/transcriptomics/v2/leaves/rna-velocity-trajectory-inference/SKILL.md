---
name: rna-velocity-trajectory-inference
description: 'Use when a scRNA-seq AnnData object (with spliced/unspliced counts)
  needs trajectory inference: recovering parallel branches, within-population sub-flows,
  and cell pseudotime that direction-free TI methods (e.g., Slingshot) or velocity
  tools that lack path output (e.g., CellRank) cannot resolve.'
when_to_use_negative:
- The task requires only initial/terminal states or probabilistic fate maps without
  explicit trajectories — CellRank may suffice since CellPath is designed to output
  paths and pseudotime
- Only cluster-level trajectory resolution is needed and a compact backbone graph
  is acceptable — Directed-PAGA provides this without meta-cell construction
- No spliced/unspliced or velocity information is available and direction inference
  cannot be supported, since CellPath relies on RNA velocity to orient trajectories
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: CellPath
  role: 'Primary trajectory inference method: meta-cell construction, kNN graph, path
    finding, greedy trajectory selection, and first-order pseudotime reconstruction'
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the RNA velocity matrix used as input (stochastic or dynamical mode)
    during CellPath initialization when preprocess=False/True
- name: statsmodels
  role: Optional dependency for per-trajectory differentially expressed gene analysis
    via GAM likelihood-ratio test
- name: Slingshot
  role: Baseline TI method (with user-supplied root) for Kendall correlation comparison
    on simulated datasets
- name: CellRank
  role: Velocity-based baseline providing latent_time pseudotime for benchmarking
    against CellPath
- name: VeTra
  role: Velocity-based baseline run with clusternumber 3–4 and min_dist values 0.4/0.5/0.8
    for benchmarking
- name: CellPath_test
  role: Repository of test scripts and simulation scripts (VeloSim, dyngen) to reproduce
    manuscript results
  repo: https://github.com/PeterZZQ/CellPath_test
merged_aliases:
- single-cell-rna-velocity-trajectory-inference
merged_alias_records:
- alias: rna-velocity-trajectory-inference-benchmarking
  slug: rna-velocity-trajectory-inference-benchmarking
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:2b767ec7761412cb577bfd089e06eb4fdccfe0d4d91a1ead1081a9d05fd193fd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PeterZZQ/CellPath
  repo_url: https://github.com/PeterZZQ/CellPaths.git
  tools:
  - Angle
  - CellPath
  - CellPath_test
  - CellRank
  - Cytopath
  - PAGA
  - Slingshot
  - VeTra
  - numpy
  - pandas
  - scVelo
  - scipy
  - scvelo
  - statsmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# rna-velocity-trajectory-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer multiple high-resolution cell trajectories and per-cell pseudotime by integrating scRNA-seq count matrices with RNA velocity matrices using CellPath, which constructs meta-cells, builds kNN graphs, and greedily selects major trajectories without constraints on trajectory topology.

## When to use

Use when a scRNA-seq AnnData object (with spliced/unspliced counts) needs trajectory inference: recovering parallel branches, within-population sub-flows, and cell pseudotime that direction-free TI methods (e.g., Slingshot) or velocity tools that lack path output (e.g., CellRank) cannot resolve.

## When NOT to use

- The task requires only initial/terminal states or probabilistic fate maps without explicit trajectories — CellRank may suffice since CellPath is designed to output paths and pseudotime
- Only cluster-level trajectory resolution is needed and a compact backbone graph is acceptable — Directed-PAGA provides this without meta-cell construction
- No spliced/unspliced or velocity information is available and direction inference cannot be supported, since CellPath relies on RNA velocity to orient trajectories

## Inputs

- AnnData (.h5ad) object with scRNA-seq count matrix and spliced/unspliced layers for velocity estimation
- Pre-computed RNA velocity matrix from scVelo (stochastic or dynamical mode) or velocyto, if not using preprocess=True
- Ground-truth pseudotime and branching structure for simulated (VeloSim cycle-tree, dyngen binary-tree) benchmark datasets
- User-specified root cluster or root cell when benchmarking against Slingshot

## Outputs

- cellpath_obj.pseudo_order: per-cell pseudotime and branching/trajectory assignment table
- Set of major meta-cell-level trajectories (num_trajs paths, e.g., 7–20 paths depending on dataset)
- Per-path DE gene table from the statsmodels GAM likelihood-ratio test (e.g., path6_de_genes.csv, cellpath_hema_results.csv)
- Kendall rank correlation and normalized average entropy score benchmark tables (e.g., kendall_scores.csv, bifurcating_benchmark_metrics.csv)

## How to apply

Initialize with cp.CellPath(adata=adata, preprocess=True), letting scVelo compute RNA velocity (stochastic or dynamical mode) if the velocity matrix is not already stored in the AnnData object. Aggregate cells into meta-cells via cellpath.meta_cell_construction(), choosing flavor='leiden' for datasets with intricate trajectories (e.g., mouse hematopoiesis) or flavor='k-means' for simpler datasets (e.g., dentate gyrus, pancreatic endocrinogenesis). Run cellpath_obj.all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50, selecting num_trajs per dataset (e.g., 20 for hematopoiesis, 14 for dentate gyrus, 7 for endocrinogenesis) to build the symmetric kNN meta-cell graph, extract candidate paths, and greedily select major trajectories. Extract cell-level pseudotime and branching assignments from cellpath_obj.pseudo_order, then run per-trajectory DE analysis with the statsmodels-based GAM likelihood-ratio test to identify lineage markers (e.g., Elane/Mpo on the Neu-like monocyte path, Cd74 on the DC-like path, Camk2a/Adcy1 along the granule-cell path 6). For benchmarking, compute Kendall rank correlation against simulated ground-truth pseudotime and normalized average entropy of cell-to-trajectory assignments after the branching point to compare against CellRank, VeTra, Vdpt, and Slingshot.

## Related tools

- **CellPath** (Primary trajectory inference method: meta-cell construction, kNN graph, path finding, greedy trajectory selection, and first-order pseudotime reconstruction) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the RNA velocity matrix used as input (stochastic or dynamical mode) during CellPath initialization when preprocess=False/True)
- **statsmodels** (Optional dependency for per-trajectory differentially expressed gene analysis via GAM likelihood-ratio test)
- **Slingshot** (Baseline TI method (with user-supplied root) for Kendall correlation comparison on simulated datasets)
- **CellRank** (Velocity-based baseline providing latent_time pseudotime for benchmarking against CellPath)
- **VeTra** (Velocity-based baseline run with clusternumber 3–4 and min_dist values 0.4/0.5/0.8 for benchmarking)
- **CellPath_test** (Repository of test scripts and simulation scripts (VeloSim, dyngen) to reproduce manuscript results) — https://github.com/PeterZZQ/CellPath_test

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = True); cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, num_trajs = num_trajs, insertion = True, prop_insert = 0.50)
```

## Evaluation signals

- The number of greedy-selected trajectories matches num_trajs (e.g., 7 for endocrinogenesis, 14 for dentate gyrus, ~20 for hematopoiesis) and cellpath_obj.pseudo_order is populated with per-cell pseudotime and path assignments
- Per-path DE tables contain expected lineage markers: Elane and Mpo on hematopoiesis path 0 and Cd74 on path 8; Camk2a, Adcy1, Tmsb10 on dentate gyrus path 6; Pcsk2/Ero1lb/Cpe on beta-cell paths, Pax4 on path 1, Arx on path 4, and cell-cycle genes Kif23/Clspn/Aurkb/Spc24
- On VeloSim cycle-tree simulations, CellPath's Kendall rank correlation against ground-truth pseudotime exceeds CellRank, VeTra (min_dist 0.4/0.5/0.8), Vdpt, and Slingshot
- On the dyngen branching dataset, CellPath attains the highest normalized average entropy score of cell-to-trajectory assignments after the branching point, with Kendall correlations comparable to or above VeTra
- Reproduction scripts from PeterZZQ/CellPath_test run against the Dropbox-deposited data folder and benchmark outputs (kendall_scores.csv, bifurcating_benchmark_metrics.csv, figure_6G_S6G_kendall_boxplots.png) are produced and valid

## Limitations

- Performance of all RNA velocity-based methods, including CellPath, is limited by the accuracy of RNA velocity inference given its noise
- CellPath detects both global and local paths but cannot yet align locally detected paths into a global view
- Connecting points between paths are not yet implemented; this is noted as a future direction
- Comparison baselines differ in requirements: Slingshot needs a user-specified root cluster/cell, and VeTra results depend on clusternumber and min_dist settings

## Evidence
