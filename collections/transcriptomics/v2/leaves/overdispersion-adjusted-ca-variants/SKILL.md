---
name: overdispersion-adjusted-ca-variants
description: Use when dimension-reducing raw scRNA-seq count matrices (SingleCellExperiment)
  whose counts are overdispersed and where log(x+1)+PCA pipelines underperform, or
  when benchmarking embeddings against ground-truth cell-type labels with ARI.
when_to_use_negative:
- When the goal is multi-table batch integration rather than single-table dimension
  reduction — corralm, not the corral CA-variant benchmarking path, is the appropriate
  entry point
- When the input is already log-transformed 'logcounts' intended for a PCA pipeline
  — CA operates on raw counts via chi-squared residuals and should not be applied
  to log-transformed matrices
- When ground-truth cell labels are independently validated complex-tissue annotations
  are required — the SCMixology/Zhengmix benchmarks are simple pseudo-cell mixtures
  without independently validated labels
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: corral
  role: R/Bioconductor package that computes standard CA (SVD of Pearson residuals)
    and the five overdispersion adaptations on SingleCellExperiment objects
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Competing dimension-reduction method run with 10 random seeds per dataset
    as the benchmark comparison for ARI
- name: R
  role: Language in which the benchmarking workflow (loading SingleCellExperiment
    objects, clustering, ARI computation) is executed
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes that corral interfaces with
    directly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:c68e41c467d693417c60a3890123c0becbbc48a4f5599f5529dc5899fb8cb904
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# overdispersion-adjusted-ca-variants

## Summary

Applies the corral R/Bioconductor package's five CA adaptations for overdispersed scRNA-seq counts (Freeman-Tukey residuals, square-root/Anscombe/Freeman-Tukey variance-stabilizing transformations, and power deflation with alpha = 0.9), and benchmarks their cell-population recovery via ARI against standard CA and glmPCA.

## When to use

Use when dimension-reducing raw scRNA-seq count matrices (SingleCellExperiment) whose counts are overdispersed and where log(x+1)+PCA pipelines underperform, or when benchmarking embeddings against ground-truth cell-type labels with ARI.

## When NOT to use

- When the goal is multi-table batch integration rather than single-table dimension reduction — corralm, not the corral CA-variant benchmarking path, is the appropriate entry point
- When the input is already log-transformed 'logcounts' intended for a PCA pipeline — CA operates on raw counts via chi-squared residuals and should not be applied to log-transformed matrices
- When ground-truth cell labels are independently validated complex-tissue annotations are required — the SCMixology/Zhengmix benchmarks are simple pseudo-cell mixtures without independently validated labels

## Inputs

- Nine benchmark scRNA-seq count datasets as SingleCellExperiment objects (including Zhengmix4eq, SCMixology, and Zhengmix series)
- Published ground-truth cell-type label vectors for each dataset

## Outputs

- Cell embeddings (low-dimensional representations) from standard CA and each of the five overdispersion-adapted CA variants and glmPCA
- ARI benchmark table of maximum ARI per method per dataset (e.g., ari_benchmark_table.csv)

## How to apply

Load the nine benchmark datasets (Zhengmix4eq, SCMixology, Zhengmix series) as SingleCellExperiment objects in R/Bioconductor. Compute cell embeddings with corral: standard CA via SVD of the Pearson residuals, then the overdispersion adaptations — CA with Freeman-Tukey residuals, CA with square-root, Anscombe, and Freeman-Tukey variance-stabilizing transformations, and CA with power deflation (alpha = 0.9). Run glmPCA on each dataset with 10 random seeds to account for stochastic initialization. Cluster cells in each embedding and compute the Adjusted Rand Index (ARI) against the published ground-truth cell-type labels, tabulating the maximum ARI per method per dataset. Rationale: CA decomposes a chi-squared residual matrix and avoids the distortive log-transformation of typical logcounts/PCA pipelines, and the Freeman-Tukey-residual variant (CA-FT) was most performant overall in the cluster recovery task.

## Related tools

- **corral** (R/Bioconductor package that computes standard CA (SVD of Pearson residuals) and the five overdispersion adaptations on SingleCellExperiment objects) — github.com/laurenhsu1/corral
- **glmPCA** (Competing dimension-reduction method run with 10 random seeds per dataset as the benchmark comparison for ARI)
- **R** (Language in which the benchmarking workflow (loading SingleCellExperiment objects, clustering, ARI computation) is executed)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes that corral interfaces with directly)

## Examples

```
BiocManager::install('laurenhsu1/corral', dependencies = TRUE); sce <- corral(sce, method = 'ft'); ari <- mclust::adjustedRandIndex(emb_clusters, sce$cell_type)
```

## Evaluation signals

- The reproduced ARI table should show the CA adaptations outperforming standard CA and glmPCA in 8 out of 9 datasets
- CA-FT should be the highest-ARI method overall in 6 out of 9 benchmark datasets and outperform standard CA in all nine datasets
- glmPCA results should reflect maximum ARI across 10 random seeds to control for stochastic initialization
- Each method's embedding must be compared on the same clustering procedure and the same published ground-truth cell-type labels to yield a fair ARI per dataset
- corral installation should succeed via BiocManager::install('laurenhsu1/corral', dependencies = TRUE) and the corral/corralm main function calls should return low-dimension embeddings

## Limitations

- No tool or package versions reported for corral or its dependencies
- Benchmarking ground-truth labels come from original dataset publications and were not independently validated
- ARI evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- High-quality ground-truth benchmarking datasets are limited to simple pseudo-cell mixtures or pools of distinct cell types and do not reflect real-world tissue complexity
- No accession IDs or links for the SCMixology and Zhengmix benchmarking datasets are provided in the discussion

## Evidence

- [abstract] Five proposed adaptations of CA are fast, scalable, and outperform standard CA and glmPCA, achieving more performant or comparable clustering accuracy in 8 out of 9 datasets.: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9"
- [discussion] CA-FT most performant in cluster recovery.: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [intro] Standard CA computed via SVD of Pearson residuals.: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [intro] corral implemented in R/Bioconductor interfacing with Bioconductor classes.: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes"
- [readme] README: two main function calls corral and corralm.: "The package contains two main function calls: 1. `corral` for dimensionality reduction on a single table"
