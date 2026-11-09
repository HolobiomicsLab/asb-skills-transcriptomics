---
name: single-cell-dimension-reduction
description: Use when reducing dimensions of raw scRNAseq count matrices (SingleCellExperiment
  objects) to embed cells for clustering and cell-type recovery, and standard log(x+1)+PCA
  pipelines distort count data or overdispersion is a concern.
when_to_use_negative:
- The goal is multi-table batch integration across samples or modes — use corralm,
  the companion corral function for integrative alignment, rather than single-table
  corral.
- 'The benchmark context demands independently validated ground-truth labels: the
  article notes dataset annotations came from original publications without independent
  validation, limiting conclusions on complex tissues.'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: R/Bioconductor package used to compute standard CA and the five overdispersion
    adaptations (CA-FT, square-root/Anscombe/Freeman-Tukey variance-stabilizing transformations,
    power deflation) as cell embeddings from scRNAseq count matrices
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Baseline dimension-reduction comparator run with 10 random seeds on each benchmark
    dataset for ARI comparison against CA variants
- name: corralm
  role: Companion method for integrative multi-table dimension reduction of scRNAseq
    data (batch integration), distinct from single-table reduction in this skill
  repo: github.com/laurenhsu1/corral
- name: R
  role: Language environment for running the corral/glmPCA benchmarking pipeline
- name: Bioconductor
  role: Package ecosystem providing SingleCellExperiment classes that corral interfaces
    with directly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:9d58583f89f02ece43865cd77ed2640f407aa78c570652993f791a6f11f36dc0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Bioconductor (>=3.6)
  - R
  - R (>=3.4)
  - corral
  - corralm
  - glmPCA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-dimensionality-reduction
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-cell-dimension-reduction

## Summary

Dimension reduction of single-cell RNA-seq count matrices via correspondence analysis (CA) and five overdispersion-adapted CA variants (notably CA with Freeman-Tukey residuals), implemented in the corral R/Bioconductor package, benchmarked against standard CA and glmPCA using Adjusted Rand Index of recovered cell populations.

## When to use

Use when reducing dimensions of raw scRNAseq count matrices (SingleCellExperiment objects) to embed cells for clustering and cell-type recovery, and standard log(x+1)+PCA pipelines distort count data or overdispersion is a concern.

## When NOT to use

- The goal is multi-table batch integration across samples or modes — use corralm, the companion corral function for integrative alignment, rather than single-table corral.
- The benchmark context demands independently validated ground-truth labels: the article notes dataset annotations came from original publications without independent validation, limiting conclusions on complex tissues.

## Inputs

- scRNAseq raw count matrices as SingleCellExperiment objects (e.g., Zhengmix4eq, SCMixology, Zhengmix benchmark datasets)
- Published ground-truth cell-type label annotations per dataset

## Outputs

- Low-dimensional cell embeddings from corral (standard CA, CA-FT, variance-stabilized CA variants, power-deflated CA) and glmPCA
- Per-dataset clustering ARI benchmark table comparing methods (e.g., ari_benchmark_table.csv)

## How to apply

Load scRNAseq count datasets (e.g., Zhengmix4eq, SCMixology, Zhengmix benchmarks) as SingleCellExperiment objects in R/Bioconductor. Compute cell embeddings with the corral package: standard CA via SVD of the Pearson residuals, or overdispersion adaptations including CA with Freeman-Tukey residuals (CA-FT), CA with square-root, Anscombe, and Freeman-Tukey variance-stabilizing transformations, and CA with power deflation (alpha = 0.9). As a baseline comparator, run glmPCA with 10 random seeds to account for stochastic initialization. Cluster cells in each method's embedding and compute Adjusted Rand Index (ARI) against published ground-truth cell-type labels; compare maximum ARI per method per dataset across all nine benchmark datasets.

## Related tools

- **corral** (R/Bioconductor package used to compute standard CA and the five overdispersion adaptations (CA-FT, square-root/Anscombe/Freeman-Tukey variance-stabilizing transformations, power deflation) as cell embeddings from scRNAseq count matrices) — github.com/laurenhsu1/corral
- **glmPCA** (Baseline dimension-reduction comparator run with 10 random seeds on each benchmark dataset for ARI comparison against CA variants)
- **corralm** (Companion method for integrative multi-table dimension reduction of scRNAseq data (batch integration), distinct from single-table reduction in this skill) — github.com/laurenhsu1/corral
- **R** (Language environment for running the corral/glmPCA benchmarking pipeline)
- **Bioconductor** (Package ecosystem providing SingleCellExperiment classes that corral interfaces with directly)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); sce <- corral(my_sce, method = "ft")
```

## Evaluation signals

- The reproduced ARI table should show the five CA adaptations outperforming standard CA and glmPCA in 8 out of 9 datasets.
- CA with Freeman-Tukey residuals (CA-FT) should achieve the highest overall ARI in 6 out of 9 benchmark datasets and outperform standard CA in all nine datasets.
- glmPCA results should be stable across 10 random seeds (maximum ARI taken per dataset) to account for stochastic initialization.
- Embeddings should be computed on raw counts via SVD of chi-squared/Pearson residual matrices, not on log(x+1)-transformed 'logcounts'.

## Limitations

- No tool or package versions reported for corral or its dependencies.
- Benchmarking ground-truth labels come from original dataset publications and were not independently validated, so performance is limited by the original cell type assignments.
- ARI evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels.
- High-quality ground-truth benchmarking datasets are limited to simple pseudo-cell mixtures or pools of distinct cell types and do not reflect real-world tissue complexity; no accession IDs or links for SCMixology and Zhengmix datasets are provided in the discussion.
- Optimal dimension reduction approach cannot yet be identified based on data characteristics due to lack of systematic benchmarking frameworks; the number of true clusters is typically unknown a priori and context-dependent.

## Evidence

- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [abstract] Five adaptations outperform standard CA and glmPCA: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9"
- [intro] CA computed by SVD of Pearson residuals: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [discussion] CA-FT most performant overall: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [readme] corral installation from GitHub: "To install the dev version: `devtools::install_github('laurenhsu1/corral')`"
