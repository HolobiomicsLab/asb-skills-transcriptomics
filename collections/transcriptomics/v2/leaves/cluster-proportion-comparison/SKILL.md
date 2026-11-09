---
name: cluster-proportion-comparison
description: Use when synthetic scRNA-seq data generated from pilot sub-samples (e.g.,
  PBMC4k or Segerstolpe) must be checked for whether cluster/cell-type proportions
  match ground-truth proportions, and how this improves with pilot size.
when_to_use_negative:
- When evaluating per-gene distribution fidelity (medians, interquartile ranges, coefficients
  of variation, zero-count proportions) rather than cluster composition — use the
  MAD per gene skill instead
- When neither ground-truth cluster labels nor a nearest-neighbor mapping of synthetic
  cells in UMAP space is available
- When assessing batch correction, which was explicitly outside the paper's scope
  (all models examined without batch correction capabilities)
tools:
- name: scDBM
  role: Deep Boltzmann machine model whose synthetic samples' cluster proportions
    are compared to ground truth
  repo: https://github.com/MTreppner/scDBM.jl
- name: scVI
  role: Variational inference model sampled from prior and posterior distributions;
    its synthetic cluster proportions are evaluated against original data
  repo: https://github.com/scverse/scvi-tools
- name: Seurat
  role: Provides the ground-truth clustering of the original data on which per-cluster
    cell counts are based
- name: UMAP
  role: Embedding space in which synthetic observations are mapped to nearest original
    observations to acquire cluster labels
- name: BoltzmannMachines.jl
  role: Julia implementation underlying the scDBM sampling used in the proportion
    analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:61794075df8472da41c90e2fac7457dcfedf55e73353e7d270ad2d8b078a3589
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MTreppner/scDBM.jl
  tools:
  - BoltzmannMachines.jl
  - Seurat
  - UMAP
  - scDBM
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cluster-proportion-comparison

## Summary

Quantifies how faithfully deep generative models (scDBM, scVI prior/posterior) reproduce the proportion of cells per Seurat cluster by computing mean absolute differences in per-cluster cell counts between synthetic and original datasets across pilot sizes (384–2304 cells), as reported in Table 1 of the scDBM paper.

## When to use

Use when synthetic scRNA-seq data generated from pilot sub-samples (e.g., PBMC4k or Segerstolpe) must be checked for whether cluster/cell-type proportions match ground-truth proportions, and how this improves with pilot size.

## When NOT to use

- When evaluating per-gene distribution fidelity (medians, interquartile ranges, coefficients of variation, zero-count proportions) rather than cluster composition — use the MAD per gene skill instead
- When neither ground-truth cluster labels nor a nearest-neighbor mapping of synthetic cells in UMAP space is available
- When assessing batch correction, which was explicitly outside the paper's scope (all models examined without batch correction capabilities)

## Inputs

- Ground-truth cluster labels from Seurat clustering on UMAP of original datasets (PBMC4k, Segerstolpe)
- Cluster labels for synthetic datasets from scDBM (Gibbs sampling) and scVI (prior and posterior sampling), assigned by nearest-neighbor mapping in UMAP space
- Pilot-size grid (384, 768, 1152, 1536, 1920, 2304 cells) with 30 sub-sampling repetitions per size

## Outputs

- Mean absolute difference in per-cluster cell counts per model, dataset, and pilot size (Table 1 structure)
- Convergence trend of cluster proportions toward ground truth as pilot size increases

## How to apply

Load ground-truth cluster labels from Seurat clustering on the UMAP representation of each original dataset (PBMC4k, Segerstolpe), and cluster labels for each synthetic dataset obtained by mapping synthetic observations to their nearest original observation via Euclidean distance in UMAP space. For each model (scDBM, scVI prior, scVI posterior), dataset, and pilot size (384, 768, 1152, 1536, 1920, 2304 cells; 30 repetitions per size), tally the number of cells per cluster in original and synthetic data and compute absolute differences across all clusters. Average the absolute differences over the 30 repetitions to yield one mean absolute difference per model-dataset-pilot-size combination, matching the structure of Table 1. The rationale is that growing cluster-proportion fidelity with pilot size indicates the generative model has learned enough of the composition to support experiment design, whereas systematic deviations reveal overrepresentation of abundant cell types and missing rare ones.

## Related tools

- **scDBM** (Deep Boltzmann machine model whose synthetic samples' cluster proportions are compared to ground truth) — https://github.com/MTreppner/scDBM.jl
- **scVI** (Variational inference model sampled from prior and posterior distributions; its synthetic cluster proportions are evaluated against original data) — https://github.com/scverse/scvi-tools
- **Seurat** (Provides the ground-truth clustering of the original data on which per-cluster cell counts are based)
- **UMAP** (Embedding space in which synthetic observations are mapped to nearest original observations to acquire cluster labels)
- **BoltzmannMachines.jl** (Julia implementation underlying the scDBM sampling used in the proportion analysis)

## Examples

```
import Pluto; Pluto.run()  # launch Pluto, then open scDBM_notebook.jl; reproduce Table 1 via Rscript with table_1.Rmd from MTreppner/scDBM-paper
```

## Evaluation signals

- Output table contains exactly one row per model (scDBM, scVI prior, scVI posterior) × dataset (PBMC4k, Segerstolpe) × pilot size (384–2304) combination
- All absolute differences are non-negative by construction
- Mean absolute differences decrease (or near-monotonically decrease) with increasing pilot size, consistent with proportions converging toward ground truth
- Per-cluster counts in synthetic datasets are tallied against original counts across all clusters before averaging over the 30 repetitions
- Comparison structure matches the paper's Table 1

## Limitations

- The approach relies on the unverified assumption that synthetic cells mapped to the original data are also mapped to the correct clusters
- All approaches overrepresent highly abundant cell types and miss less abundant cell types
- The scVI (posterior) variant showed high variability, most likely due to amplifying artifacts of small datasets
- For sparser 10x Genomics datasets (e.g., PBMC4k), inference from small to larger datasets is more challenging than for Smart-seq2 (Segerstolpe)

## Evidence

- [results] mean absolute difference calculation: "We calculated the mean of the absolute differences between the number of cells in the original dataset and the respective synthetic datasets across all clusters"
- [results] cluster label mapping: "acquired the cluster labels by mapping the synthetic observations to the original data based on the Euclidean distance in UMAP space"
- [abstract] convergence finding: "With increasing pilot dataset sizes, the proportions of the cells in each cluster became more similar to that of ground-truth data."
- [abstract] abundance bias finding: "All approaches showed mixed results for cell types with different abundance by overrepresenting highly abundant cell types and missing less abundant cell types."
- [intro] pilot size grid: "The complete analysis is performed for different sizes of pilot datasets (384, 768, 1152, 1536, 1920, and 2304 cells) and repeated 30 times for each size."
- [readme] Table 1 reproduction script: [evidence span withheld — anchor status: external_doc]
- [discussion] mapping assumption caveat: "We rely on the assumption that synthetic cells mapped to the original data are also mapped to the correct clusters."
