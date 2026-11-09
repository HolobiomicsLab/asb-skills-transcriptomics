---
name: knn-graph-clustering-hematopoiesis-scrna-seq
description: Use when you need to test how sensitive an automated terminal-state/fate-detection
  method (VIA) is to graph parameters (KNN, ncomps) on a small annotated scRNA-seq
  hematopoiesis dataset, or to recover rare dendritic lineages and validate them by
  marker gene trends.
when_to_use_negative:
- Input is not an annotated single-cell dataset with a known root population — the
  F1 benchmarking requires ground-truth lineage labels and a valid early start cell.
- Trajectory topology is disconnected or cyclic and needs multiple roots — use via_wrapper_disconnected
  with preserve_disconnected=True instead of the standard wrapper.
- Dataset is large (10^5-10^6+ cells) where the goal is scalability rather than rare-lineage
  recovery on a small 5800-cell benchmark.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIA
  role: Trajectory inference method whose via_wrapper is swept over KNN/ncomps to
    detect terminal states and compute branch probabilities via lazy-teleporting MCMCs
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph; edge pruning controlled
    by jac_std_global=0.15
  repo: https://github.com/ShobiStassen/VIA
- name: scanpy
  role: Loads the h5ad CD34+ hematopoiesis dataset into an AnnData object with PCA
    stored in adata.obsm['X_pca']
- name: UMAP
  role: Provides the embedding used for single-cell-level visualization of the inferred
    trajectory
- name: pandas
  role: Writes the combined F1 and marker-correlation table to via_hematopoiesis_f1_correlations.csv
- name: dynverse/dyntoy
  role: Source of the 4-leaf multifurcating simulation used in VIA's starter tutorial
    for testing the same wrapper workflow
  repo: https://github.com/dynverse/dyntoy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:2398cd5ecf6d11cd36c4e619bdd08a379adf9c711954377f0ed87201c68d69f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - UMAP
  - VIA
  - dynverse/dyntoy
  - pandas
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# knn-graph-clustering-hematopoiesis-scrna-seq

## Summary

Benchmark robustness of VIA's automated terminal-state detection on the Setty 5800-cell human CD34+ hematopoiesis scRNA-seq dataset by sweeping KNN neighborhood size and number of PCs, scoring per-lineage F1 against ground-truth annotations (cDC, pDC, megakaryocyte, erythroid, monocyte, B-cell), and validating rare DC lineages with IRF8, CSF1R and CD123 marker-pseudotime trends.

## When to use

Use when you need to test how sensitive an automated terminal-state/fate-detection method (VIA) is to graph parameters (KNN, ncomps) on a small annotated scRNA-seq hematopoiesis dataset, or to recover rare dendritic lineages and validate them by marker gene trends.

## When NOT to use

- Input is not an annotated single-cell dataset with a known root population — the F1 benchmarking requires ground-truth lineage labels and a valid early start cell.
- Trajectory topology is disconnected or cyclic and needs multiple roots — use via_wrapper_disconnected with preserve_disconnected=True instead of the standard wrapper.
- Dataset is large (10^5-10^6+ cells) where the goal is scalability rather than rare-lineage recovery on a small 5800-cell benchmark.

## Inputs

- Setty 5800-cell (5780-cell) human CD34+ bone-marrow scRNA-seq dataset from VIA Datasets (anndata h5ad with PCA in adata.obsm['X_pca'])
- Ground-truth lineage annotations (cDC, pDC, megakaryocyte, erythroid, monocyte, B-cell)
- Marker genes: IRF8, CSF1R, CD123
- Root/start cell index for early hematopoietic state

## Outputs

- Per-lineage terminal-state F1 scores for every KNN/ncomps grid combination
- Marker-pseudotime correlations for IRF8, CSF1R, CD123 per lineage
- Combined results table via_hematopoiesis_f1_correlations.csv
- VIA trajectory outputs: pseudotime, terminal states, single-cell branch probabilities (500 MCMCs per terminal state)

