---
name: multi-dataset-benchmark-comparison
description: Use when you need to compare multiple dimension-reduction or embedding
  methods across several labeled scRNAseq benchmark datasets to quantify which method
  recovers known cell populations.
when_to_use_negative:
- Datasets lack ground-truth cell-type labels, since ARI against published annotations
  cannot be computed
- The goal is batch integration of multiple tables rather than single-table method
  benchmarking (corralm, not the corral benchmark workflow, is the appropriate tool)
- The aim is to evaluate methods on complex-tissue data with realistic, independently
  validated cell identities — the article notes available ground-truth datasets are
  limited to simple pseudo-cell mixtures or pools of distinct cell types
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: corral
  role: Computes standard CA (SVD of Pearson residuals) and the five overdispersion-adapted
    CA embeddings on each benchmark dataset
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Baseline dimension-reduction method run with 10 random seeds per dataset for
    stochastic-initialization robustness
- name: R
  role: Language in which the benchmark workflow is executed
- name: Bioconductor
  role: Framework providing SingleCellExperiment classes that corral interfaces with
    directly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:a07e27a171bd7df698287b3d0053e2de592c506c7a5c0878d86dc9417983cbe2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# multi-dataset-benchmark-comparison

## Summary

A benchmarking workflow that compares dimension-reduction methods (standard CA, five overdispersion-adapted CA variants, and glmPCA) across nine scRNAseq datasets by clustering embeddings and computing ARI against published cell-type labels, as done in Hsu & Culhane's corral paper.

## When to use

Use when you need to compare multiple dimension-reduction or embedding methods across several labeled scRNAseq benchmark datasets to quantify which method best recovers known cell populations.

## When NOT to use

- Datasets lack ground-truth cell-type labels, since ARI against published annotations cannot be computed
- The goal is batch integration of multiple tables rather than single-table method benchmarking (corralm, not the corral benchmark workflow, is the appropriate tool)
- The aim is to evaluate methods on complex-tissue data with realistic, independently validated cell identities — the article notes available ground-truth datasets are limited to simple pseudo-cell mixtures or pools of distinct cell types

## Inputs

- Nine scRNAseq benchmark count datasets as SingleCellExperiment objects (Zhengmix4eq, SCMixology series, Zhengmix series)
- Published ground-truth cell-type labels for each dataset
- Method implementations from the corral and glmPCA R/Bioconductor packages

## Outputs

- ARI benchmark table (e.g., ari_benchmark_table.csv) with maximum ARI per method per dataset
- Per-dataset cell embeddings for standard CA, CA-FT, variance-stabilized CA variants, power-deflated CA (alpha = 0.9), and glmPCA

## How to apply

Load the nine benchmark SingleCellExperiment count datasets (including Zhengmix4eq and the SCMixology and Zhengmix series) in R/Bioconductor. Compute cell embeddings per dataset: standard CA via SVD of Pearson residuals using corral, the overdispersion adaptations (CA with Freeman-Tukey residuals, square-root, Anscombe, and Freeman-Tukey variance-stabilizing transformations), and CA with power deflation (alpha = 0.9). Run glmPCA with 10 random seeds per dataset to account for stochastic initialization. Cluster cells in each method's embedding, compute the Adjusted Rand Index against the published ground-truth cell-type labels, and tabulate the maximum ARI per method per dataset (e.g., written to ari_benchmark_table.csv). A correct reproduction shows the CA adaptations outperforming standard CA and glmPCA in 8 out of 9 datasets, with CA-FT highest overall in 6 of 9 datasets.

## Related tools

- **corral** (Computes standard CA (SVD of Pearson residuals) and the five overdispersion-adapted CA embeddings on each benchmark dataset) — github.com/laurenhsu1/corral
- **glmPCA** (Baseline dimension-reduction method run with 10 random seeds per dataset for stochastic-initialization robustness)
- **R** (Language in which the benchmark workflow is executed)
- **Bioconductor** (Framework providing SingleCellExperiment classes that corral interfaces with directly)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); sce_emb <- corral(sce, whichmat = "counts")
```

## Evaluation signals

- Reproduced ARI table shows the five CA adaptations outperforming standard CA and glmPCA in 8 out of 9 datasets
- CA-FT achieves the highest ARI overall in 6 of the 9 benchmark datasets and beats standard CA in all nine
- glmPCA results are stable across the 10 random seeds used per dataset
- Output table contains one maximum-ARI entry per method per dataset with no missing combinations
- Clustering in each embedding uses the same protocol so ARI comparisons are method-controlled

## Limitations

- Ground-truth labels come from original dataset publications and were not independently validated, limiting performance ceilings
- ARI may penalize methods that better distinguish sub-populations or rare cell types relative to the reference labels
- High-quality ground-truth benchmark datasets are simple pseudo-cell mixtures or pools of distinct cell types that do not reflect real-world tissue complexity
- No tool or package versions are reported for corral or its dependencies, hindering exact reproduction

## Evidence

- [abstract] Five proposed adaptations of CA are fast, scalable, and outperform standard CA and glmPCA, achieving more performant or comparable clustering accuracy in 8 out of 9 datasets.: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9"
- [discussion] CA-FT was most performant overall in the cluster recovery benchmark.: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [discussion] Benchmark labels are not independently validated, capping achievable agreement.: "the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments"
- [discussion] ARI can penalize methods that better resolve sub-populations or rare cell types.: "these advantages may not be reflected in the ARI, and the method would actually receive a small penalty for differences from “reference.”"
- [readme] corral is installed via BiocManager and exposes two main calls, corral and corralm.: "Or it can be installed with `BiocManager::install("laurenhsu1/corral", dependencies = TRUE)`"
