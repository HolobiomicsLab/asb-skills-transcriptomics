---
name: pvalue-threshold-classification
description: Use when a sparse scRNA-seq count/FPKM matrix contains zero entries that
  must be partitioned into genuine biological zeros versus dropout-affected zeros
  before imputation or regression-based value recovery.
when_to_use_negative:
- When the goal is to impute ALL zero-valued entries indiscriminately (the article
  criticizes methods that 'attempt to alter the expression of all zero-valued entries,
  including those not affected by dropout events'); this skill deliberately preserves
  genuine zeros as leave-alone
- When the input is dense or contains no zero-valued entries, since classification
  operates specifically on zero entries per gene
- When the matrix is already imputed or transformed such that dropout zeros are no
  longer distinguishable as zeros
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Implements the hypergeometric-testing first module that segregates the dataset
    into training data and imputable data
  repo: https://github.com/duct317/scISR
- name: stats (R package)
  role: Provides the hypergeometric testing functions (e.g. phyper) used for the per-gene
    under-/over-representation tests
- name: R
  role: Runtime environment in which the scISR package and its hypothesis-testing
    module execute
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:3a753b276ecf9be5929e01ed34a696f3613457d6a0bc64d99bfc1f367248d2e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R
  - scISR
  - stats (R package)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# pvalue-threshold-classification

## Summary

Classifies zero-valued entries of a scRNA-seq expression matrix as trustworthy, imputable, or leave-alone by per-gene hypergeometric tests (under-representation and over-representation) with a p-value significance threshold of 0.01, as implemented in scISR's first module.

## When to use

Use when a sparse scRNA-seq count/FPKM matrix contains zero entries that must be partitioned into genuine biological zeros versus dropout-affected zeros before imputation or regression-based value recovery.

## When NOT to use

- When the goal is to impute ALL zero-valued entries indiscriminately (the article criticizes methods that 'attempt to alter the expression of all zero-valued entries, including those not affected by dropout events'); this skill deliberately preserves genuine zeros as leave-alone
- When the input is dense or contains no zero-valued entries, since classification operates specifically on zero entries per gene
- When the matrix is already imputed or transformed such that dropout zeros are no longer distinguishable as zeros

## Inputs

- scRNA-seq expression matrix (genes x cells) with zero-valued entries, e.g. Goolam dataset counts from scISR
- Preprocessed matrix after log2 transformation (base 2) for value ranges larger than 100 and gene filtering

## Outputs

- Per-gene/per-entry zero classification: trustworthy (training data), imputable (dropout-affected), and leave-alone labels, e.g. gene_partition.csv
- Segregated training-data and imputable-data gene sets used by the downstream subspace regression imputation

## How to apply

Load and preprocess the expression matrix: log2-transform (base 2) when the value range exceeds 100, and remove genes expressed in fewer than two cells or with less than one percent non-zero entries. For each gene, tally its zero and non-zero entries and compute the hypergeometric p-value for the observed number of zeros using the overlap-corrected sample size n+m-1 via R stats::phyper. Run an under-representation test per gene: zeros with significant p-values below 0.01 are labeled trustworthy (genuine zeros) and form the training data. Run an over-representation test per gene: zeros with significant p-values below 0.01 are labeled imputable (likely dropout-affected), and all remaining non-significant zeros are labeled leave-alone so downstream imputation does not modify them.

## Related tools

- **scISR** (Implements the hypergeometric-testing first module that segregates the dataset into training data and imputable data) — https://github.com/duct317/scISR
- **stats (R package)** (Provides the hypergeometric testing functions (e.g. phyper) used for the per-gene under-/over-representation tests)
- **R** (Runtime environment in which the scISR package and its hypothesis-testing module execute)

## Examples

```
install.packages('scISR'); library(scISR); data('Goolam'); imputed <- scISR(data = Goolam$data)
```

## Evaluation signals

- Every zero-valued entry receives exactly one of three labels (trustworthy, imputable, leave-alone); no zero is left unclassified and non-zero entries are never relabeled
- A zero is labeled imputable only if the over-representation p-value is below 0.01, and trustworthy only if the under-representation p-value is below 0.01; check the recorded p-values against these cutoffs
- Trustworthy entries form the training-data gene set and imputable entries the imputable-data set; the two sets should be disjoint and their union with leave-alone entries should cover all zeros
- Genes expressed in fewer than two cells or with less than one percent non-zero entries must have been removed before testing
- Leave-alone labels must correspond to non-significant p-values (>= 0.01), ensuring these genuine zeros are not modified by downstream imputation

## Limitations

- R stats package and scISR software versions are not stated, limiting reproducibility
- The method only distinguishes zeros statistically per gene; misclassification of borderline dropout vs. genuine zeros is possible for genes with few entries
- Preprocessing thresholds (log2 when range > 100, genes in < 2 cells or < 1% non-zero) must match the original implementation or the p-value landscape changes

## Evidence

- [discussion] An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable).: "An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable)."
- [discussion] An entry with a significant p-value in the under-representation analysis is considered trustworthy.: "An entry with a significant p-value in the under-representation analysis is considered trustworthy."
- [readme] scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach.: "scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach."
- [discussion] This section describes the first module in scISR which aims at determining whether each zero value observed is the result of dropouts.: "This section describes the first module in scISR which aims at determining whether each zero value observed is the result of dropouts."
