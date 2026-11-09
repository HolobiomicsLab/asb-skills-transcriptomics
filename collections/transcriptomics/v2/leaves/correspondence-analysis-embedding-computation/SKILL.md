---
name: correspondence-analysis-embedding-computation
description: Use when starting from raw scRNA-seq count data (e.g., SingleCellExperiment
  objects) and you need a dimension-reduction embedding for clustering, cell-type
  annotation, or visualization that avoids distortive log(x+1) transformation.
when_to_use_negative:
- The input is already log(x+1)-transformed 'logcounts' or another continuous transformed
  matrix — CA's residual decomposition is defined on raw counts and the pipeline substitution
  works from counts directly.
- You need independently validated ground truth for strict benchmarking — the article
  notes the benchmark datasets' cell annotations come from original publications and
  were not independently validated, so ARI gains should be interpreted cautiously.
- Complex-tissue data where you expect rare sub-populations the reference labels miss
  — the article warns ARI may penalize methods that better distinguish sub-populations
  relative to 'reference'.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: 'Main implementation: computes standard CA and the five overdispersion adaptations
    (CA-FT, square-root/Anscombe/Freeman-Tukey VSTs, power deflation) on a single
    count table, interfacing with Bioconductor SingleCellExperiment'
  repo: github.com/laurenhsu1/corral
- name: corralm
  role: Multi-table variant used for integrative dimension reduction and batch alignment
    across multiple count tables (e.g., Baron/Lawlor/Muraro pancreas datasets with
    Freeman-Tukey residuals)
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Baseline dimension-reduction method used for comparison in the ARI benchmark
    (run with 10 random seeds to account for stochastic initialization)
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:41e2ab1e2f4921d0dd0afc6f8e2139e1bfc8f1194669659ccb062c5b9995744f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - MNN
  - R
  - corral
  - corralm
  - glmPCA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# correspondence-analysis-embedding-computation

## Summary

Computes low-dimensional cell embeddings from scRNA-seq count matrices by SVD of chi-squared (Pearson or Freeman-Tukey) residual matrices instead of log(x+1)-transformed PCA, using the corral R/Bioconductor package; avoids log-transformation distortion and improves cluster recovery.

## When to use

Use when starting from raw scRNA-seq count data (e.g., SingleCellExperiment objects) and you need a dimension-reduction embedding for clustering, cell-type annotation, or visualization that avoids distortive log(x+1) transformation.

## When NOT to use

- The input is already log(x+1)-transformed 'logcounts' or another continuous transformed matrix — CA's residual decomposition is defined on raw counts and the pipeline substitution works from counts directly.
- You need independently validated ground truth for strict benchmarking — the article notes the benchmark datasets' cell annotations come from original publications and were not independently validated, so ARI gains should be interpreted cautiously.
- Complex-tissue data where you expect rare sub-populations the reference labels miss — the article warns ARI may penalize methods that better distinguish sub-populations relative to 'reference'.

## Inputs

- scRNA-seq count matrix (SingleCellExperiment object or counts matrix) for a single table (corral) or multiple batches/samples/modes (corralm)
- Published ground-truth cell-type labels for ARI evaluation (optional)

## Outputs

- Low-dimensional cell embeddings (reduced-dimension representation, e.g., first 8-10 dimensions) stored in the SingleCellExperiment
- ARI benchmark table per method per dataset (e.g., ari_benchmark_table.csv)
- CA biplot for geometric interpretation of cells and features

## How to apply

Load the count matrix as a SingleCellExperiment in R/Bioconductor and run corral, which computes correspondence analysis by SVD of a chi-squared residual matrix. For overdispersed scRNA-seq counts, choose an adaptation: CA with Freeman-Tukey residuals (best overall), square-root, Anscombe, or Freeman-Tukey variance-stabilizing transformations, or power deflation (alpha = 0.9). Use the first ~8 dimensions of the resulting embeddings for downstream clustering; compare embeddings by clustering cells and computing Adjusted Rand Index against annotated cell-type labels, or by silhouette width (ASW_celltype, 1-ASWbatch on 8 PCs) in batch-integration settings. CA-FT outperformed standard CA in all nine benchmark datasets and the CA adaptations outperformed standard CA and glmPCA in 8 out of 9.

## Related tools

- **corral** (Main implementation: computes standard CA and the five overdispersion adaptations (CA-FT, square-root/Anscombe/Freeman-Tukey VSTs, power deflation) on a single count table, interfacing with Bioconductor SingleCellExperiment) — github.com/laurenhsu1/corral
- **corralm** (Multi-table variant used for integrative dimension reduction and batch alignment across multiple count tables (e.g., Baron/Lawlor/Muraro pancreas datasets with Freeman-Tukey residuals)) — github.com/laurenhsu1/corral
- **glmPCA** (Baseline dimension-reduction method used for comparison in the ARI benchmark (run with 10 random seeds to account for stochastic initialization))

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); sce <- corral(sce)
```

## Evaluation signals

- CA adaptations should outperform standard CA and glmPCA in 8 out of 9 benchmark datasets by ARI.
- CA-FT should be the most performant overall, achieving the highest ARI in 6 out of 9 datasets and beating standard CA in all nine.
- Reproduced table (ari_benchmark_table.csv) should match maximum ARI per method per dataset structure across Zhengmix4eq, SCMixology, and Zhengmix series datasets.
- In integration settings (corralm+Harmony vs Seurat on Baron/Lawlor/Muraro), ASW_celltype and 1-ASWbatch on the first 8 PCs should be comparable to the Seurat routine.
- glmPCA comparisons should use 10 random seeds and take the best ARI to account for stochastic initialization.

## Limitations

- No tool or package versions reported for corral or dependencies.
- Ground-truth labels come from original dataset publications and were not independently validated; performance is limited by original cell type assignments.
- ARI evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels.
- High-quality ground-truth benchmarks are limited to simple pseudo-cell mixtures or pools of distinct cell types and do not reflect real-world tissue complexity; no accession IDs provided for SCMixology and Zhengmix datasets.

## Evidence

- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [intro] CA computed via SVD of Pearson residuals: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [discussion] CA-FT most performant: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [readme] corral is a package to perform correspondence analysis on count data: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
- [readme] Two main function calls: "1. `corral` for dimensionality reduction on a single table 
2. `corralm` for alignment and batch integration of multiple tables"
