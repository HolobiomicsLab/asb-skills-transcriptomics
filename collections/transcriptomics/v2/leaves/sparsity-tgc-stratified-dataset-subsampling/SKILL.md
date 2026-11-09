---
name: sparsity-tgc-stratified-dataset-subsampling
description: Use when benchmarking scRNA-seq dimensionality reduction or clustering
  tools requires datasets whose sparsity and total gene count variability are systematically
  varied, isolating robustness of methods like scLENS to dropout.
when_to_use_negative:
- The task is single-dataset analysis of real experimental data where the goal is
  biological interpretation, not controlled robustness benchmarking across sparsity
  gradients.
- Data are already low-sparsity bulk or spatial expression matrices where dropout-based
  stress-testing of DR packages is not informative.
- The evaluation requires preserving rare subpopulations below the subsampling scale;
  the ~3000-cell subsample and count-based weights may under-represent low-TGC cells.
tools:
- name: scDesign2
  role: Simulator (v1.0.0) trained on the reference immune cell scRNA-seq dataset
    to generate ~60,000 cells across ten T-cell subtypes before stratified subsampling
- name: scLENS
  role: Target DR method whose default pipeline (log normalization scaling factor
    1, L2 normalization, RMT noise filtering, signal robustness test) is applied to
    each stratified dataset
  repo: github.com/Mathbiomed/scLENS
- name: UMAP
  role: UMAP.jl used to build the 2D embedding (15 neighbors, min_dist 0.01, cosine
    metric) for hierarchical clustering of every package's output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:996512ddd5fad7cecf3f39c26d487c3462fdce953764624f0689e7034e0f99f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ACTIONet
  - Monocle3
  - ParallelPCA
  - Randomly
  - SHARP
  - Scanpy
  - Seurat
  - UMAP
  - ZINB-WaVE
  - scDHA
  - scDesign2
  - scLENS
  - scTransform
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# sparsity-tgc-stratified-dataset-subsampling

## Summary

Construct sparsity- and total-gene-count-variability stratified simulated scRNA-seq benchmark datasets by training a scDesign2 (v1.0.0) simulator on a reference immune cell dataset and subsampling ~3000 cells with count-based probability weights, to stress-test dimensionality reduction methods across controlled sparsity levels and CVs of total gene count (TGC).

## When to use

Use when benchmarking scRNA-seq dimensionality reduction or clustering tools requires datasets whose sparsity and total gene count variability are systematically varied, isolating robustness of methods like scLENS to dropout.

## When NOT to use

- The task is single-dataset analysis of real experimental data where the goal is biological interpretation, not controlled robustness benchmarking across sparsity gradients.
- Data are already low-sparsity bulk or spatial expression matrices where dropout-based stress-testing of DR packages is not informative.
- The evaluation requires preserving rare subpopulations below the subsampling scale; the ~3000-cell subsample and count-based weights may under-represent low-TGC cells.

## Inputs

- Reference immune cell scRNA-seq dataset for scDesign2 training (cell-by-gene count matrix)
- scDesign2 simulator, version 1.0.0
- Simulated cell population (~60,000 cells, ten T-cell subtypes) with ground-truth subtype labels
- Weights per cell proportional to total gene counts (TGC) for weighted random subsampling

## Outputs

- 13 simulated scRNA-seq datasets of ~3000 cells each, stratified by sparsity (up to ~0.84) and CV of TGC (~0.3), with ground-truth subtype labels
- SIL and ECS score tables per package per dataset (Fig. 4a,c benchmark tables)
- Cluster assignments from UMAP hierarchical clustering and Leiden SNN clustering for each package embedding

## How to apply

Train the scDesign2 simulator (version 1.0.0) on a reference immune cell scRNA-seq dataset to simulate approximately 60,000 cells spanning ten T-cell subtypes. Subsample ~3000 cells per simulated dataset using weighted random sampling where probability weights are based on cells' total gene counts, yielding 13 datasets stratified so that five have similar CV of TGC (~0.3) with varying sparsity and five have similar sparsity (~0.84) with varying CV of TGC. Process each dataset with scLENS' default pipeline (log normalization with scaling factor 1, gene scaling, L2 normalization, RMT-based noise filtering, signal robustness test) and with 11 baseline DR packages at default settings. Evaluate every package-dataset pair by hierarchical (UMAP-based, 15 neighbors, min_dist 0.01, cosine) and Leiden (SNN graph, 20 neighbors, edge cutoff 1/15, cosine) clustering, sweeping cut-tree heights/resolutions to maximize ECS against ground truth, then compute SIL and ECS (damping factor 0.975) to compile score tables. Correct application is judged by whether scLENS retains the highest SIL/ECS with the smallest decline as sparsity increases, while fixed-PC-count baselines degrade substantially.

## Related tools

- **scDesign2** (Simulator (v1.0.0) trained on the reference immune cell scRNA-seq dataset to generate ~60,000 cells across ten T-cell subtypes before stratified subsampling)
- **scLENS** (Target DR method whose default pipeline (log normalization scaling factor 1, L2 normalization, RMT noise filtering, signal robustness test) is applied to each stratified dataset) — github.com/Mathbiomed/scLENS
- **UMAP** (UMAP.jl used to build the 2D embedding (15 neighbors, min_dist 0.01, cosine metric) for hierarchical clustering of every package's output)

## Evaluation signals

- The 13 generated datasets split cleanly into five with CV of TGC ~0.3 and varying sparsity, and five with sparsity ~0.84 and varying CV of TGC; verify these moments per dataset.
- Each dataset contains ~3000 cells drawn from the ~60,000-cell scDesign2 simulation of ten T-cell subtypes; check cell counts and subtype coverage.
- Subsampling probabilities should track cells' total gene counts (weighted random sampling), i.e., sampled mean TGC should exceed the population mean.
- SIL and ECS (damping factor 0.975) computed against ground-truth labels should show scLENS with the highest scores and the smallest decline as sparsity rises, while fixed-50-PC baselines (Seurat/Scanpy/Monocle3) degrade substantially.
- Clustering sweeps (cut-tree heights, Leiden resolutions) should terminate at the assignment maximizing ECS to ground truth for every package-dataset pair.

## Limitations

- Simulated data depend on the scDesign2 fit; conclusions about real high-sparsity datasets rely on the reference immune cell training data.
- RMT-based signal detection (as in scLENS) requires computing all eigenvalues of a large cell-to-cell similarity matrix, demanding significant memory resources; running on large data with whole genes still requires a computer with high specifications.
- Baselines run at fixed settings (50 PCs for Seurat/Scanpy/Monocle3, 10,000 highly variable genes for scVI) may understate their performance compared to per-dataset tuning.

## Evidence

- [other] scDesign2 simulation and stratified subsampling protocol: [evidence span withheld — anchor status: not_found]
- [other] Count-weighted subsampling into sparsity/TGC strata: [evidence span withheld — anchor status: not_found]
- [abstract] Motivation for high-sparsity stress-testing: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
- [readme] QC filtering thresholds in scLENS that shape sparsity of processed inputs: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality."
