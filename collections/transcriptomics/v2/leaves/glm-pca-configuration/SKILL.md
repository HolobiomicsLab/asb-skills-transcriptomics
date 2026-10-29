---
name: glm-pca-configuration
description: Use when benchmarking dimension-reduction methods on scRNAseq counts
  and a fast, scalable non-CA baseline (glmPCA) is needed to compare runtime or cluster
  accuracy against standard CA in corral.
when_to_use_negative:
- 'When the goal is the article''s best-performing embedding: the five CA adaptations
  (especially CA-FT) outperformed standard CA and glmPCA in 8 out of 9 datasets, so
  glmPCA is a baseline, not the recommended method'
- When input dimensionality has not been standardized (the timing comparison fixes
  features at 1500, so runtimes are not comparable across differently sized matrices)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: glmPCA
  role: Scalable generalized-PCA baseline run on the down-selected count matrices
    for runtime and clustering-accuracy comparison against corral's standard CA
- name: corral
  role: Provides the standard CA comparator (irlba-based partial SVD) and the SingleCellExperiment
    interface that defines the shared input format
  repo: github.com/laurenhsu1/corral
- name: R
  role: Language in which the timing benchmark and comparison table compilation are
    executed
- name: Bioconductor
  role: Package ecosystem supplying the SingleCellExperiment classes used to hold
    and pass benchmark count matrices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:cfd9a212afca2843d5e77f0f87b06b042d90d2d83753f8b7cd691ecfb768604d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# glm-pca-configuration

## Summary

Configure and run glmPCA on down-selected single-cell count matrices as a scalable generalized-PCA baseline against which corral's standard CA runtime and clustering performance are benchmarked.

## When to use

Use when benchmarking dimension-reduction methods on scRNAseq counts and a fast, scalable non-CA baseline (glmPCA) is needed to compare runtime or cluster accuracy against standard CA in corral.

## When NOT to use

- When the goal is the article's best-performing embedding: the five CA adaptations (especially CA-FT) outperformed standard CA and glmPCA in 8 out of 9 datasets, so glmPCA is a baseline, not the recommended method
- When input dimensionality has not been standardized (the timing comparison fixes features at 1500, so runtimes are not comparable across differently sized matrices)

## Inputs

- Down-selected single-cell count matrix (e.g., 1500 features x ~22,000 cells) from benchmark scRNAseq datasets such as Zhengmix4eq or SCMixology
- SingleCellExperiment or similar Bioconductor count container holding the raw counts

## Outputs

- Per-dataset, per-method wall-clock timing table (runtime_comparison.csv)
- glmPCA cell embeddings for downstream clustering/ARI comparison against corral CA embeddings

## How to apply

Run glmPCA on the same standardized input matrices used for corral's standard CA — in the paper's runtime comparison, ten benchmark scRNAseq datasets (including Zhengmix4eq and SCMixology) were down-selected to 1500 features to fix dimensionality. Record elapsed wall-clock time per dataset per method and compile the timings into a single comparison table (runtime_comparison.csv), reporting runtime per method per dataset. In the article's setup, glmPCA was applied to matrices of 1500 features x ~22,000 cells, where it required over an hour while corral's irlba-based partial SVD CA completed in under a minute. Beyond timing, glmPCA is the reference method in the article's clustering benchmark, where the five CA adaptations were evaluated against standard CA and glmPCA on cluster recovery (ARI) across 9 datasets. Use the corral SingleCellExperiment interface so both methods consume identical count inputs.

## Related tools

- **glmPCA** (Scalable generalized-PCA baseline run on the down-selected count matrices for runtime and clustering-accuracy comparison against corral's standard CA)
- **corral** (Provides the standard CA comparator (irlba-based partial SVD) and the SingleCellExperiment interface that defines the shared input format) — github.com/laurenhsu1/corral
- **R** (Language in which the timing benchmark and comparison table compilation are executed)
- **Bioconductor** (Package ecosystem supplying the SingleCellExperiment classes used to hold and pass benchmark count matrices)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); system.time(gp <- glmPCA(counts(sce_downsel), factors = 30))
```

## Evaluation signals

- Runtimes are recorded per dataset and per method in runtime_comparison.csv with one row per method-dataset pair
- For ~22,000-cell datasets at 1500 features, corral CA wall-clock time is under one minute while glmPCA exceeds one hour — reproduced timings should match this ordering and rough magnitudes
- Both methods are run on identical down-selected matrices (same 1500 features, same cell set), so embeddings are directly comparable
- When glmPCA is used in the clustering benchmark, ARI against original-publication cell labels should place standard CA and glmPCA at or below the CA adaptations in 8 of 9 datasets

## Limitations

- No tool or package versions were reported for corral, glmPCA, or their dependencies, so exact runtime reproduction may vary with versions
- Ground-truth labels for ARI evaluation come from original dataset publications and were not independently validated; ARI may penalize methods that better distinguish sub-populations or rare cell types
- Benchmark datasets (SCMixology, Zhengmix) are simple mixtures of well-defined cell types and do not reflect complex-tissue data; no accession IDs or links are provided in the discussion

## Evidence

- [abstract] we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA, to compute cell embeddings with more performant or comparable clustering accuracy in 8 out of 9"
- [intro] We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment): "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [intro] We benchmark the performance of each of these compared to standard CA and with glmPCA2, a popular method in the field.: "We benchmark the performance of each of these compared to standard CA and with glmPCA2, a popular method in the field."
- [discussion] the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments: "the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments"
- [readme] To install the dev version: `devtools::install_github('laurenhsu1/corral')`: "To install the dev version: `devtools::install_github('laurenhsu1/corral')`"
