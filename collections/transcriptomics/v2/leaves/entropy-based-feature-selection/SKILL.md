---
name: entropy-based-feature-selection
description: Use when a log-normalized cells-by-genes scRNA-seq matrix needs a small
  set of feature (differentially expressed) genes tied to cell types, alongside dimensionality
  reduction, rather than generic highly-variable-gene selection.
when_to_use_negative:
- The input is already a feature/DE gene table or the goal is purely visualization
  without gene selection — a plain embedding tool (t-SNE, UMAP) alone would suffice
- The dataset is a Splatter-simulated matrix, where the gene-selection step should
  use 500 instead of 1000 top variable genes as the restricted input
- Only global hierarchical structure preservation is required and no cell-type-associated
  feature genes are needed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: EDGE
  role: Provides the endr() ensemble pipeline (weak learners, hashing, spectral embedding,
    SGD optimization) and the averaged entropy values used for the μ − 1.5 × σ feature-gene
    cutoff
  repo: https://github.com/shawnstat/EDGE
- name: R
  role: Implementation language for normalization (log2(x+1)), running endr, and applying
    the entropy cutoff
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition for spectral embedding inside
    EDGE
- name: Rcpp
  role: C++ dependency (>= 1.0.1) required to build and run the EDGE package
- name: Seurat
  role: FindVariableFeatures used to select the 1000 top variable genes restricting
    real datasets before feature selection
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:d255a54db877515cc2014d79c5457800a8ef8e3dd5f5ff316721ed0abea00116
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  - Seurat
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# entropy-based-feature-selection

## Summary

Select feature genes in scRNA-seq data by ranking genes on their averaged entropy contribution across an ensemble of weak learners run by the EDGE endr function, and apply a μ − 1.5 × σ cutoff on the averaged entropy values to define the feature-gene set.

## When to use

Use when a log-normalized cells-by-genes scRNA-seq matrix needs a small set of feature (differentially expressed) genes tied to cell types, alongside dimensionality reduction, rather than generic highly-variable-gene selection.

## When NOT to use

- The input is already a feature/DE gene table or the goal is purely visualization without gene selection — a plain embedding tool (t-SNE, UMAP) alone would suffice
- The dataset is a Splatter-simulated matrix, where the gene-selection step should use 500 instead of 1000 top variable genes as the restricted input
- Only global hierarchical structure preservation is required and no cell-type-associated feature genes are needed

## Inputs

- Raw scRNA-seq count matrix (cells × genes), e.g. from PBMC, Jurkat, or mouse brain datasets
- Library-size normalization factors (median of per-cell column sums)
- endr configuration object (endr_defs) with n_wl=1000, n_dm=10, n_neigs=15, opt=TRUE, seed, H=101107
- Gene-universe background for downstream GO enrichment of the selected feature genes

## Outputs

- Ranked list of genes by averaged entropy contribution across weak learners
- Feature-gene set below the μ − 1.5 × σ entropy cutoff (e.g., 17 genes in Jurkat, 35 in PBMC, 43 in mouse brain)
- Low-dimensional embedding and similarity matrix from endr, usable for visualization and clustering

## How to apply

First normalize counts by library size to the median and log2-transform (log2(counts/size_factor + 1)), producing a cells-by-genes matrix, then restrict real datasets to the 1000 top variable genes (500 for simulated data). Run the EDGE ensemble (endr(dat_log, custom_defs)) with the README hyperparameters: 1000 weak learners (n_wl), 10 genes per weak learner (n_dm), 15 nearest neighbors (n_neigs), optimization enabled, fixed seed, and hash table size 101,107 (reduced from the optimal 1,017,881 to save memory). Each weak learner randomly selects B genes and gene-specific thresholds to build binary sketch vectors mapped via modulo hashing to cell-similarity votes; the genes contributing across weak learners yield averaged entropy values. Rank genes by averaged entropy and select those below the μ − 1.5 × σ cutoff as feature genes; this simultaneously supports dimensionality reduction via spectral embedding plus stochastic-gradient-descent optimization. Validate the selected set against known markers (e.g., in PBMC: PPBP, LYZ, CST3, NKG7, with S100A9 highest-scored).

## Related tools

- **EDGE** (Provides the endr() ensemble pipeline (weak learners, hashing, spectral embedding, SGD optimization) and the averaged entropy values used for the μ − 1.5 × σ feature-gene cutoff) — https://github.com/shawnstat/EDGE
- **R** (Implementation language for normalization (log2(x+1)), running endr, and applying the entropy cutoff)
- **RSpectra** (Performs the large-scale eigenvalue decomposition for spectral embedding inside EDGE)
- **Rcpp** (C++ dependency (>= 1.0.1) required to build and run the EDGE package)
- **Seurat** (FindVariableFeatures used to select the 1000 top variable genes restricting real datasets before feature selection)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$n_neigs <- 15; custom_defs$opt <- TRUE; custom_defs$seed <- 2020; custom_defs$H <- 101107; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Feature-gene counts match the reference: 17 genes (Jurkat), 35 genes (PBMC), 43 genes (mouse brain) under the μ − 1.5 × σ cutoff
- PBMC feature set contains the known marker genes PPBP, LYZ, CST3, and NKG7, with S100A9 receiving the highest importance score
- Selected gene count equals the number of averaged entropy values falling below μ − 1.5 × σ computed from the endr output
- Downstream GO enrichment of the 35 PBMC feature genes yields only immune/response-to-stimulus biological processes (e.g., defense response, leukocyte-mediated immunity) in the top ten terms
- Hash table size, weak learner count, and other endr_defs parameters match the documented example (n_wl=1000, n_dm=10, n_neigs=15, H=101107, seed fixed)

## Limitations

- Larger numbers of weak learners and bigger hash table sizes improve results but require large memory; the example reduces H from the optimal 1,017,881 to 101,107
- Exact per-dataset value of B is not stated, only the heuristic B < 0.05*G; hyperparameters a and b are estimated externally by nonlinear least squares
- The cutoff μ − 1.5 × σ is a heuristic; results depend on the fixed random seed and on restricting to 1000 top variable genes for real datasets (500 for simulated)

## Evidence

- [discussion] Feature gene cutoff at μ − 1.5 × σ of averaged entropy values: "We choose μ −  1.5 × σ as the cutoff value to select top feature genes"
- [discussion] Select 1000 top variable genes for real datasets, 500 for simulated: "We selected 1000 top variable genes for real datasets and 500 for simulated datasets"
- [intro] Ensemble weak learners select contributing genes as feature genes: "massive weak learners are allowed in EDGE to learn cell similarities accurately, while the genes that make significant contributions in the process are selected as feature genes."
- [readme] endr invocation from README with custom_defs hyperparameters: "simu.endr <- endr(dat_log,custom_defs)"
- [readme] Hash table size trade-off documented in README: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example."
