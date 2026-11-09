---
name: clustering-accuracy-ari-evaluation
description: Use when comparing dimension-reduction embeddings (e.g., CA variants,
  PCA, glmPCA) on labeled scRNA-seq benchmarks and you need a single cluster-recovery
  score per method per dataset.
when_to_use_negative:
- Datasets lack independently validated cell-type annotations; performance is then
  limited by possibly noisy original cell-type assignments
- Ground-truth labels under-count sub-populations or rare cell types — ARI may penalize
  methods that better distinguish them relative to the 'reference'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: Computes CA and overdispersion-adapted CA (including Freeman-Tukey residuals)
    cell embeddings on single tables for benchmarking
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Baseline dimension-reduction method; run with 10 random seeds to account for
    stochastic initialization when benchmarking
- name: R
  role: Language for the benchmarking workflow and ARI tabulation
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes the benchmark datasets and
    corral interface through
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:7f2af89e926f7b69a19df41cf49f7936a1fbb9d29f425c4be4401b4a3baf50b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# clustering-accuracy-ari-evaluation

## Summary

Benchmark dimension-reduction methods for scRNA-seq by clustering cells in each method's embedding and scoring recovery of published cell-type labels with the Adjusted Rand Index (ARI), as done for CA variants vs standard CA and glmPCA across nine datasets.

## When to use

Use when comparing dimension-reduction embeddings (e.g., CA variants, PCA, glmPCA) on labeled scRNA-seq benchmarks and you need a single cluster-recovery score per method per dataset.

## When NOT to use

- Datasets lack independently validated cell-type annotations; performance is then limited by possibly noisy original cell-type assignments
- Ground-truth labels under-count sub-populations or rare cell types — ARI may penalize methods that better distinguish them relative to the 'reference'

## Inputs

- SingleCellExperiment objects of scRNA-seq count matrices for nine benchmark datasets (e.g., Zhengmix4eq, SCMixology, Zhengmix series)
- Published ground-truth cell-type label vector per dataset
- Cell embeddings from each dimension-reduction method (e.g., corral CA/CA-FT, glmPCA)

## Outputs

- ARI value per method per dataset
- Benchmark table of maximum ARI per method per dataset (e.g., ari_benchmark_table.csv)

## How to apply

Load each benchmark scRNA-seq count dataset as a SingleCellExperiment object and compute cell embeddings with each candidate method (e.g., corral's standard CA via SVD of Pearson residuals, CA-FT, other overdispersion adaptations, and glmPCA with 10 random seeds to account for stochastic initialization). Cluster cells in each embedding and compute the Adjusted Rand Index (ARI) against the published ground-truth cell-type labels. Tabulate the maximum ARI per method per dataset (maximizing over seeds/parameters for stochastic methods). In the source benchmark, the CA adaptations outperformed standard CA and glmPCA in 8 out of 9 datasets, with CA-FT highest overall in 6 of 9 datasets (including Zhengmix4eq and the SCMixology and Zhengmix series).

## Related tools

- **corral** (Computes CA and overdispersion-adapted CA (including Freeman-Tukey residuals) cell embeddings on single tables for benchmarking) — github.com/laurenhsu1/corral
- **glmPCA** (Baseline dimension-reduction method; run with 10 random seeds to account for stochastic initialization when benchmarking)
- **R** (Language for the benchmarking workflow and ARI tabulation)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes the benchmark datasets and corral interface through)

## Examples

```
Rscript -e 'library(corral); emb <- corral(sce, method="svd"); ari <- mclust::adjustedRandIndex(clusters(emb), sce$celltype); write.csv(ari, "ari_benchmark_table.csv")'
```

## Evaluation signals

- Reproduced ARI table should show CA adaptations outperforming standard CA and glmPCA in 8 out of 9 datasets
- CA-FT should be the most performant method overall, highest ARI in 6 of 9 benchmark datasets
- glmPCA results should be summarized as the maximum ARI over 10 random seeds to handle stochastic initialization
- Every method-dataset pair in the output table must have a finite ARI value against the published labels

## Limitations

- Benchmark ground-truth labels come from original dataset publications and were not independently validated
- ARI evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- High-quality ground-truth benchmark datasets are largely simple pseudo-cell mixtures or pools of distinct cell types, not complex tissues; complex-tissue labels come from a single analytical method
- No tool or package versions reported for corral or dependencies
- No accession IDs or links for the SCMixology and Zhengmix datasets are provided in the discussion

## Evidence

- [abstract] we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9"
- [discussion] CA-FT was most performant overall in a scRNAseq cluster recovery task.: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [discussion] the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments: "the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments"
- [discussion] these advantages may not be reflected in the ARI, and the method would actually receive a small penalty for differences from “reference.”: "these advantages may not be reflected in the ARI, and the method would actually receive a small penalty for differences from “reference.”"
- [discussion] most high-quality, “ground-truth” benchmarking datasets are derived from simple “pseudo”-cell mixtures, or from pools of distinct cell types: "most high-quality, “ground-truth” benchmarking datasets are derived from simple “pseudo”-cell mixtures, or from pools of distinct cell types"
