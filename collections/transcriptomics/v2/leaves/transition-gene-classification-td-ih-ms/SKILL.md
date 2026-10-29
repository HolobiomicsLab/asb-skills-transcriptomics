---
name: transition-gene-classification-td-ih-ms
description: Use when a stable-attractor-to-attractor transition has been defined
  (e.g., states i and j from MuTrans dynamical analysis) and you need to identify
  which genes drive the transition, mark transient/hybrid cells, or label meta-stable
  cells.
when_to_use_negative:
- When attractor basins and the soft membership matrix have not yet been established
  (no prior dynamical_analysis on the dataset)
- When the analysis goal is pseudo-time ordering or clustering rather than gene classification
  along a specific two-state transition
- When the dataset is too small for high-resolution transition-cell identification
  (the article notes special care is needed for small datasets like myelopoiesis)
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MuTrans
  role: 'Provides the Transcendental procedure and GeneAnalysis function (Matlab:
    GeneAnalysis(i,j,Output,par); Python: pyMuTrans) that compute TCS and classify
    TD/IH/MS genes between states i and j'
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Matlab Curve Fitting Toolbox (smooth)
  role: Smoothing of gene expression matrix display by the Transcendental step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:ef76af19c66d7bed237054af650fe3b719b40beec8107e8076249d0149f3ff97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - Matlab Curve Fitting Toolbox (smooth)
  - MuTrans
  - PyEMMA
  - Scanpy
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# transition-gene-classification-td-ih-ms

## Summary

Classifies genes along a cell-fate transition (e.g., Pro-B to Pre-B) into transition-driver (TD), intermediate/hybrid-state (IH), and meta-stable (MS) categories using MuTrans' Transcendental/GeneAnalysis procedure, which couples transition cell scores (TCS) with gene expression dynamics across the reconstructed dynamical manifold.

## When to use

Use when a stable-attractor-to-attractor transition has been defined (e.g., states i and j from MuTrans dynamical analysis) and you need to identify which genes drive the transition, mark transient/hybrid cells, or label meta-stable cells.

## When NOT to use

- When attractor basins and the soft membership matrix have not yet been established (no prior dynamical_analysis on the dataset)
- When the analysis goal is pseudo-time ordering or clustering rather than gene classification along a specific two-state transition
- When the dataset is too small for high-resolution transition-cell identification (the article notes special care is needed for small datasets like myelopoiesis)

## Inputs

- AnnData object (e.g., .h5ad) containing the pre-processed single-cell gene expression matrix (N_cells x N_genes) after MuTrans dynamical_analysis
- Attractor/state labels or indices for the source state i and target state j (e.g., Pro-B and Pre-B basins)
- MuTrans Output/par parameter object from DynamicalAnalysis

## Outputs

- Gene classification into TD, IH, and MS categories for the specified transition i->j
- Per-cell transition cell score (TCS) values (e.g., transcendental_tcs_proB_preB.csv)
- Gene expression trajectories along the transition path, optionally smoothed for display

## How to apply

First run pm.dynamical_analysis(adata, par) with K_cluster set from the Eigen-Peak Index (EPI) eigen-gaps or prior annotations so attractor basins and the soft membership matrix are available. Then call the Transcendental procedure (GeneAnalysis from state i to state j) on the reconstructed dynamical manifold to compute per-cell transition cell score tau_ij = rho_i*/(rho_i* + rho_j*) and the associated gene trajectories along the transition path. Genes whose expression changes drive the barrier crossing are classified as TD genes; genes peaking in intermediate-TCS transition cells are IH genes; genes stable within attractor basins are MS genes. For visualization of gene expression along the manifold in the Matlab implementation, the Curve Fitting toolbox 'smooth' function is required. The classification is meaningful only relative to the specific ordered transition (i,j) analyzed.

## Related tools

- **MuTrans** (Provides the Transcendental procedure and GeneAnalysis function (Matlab: GeneAnalysis(i,j,Output,par); Python: pyMuTrans) that compute TCS and classify TD/IH/MS genes between states i and j) — https://github.com/cliffzhou92/MuTrans-release
- **Matlab Curve Fitting Toolbox (smooth)** (Smoothing of gene expression matrix display by the Transcendental step)

## Examples

```
Genes = GeneAnalysis(i,j,Output,par)
```

## Evaluation signals

- The TCS table file exists with one score per relevant cell and each tau_ij value lies in [0,1] since tau_ij = rho_i*/(rho_i* + rho_j*)
- TD genes show expression changes concentrated along the saddle path between attractors, while MS genes remain stable within basins
- Attractor assignments for states i and j (e.g., Pro-B, Pre-B) are consistent with prior cell-type annotations (output_matches_reference)
- Fraction of intermediate-TCS transition cells at the basin boundary reproduces the reported presence of transition cells within an expert-tolerated range

## Limitations

- Classification depends on a sufficiently large number of cells; small datasets (e.g., myelopoiesis) need special care to confirm transition-cell and gene analyses
- Results are transition-specific: TD/IH/MS labels apply only to the ordered pair of states (i,j) analyzed and may change for other transitions
- Non-stationary effects such as cell cycle or proliferation dynamics are not modeled, which can confound gene classification

## Evidence

- [intro] TD/IH/MS gene categories: "we are able to identify critical genes that are transition drivers (TD genes), mark the intermediate/hybrid states (IH genes) or meta-stable cells (MS genes)"
- [methods] Transcendental procedure: "MuTrans performs the Transcendental (transition cells and relevant analysis) procedure on each transition process to identify the transition cells from the stable cells"
- [readme] GeneAnalysis call: "%% Transition cell and gene analysis from state transition i to j."
- [methods] TCS definition: [evidence span withheld — anchor status: not_found]
- [abstract] downstream gene identification: "Downstream analysis identifies distinct genes that mark the transient states or drive the transitions."
