---
name: scrna-seq-zero-entry-partitioning
description: Use when preprocessing a sparse scRNA-seq count matrix before imputation
  and you need to decide, per zero entry, whether it is a dropout event or a true
  zero rather than imputing all zeros indiscriminately.
when_to_use_negative:
- The matrix has already been imputed or the zeros have already been classified —
  re-running the hypergeometric partitioning would double-process entries.
- The goal is bulk RNA-seq or dense non-single-cell data where the sparse dropout
  structure being tested does not exist.
- Existing imputation methods that alter all zero-valued entries (including true zeros)
  are preferred and per-entry dropout evidence is not required.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scISR
  role: Implements the hypergeometric-testing first module that identifies true dropout
    values and segregates the dataset into training data and imputable data
  repo: https://github.com/duct317/scISR
- name: stats (R package)
  role: Provides the hypergeometric distribution functions (e.g. phyper) used for
    the per-gene under-/over-representation tests
- name: R
  role: Execution environment in which the scISR package and stats hypergeometric
    testing are run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:1f955f9b417dfd1f1e09e8bb7483b1f6cc833a06c78c3ad549877cb649baee8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R
  - scISR
  - stats (R package)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# scrna-seq-zero-entry-partitioning

## Summary

A hypergeometric hypothesis-testing skill (from scISR's first module) that classifies each zero-valued entry in an scRNA-seq expression matrix as trustworthy (genuine zero), imputable (dropout-affected), or leave-alone, so imputation only targets entries supported by evidence of dropout.

## When to use

Use when preprocessing a sparse scRNA-seq count matrix before imputation and you need to decide, per zero entry, whether it is a dropout event or a true zero rather than imputing all zeros indiscriminately.

## When NOT to use

- The matrix has already been imputed or the zeros have already been classified — re-running the hypergeometric partitioning would double-process entries.
- The goal is bulk RNA-seq or dense non-single-cell data where the sparse dropout structure being tested does not exist.
- Existing imputation methods that alter all zero-valued entries (including true zeros) are preferred and per-entry dropout evidence is not required.

## Inputs

- scRNA-seq expression matrix (genes x cells; raw or normalized counts, e.g. FPKM/RPKM/TPM/CPM/RPM)
- Optional gene/cell metadata (e.g. cell type labels from datasets like Goolam E-MTAB-3321)

## Outputs

- Per-gene/per-entry zero labels partitioning zeros into trustworthy, imputable, and leave-alone sets (e.g. gene_partition.csv)
- Training-data gene set (trustworthy zeros) for downstream regression imputation
- Imputable-data gene set (dropout-affected zeros) targeted for imputation

## How to apply

Load the scRNA-seq expression matrix and preprocess it: log2-transform (base 2) if the value range exceeds 100, and remove genes expressed in fewer than two cells or with less than one percent non-zero entries. For each gene, tally zero and non-zero entries and compute a hypergeometric p-value for the observed number of zeros using the overlap-corrected sample size n+m−1 (R stats::phyper). Run an under-representation test per gene: zero entries with p-value below 0.01 are labeled trustworthy (genuine zeros) and form the training data. Run an over-representation test per gene: zero entries with significant p-value below 0.01 are labeled imputable (dropout-affected). Label all remaining non-significant zeros as leave-alone so downstream imputation does not modify them, and write per-gene/per-entry labels (e.g., to gene_partition.csv).

## Related tools

- **scISR** (Implements the hypergeometric-testing first module that identifies true dropout values and segregates the dataset into training data and imputable data) — https://github.com/duct317/scISR
- **stats (R package)** (Provides the hypergeometric distribution functions (e.g. phyper) used for the per-gene under-/over-representation tests)
- **R** (Execution environment in which the scISR package and stats hypergeometric testing are run)

## Examples

```
library(scISR); imputed <- scISR(data = raw)
```

## Evaluation signals

- Every zero entry receives exactly one of three labels: trustworthy, imputable, or leave-alone; no zeros remain unlabeled.
- Only entries with hypergeometric p-value below 0.01 are assigned trustworthy (under-representation) or imputable (over-representation) status; all others default to leave-alone.
- Downstream imputation modifies only imputable-labeled entries; leave-alone and trustworthy zeros retain their original zero values.
- On masked simulations (30% of values zeroed in 40% of genes), the labeling recovers a substantial fraction of the artificially introduced dropout zeros.
- The partition supports improved clustering: k-means on PCA (e.g. irlba::prcomp_irlba, n = 50) of downstream-imputed data should yield higher adjusted Rand index (ARI) than on raw data.

## Limitations

- Only zero entries with significant p-values (p < 0.01) are classified; non-significant zeros are left alone and never imputed, so borderline dropouts may be missed.
- A p-value alone may mislabel entries: over-imputation of true zeros (as in prior methods that alter all zeros) introduces false signals, and this test only partially mitigates that risk.
- Robustness evidence is deferred to supplementary figures (S19, S21, S24, S5) not included in the main text, and the 25 real scRNA-seq datasets used in evaluation are not fully named in the discussion section.

## Evidence

- [discussion] An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable).: "An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable)."
- [discussion] An entry with a significant p-value in the under-representation analysis is considered trustworthy.: "An entry with a significant p-value in the under-representation analysis is considered trustworthy."
- [readme] scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach.: "scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach. Based on the result obtained from hyper-geometric testing, the original dataset is segregated"
- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] (i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries.: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
