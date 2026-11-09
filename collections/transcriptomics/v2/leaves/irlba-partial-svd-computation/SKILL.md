---
name: irlba-partial-svd-computation
description: Use when a large single-cell count matrix (e.g., 1500 features x >20,000
  cells) needs fast, scalable dimension reduction or when comparing wall-clock runtime
  of CA against glmPCA on benchmark datasets.
when_to_use_negative:
- The goal is integrating multiple tables or batches — use corralm for multi-table
  alignment instead of single-table corral
- The analysis requires model-based factorization that explicitly handles overdispersion
  (e.g., glmPCA-style negative binomial modeling) rather than fast chi-squared residual
  decomposition
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: corral
  role: R/Bioconductor package performing standard CA via irlba-based partial SVD
    on a single count table
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Baseline method timed against corral for the runtime comparison
- name: R
  role: Runtime environment for executing the corral timing benchmark
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes that corral interfaces with
    directly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:4d4a6a4e664d406624359f6df016092037a75ebd9081e3f0b613e3b84d653c30
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# irlba-partial-svd-computation

## Summary

Computes correspondence analysis embeddings for scRNAseq count matrices by partial SVD of the chi-squared residual matrix using irlba, as implemented in the corral R/Bioconductor package. This keeps runtime under a minute even for ~22,000-cell datasets, where glmPCA requires over an hour.

## When to use

Use when a large single-cell count matrix (e.g., 1500 features x >20,000 cells) needs fast, scalable dimension reduction or when comparing wall-clock runtime of CA against glmPCA on benchmark datasets.

## When NOT to use

- The goal is integrating multiple tables or batches — use corralm for multi-table alignment instead of single-table corral
- The analysis requires model-based factorization that explicitly handles overdispersion (e.g., glmPCA-style negative binomial modeling) rather than fast chi-squared residual decomposition

## Inputs

- Single-cell count matrix or SingleCellExperiment object (e.g., Zhengmix4eq, SCMixology benchmarks with original-publication cell labels)
- Feature-downselected matrix standardized to 1500 features

## Outputs

- Cell embeddings from the partial SVD of the chi-squared residual matrix
- Per-dataset wall-clock timing table (runtime_comparison.csv) reporting runtime per method per dataset

## How to apply

Load the single-cell count matrix (e.g., a SingleCellExperiment from Zhengmix4eq or SCMixology benchmarks) and down-select to 1500 features to standardize dimensionality. Call the corral function, which uses irlba-based partial SVD to decompose the Pearson (chi-squared) residual matrix and return cell embeddings, avoiding the log(x + 1) transformation used before PCA in typical workflows. Record elapsed wall-clock time per dataset when benchmarking against glmPCA on the same down-selected matrix. Validate that standard CA completes in under a minute even for datasets with over 20,000 cells, while glmPCA takes over an hour on the equivalent 1500-feature x ~22,000-cell input. Compile per-dataset, per-method timings into a runtime_comparison.csv table.

## Related tools

- **corral** (R/Bioconductor package performing standard CA via irlba-based partial SVD on a single count table) — github.com/laurenhsu1/corral
- **glmPCA** (Baseline method timed against corral for the runtime comparison)
- **R** (Runtime environment for executing the corral timing benchmark)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes that corral interfaces with directly)

## Examples

```
devtools::install_github('laurenhsu1/corral'); sce <- corral(sce, whichmat = 'counts')
```

## Evaluation signals

- Standard CA (corral) wall-clock runtime stays under one minute even for datasets with over 20,000 cells
- glmPCA on the same 1500-feature x ~22,000-cell matrix requires over an hour, reproducing the reported scaling gap
- Timings are compiled per method per dataset into runtime_comparison.csv with all ten benchmark datasets represented
- Resulting CA embeddings support downstream clustering accuracy comparable to or better than PCA/logcounts baselines

## Limitations

- No tool or package versions reported for corral or dependencies, complicating exact reproduction of timings
- Runtime advantage is demonstrated only on down-selected matrices (1500 features); full-feature scalability is not characterized in this task
- Benchmarking ground-truth labels come from original dataset publications and were not independently validated

## Evidence

- [results] Standard CA (corral) consistently runs in under a minute even for datasets with over 20,000 cells, while glmPCA scales less favorably: [evidence span withheld — anchor status: not_found]
- [abstract] corral is an R/Bioconductor package which interfaces directly with single cell classes in Bioconductor: "We implement CA for scRNAseq data in corral, an R/Bioconductor package which interfaces directly with single cell classes in Bioconductor."
- [intro] CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [readme] The package contains two main function calls: corral for dimensionality reduction on a single table: "1. `corral` for dimensionality reduction on a single table"
- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
