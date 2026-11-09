---
name: terminal-state-lineage-f1-evaluation
description: Use when benchmarking how robustly a trajectory-inference method recovers
  rare terminal cell fates (e.g., mDC, pDC) under varying KNN/ncomps parameters, and
  when validating detected lineages by marker gene trends.
when_to_use_negative:
- The goal is simulating synthetic trajectories rather than evaluating terminal-state
  detection on a real annotated dataset — use dynverse/dyntoy to generate toy data
  instead.
- The dataset has disconnected or cyclic topology requiring via_wrapper_disconnected
  with multiple roots and preserve_disconnected=True rather than a single-root hematopoiesis
  tree.
- Input is raw count data without precomputed PCA — the workflow assumes PCA is already
  stored in adata.obsm['X_pca'].
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: 'Trajectory inference: pseudotime computation, automated terminal-state detection
    (pseudotime_threshold_TS=30), and lineage probability estimation via lazy-teleporting
    MCMC; run through via_wrapper across the KNN/ncomps grid'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Underlying clustering algorithm whose graph edge pruning (jac_std_global=0.15,
    edgepruning_clustering_resolution) defines the cluster-graph used for terminal-state
    detection
- name: scanpy
  role: Loading the CD34+ dataset as AnnData with PCA stored in adata.obsm['X_pca']
- name: pandas
  role: Assembling and writing the terminal-state F1 scores and marker-pseudotime
    correlations to via_hematopoiesis_f1_correlations.csv
- name: UMAP
  role: Provides the single-cell embedding used for visualization of the inferred
    trajectory
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:086d978359e8c105e197fb45aadff694e73abbc9962be1c71ee5564a9ff79349
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - UMAP
  - VIA
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# terminal-state-lineage-f1-evaluation

## Summary

A robustness-and-validation skill for VIA-based terminal state detection on the 5780-cell human CD34+ hematopoiesis scRNA-seq dataset: sweep KNN and principal-component parameters, score per-lineage F1 recovery of rare dendritic, megakaryocyte, erythroid, monocyte and B-cell fates, and validate rare lineages with marker-gene pseudotime trends (IRF8, CSF1R, CD123).

## When to use

Use when benchmarking how robustly a trajectory-inference method recovers rare terminal cell fates (e.g., mDC, pDC) under varying KNN/ncomps parameters, and when validating detected lineages by marker gene trends.

## When NOT to use

- The goal is simulating synthetic trajectories rather than evaluating terminal-state detection on a real annotated dataset — use dynverse/dyntoy to generate toy data instead.
- The dataset has disconnected or cyclic topology requiring via_wrapper_disconnected with multiple roots and preserve_disconnected=True rather than a single-root hematopoiesis tree.
- Input is raw count data without precomputed PCA — the workflow assumes PCA is already stored in adata.obsm['X_pca'].

## Inputs

- AnnData object of the 5780-cell Setty CD34+ human hematopoiesis scRNA-seq dataset with PCA in adata.obsm['X_pca']
- Ground-truth lineage annotations for cDC, pDC, megakaryocyte, erythroid, monocyte, and B-cell fates
- Gene expression matrix for marker genes IRF8, CSF1R, CD123
- A user-specified early hematopoietic start cell (root_user)

## Outputs

- Per-lineage terminal-state F1 scores for every KNN/ncomps grid configuration
- Marker-pseudotime correlation values for IRF8, CSF1R, and CD123 per lineage
- Combined results table via_hematopoiesis_f1_correlations.csv
- Gene-expression-trend plots along pseudotime from VIA's get_gene_expression plotting function

## How to apply

Load the Setty 5800-cell CD34+ bone-marrow scRNA-seq AnnData with PCA precomputed in adata.obsm['X_pca'] and ground-truth lineage annotations from the VIA Datasets repository. Sweep the via_wrapper over KNN neighborhood sizes around the default knn=30 and principal-component counts (e.g., 10-20), keeping jac_std_global=0.15 for PARC edge pruning and supplying root_user with an early hematopoietic start cell. For each grid point detect terminal states with pseudotime_threshold_TS=30 (percentile threshold) and run num_sim_branch_probability=500 lazy-teleporting MCMCs per terminal state to estimate branch probabilities. Match each predicted terminal cluster to the annotated fates (cDC, pDC, megakaryocyte, erythroid, monocyte, B-cell), compute per-lineage F1 scores, then extract IRF8, CSF1R and CD123 expression trends along pseudotime and correlate them with lineage assignment to confirm the rare DC lineages (IRF8 elevated in both DC lineages, CSF1R cDC-specific, CD123 elevated for pDCs).

## Related tools

- **VIA** (Trajectory inference: pseudotime computation, automated terminal-state detection (pseudotime_threshold_TS=30), and lineage probability estimation via lazy-teleporting MCMC; run through via_wrapper across the KNN/ncomps grid) — https://github.com/ShobiStassen/VIA
- **PARC** (Underlying clustering algorithm whose graph edge pruning (jac_std_global=0.15, edgepruning_clustering_resolution) defines the cluster-graph used for terminal-state detection)
- **scanpy** (Loading the CD34+ dataset as AnnData with PCA stored in adata.obsm['X_pca'])
- **pandas** (Assembling and writing the terminal-state F1 scores and marker-pseudotime correlations to via_hematopoiesis_f1_correlations.csv)
- **UMAP** (Provides the single-cell embedding used for visualization of the inferred trajectory)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=30, ncomps=20, jac_std_global=0.15, root_user=[42], pseudotime_threshold_TS=30, num_sim_branch_probability=500)
```

## Evaluation signals

- The output table covers all six lineages (mDC/cDC, pDC, megakaryocyte, erythroid, monocyte, B-cell) and all three markers (IRF8, CSF1R, CD123) for every KNN/ncomps grid point
- Terminal-state F1 scores remain stable across the swept KNN (centered on knn=30) and ncomps (10-20) values, reproducing the paper's reported parameter robustness
- IRF8 expression is elevated along both dendritic lineages, CSF1R is specific to cDC, and CD123 remains elevated for pDCs in the pseudotime trend plots
- Predicted terminal clusters match the annotated fates after the matching step at each grid configuration
- Each MCMC branch-probability estimate used the specified num_sim_branch_probability=500 simulations per terminal state

## Limitations

- Performance depends on user-set parameters; VIA's robustness claim is specific to varying KNN neighbors and number of PCs, with jac_std_global=0.15 and pseudotime_threshold_TS=30 held fixed.
- No accession identifiers are provided in the discussion for the hematopoiesis validation dataset; the data must be sourced from the VIA Datasets repository or linked Google Drive.
- Terminal-state detection uses a percentile threshold (default 30) and clusters smaller than small_pop=10 cells may be merged, which can affect recovery of very rare fates.