## How to apply

Load the Setty 5800-cell CD34+ bone-marrow scRNA-seq dataset and its ground-truth lineage annotations from the VIA Datasets repository into an AnnData object with PCA precomputed in adata.obsm['X_pca']. Sweep via_wrapper over KNN sizes centered on the HNSWlib default knn=30 and principal-component counts (e.g. 10-20 as used in VIA's wrapper examples), keeping jac_std_global=0.15 for PARC edge pruning and setting root_user to an early hematopoietic start cell. For each grid point, detect terminal states with pseudotime_threshold_TS=30 (percentile threshold) and estimate branch probabilities with num_sim_branch_probability=500 lazy-teleporting MCMC simulations per terminal state. Match each predicted terminal cluster to the annotated fates and compute per-lineage F1 scores. Extract IRF8, CSF1R and CD123 trends along pseudotime with VIA's gene-expression plotting and compute marker-pseudotime correlations; rationale is that the paper shows VIA detects rare dendritic subtypes and behaves predictably under parameter variation ('stress tests' of topology and cell fates).

## Related tools

- **VIA** (Trajectory inference method whose via_wrapper is swept over KNN/ncomps to detect terminal states and compute branch probabilities via lazy-teleporting MCMCs) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm underlying VIA's cluster-graph; edge pruning controlled by jac_std_global=0.15) — https://github.com/ShobiStassen/VIA
- **scanpy** (Loads the h5ad CD34+ hematopoiesis dataset into an AnnData object with PCA stored in adata.obsm['X_pca'])
- **UMAP** (Provides the embedding used for single-cell-level visualization of the inferred trajectory)
- **pandas** (Writes the combined F1 and marker-correlation table to via_hematopoiesis_f1_correlations.csv)
- **dynverse/dyntoy** (Source of the 4-leaf multifurcating simulation used in VIA's starter tutorial for testing the same wrapper workflow) — https://github.com/dynverse/dyntoy

## Examples

```
via.via_wrapper(adata, true_label, adata.obsm['X_umap'], knn=30, ncomps=15, jac_std_global=0.15, root=[42], pseudotime_threshold_TS=30, num_sim_branch_probability=500)
```

## Evaluation signals

- Output table covers all six lineages (cDC, pDC, megakaryocyte, erythroid, monocyte, B-cell) and all three markers (IRF8, CSF1R, CD123) for every KNN/ncomps grid point.
- Per-lineage F1 scores remain stable across the KNN sweep centered on knn=30 and ncomps 10-20, reproducing the paper's lineage-recovery pattern.
- Marker trends match the paper: IRF8 is elevated in both DC lineages, CSF1R is specific to cDC, and CD123 remains elevated for pDCs.
- Terminal states are detected consistently at pseudotime_threshold_TS=30 with branch probabilities from num_sim_branch_probability=500 MCMCs per terminal state.
- Predicted terminal clusters can be matched one-to-one to the annotated fates in the Setty ground-truth labels.

## Limitations

- No accession identifiers are given for the hematopoiesis dataset in the paper; data must be obtained from the VIA Datasets folder/Drive links.
- Specific parameter ranges used in the paper's 'stress tests' are not reported in the discussion, so the grid must be chosen from wrapper-example conventions (e.g. knn=10-30, ncomps 10-20).
- No software version numbers are stated for VIA or compared methods, which may affect exact reproducibility of F1-scores.

## Evidence

- [methods] wrapper signature: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset=''"
- [methods] terminal state threshold: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [discussion] rare dendritic lineage detection: "VIA distinguished between dendritic subtypes in an scRNA-seq hematopoiesis dataset; identified the rare delta cell islet in pancreatic development"
- [discussion] stress-test motivation: "we conducted a series of ‘stress tests’ of the inferred topology and cell fates on both simulated and biological data"
- [readme] hematopoiesis tutorial dataset: [evidence span withheld — anchor status: not_found]
