---
name: hypergeometric-dropout-significance-testing
description: Use when an scRNA-seq count/normalized expression matrix contains many
  zero entries and you must decide which zeros are true biological zeros versus dropout
  artifacts before imputation.
when_to_use_negative:
- The zero entries are already annotated or the goal is plain normalization/clustering
  of the raw matrix without imputation (e.g. the README's ARI-on-raw-data baseline
  path)
- The dataset is not scRNA-seq (e.g. bulk RNA-seq, which the paper notes cannot measure
  cell-to-cell variability and is not the target of dropout classification)
- Non-zero missing values (NaNs) rather than encoded zero-valued entries are the data-quality
  problem
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scISR
  role: Package whose first module performs the hypergeometric significance testing
    that segregates the dataset into training data and imputable data
  repo: https://github.com/duct317/scISR
- name: stats (R package)
  role: Provides the hypergeometric distribution functions used to compute per-gene
    zero-enrichment p-values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:baa3b311480bf32a99112336dd642f1306f1b4a162b6f26d9b3dd6ce060691b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R
  - scISR
  - stats (R package)
  - stats (R package, hypergeometric testing)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hypergeometric-dropout-significance-testing

## Summary

Classifies zero-valued entries in a scRNA-seq expression matrix as trustworthy, imputable (dropout-affected), or leave-alone using per-gene hypergeometric tests of zero under- and over-representation, as implemented in scISR's first module.

## When to use

Use when an scRNA-seq count/normalized expression matrix contains many zero entries and you must decide which zeros are true biological zeros versus dropout artifacts before imputation.

## When NOT to use

- The zero entries are already annotated or the goal is plain normalization/clustering of the raw matrix without imputation (e.g. the README's ARI-on-raw-data baseline path)
- The dataset is not scRNA-seq (e.g. bulk RNA-seq, which the paper notes cannot measure cell-to-cell variability and is not the target of dropout classification)
- Non-zero missing values (NaNs) rather than encoded zero-valued entries are the data-quality problem

## Inputs

- scRNA-seq gene-by-cell expression matrix (counts or normalized values, e.g. FPKM/RPKM/CPM/TPM/RPM as in the paper's datasets)
- Optional: cell type labels for downstream ARI-based evaluation

## Outputs

- Per-entry zero-classification labels: trustworthy, imputable, leave-alone
- Training-data (trustworthy) and imputable-data gene sets, exportable as gene_partition.csv

## How to apply

First preprocess the expression matrix: log2-transform (base 2) when the value range exceeds 100, and remove genes expressed in fewer than two cells or with less than one percent non-zero entries. For each gene, tally zero-valued and non-zero entries and compute hypergeometric p-values (R stats::phyper) for the observed number of zeros, using the overlap-corrected sample size n+m−1. Run an under-representation test: zeros with p < 0.01 are labeled trustworthy (genuine zeros). Run an over-representation test: zeros with p < 0.01 are labeled imputable (likely dropout-affected); all remaining non-significant zeros are left alone. The rationale is that zeros enriched beyond expectation signal dropout, while zeros significantly under-represented signal true zero expression, so only the imputable set is passed to downstream regression imputation.

## Related tools

- **scISR** (Package whose first module performs the hypergeometric significance testing that segregates the dataset into training data and imputable data) — https://github.com/duct317/scISR
- **stats (R package)** (Provides the hypergeometric distribution functions used to compute per-gene zero-enrichment p-values)

## Examples

```
library(scISR); data('Goolam'); imputed <- scISR(data = Goolam$data)
```

## Evaluation signals

- Every zero entry in the input matrix receives exactly one label: trustworthy, imputable, or leave-alone — no zeros left unclassified
- Imputable and trustworthy sets are disjoint, and no non-zero entry is relabeled
- Per-gene p-values follow phyper with sample size n+m−1 (overlap-corrected), and the significance cutoff is 0.01 for both over- and under-representation tests
- Genes expressed in fewer than two cells, or with <1% non-zero entries, are absent from the partition
- Downstream check: k-means clustering on imputed data (true k) should improve adjusted Rand index relative to raw-data clustering, as in the README's Goolam example

## Limitations

- The R stats package and scISR software versions are not stated, limiting exact reproducibility
- The test treats zeros as the only dropout evidence; all-or-nothing behavior at very low sequencing depths may confound the enrichment tests
- Non-significant zeros are left untouched (leave-alone), so dropouts below the 0.01 significance threshold will not be imputed

## Evidence

- [discussion] An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable).: "An entry with a significant p-value in the over-representation analysis is considered untrustworthy and should be imputed (imputable)."
- [discussion] An entry with a significant p-value in the under-representation analysis is considered trustworthy.: "An entry with a significant p-value in the under-representation analysis is considered trustworthy."
- [abstract] The scISR method first uses a hypothesis-testing technique to identify zero-valued entries that are most likely affected by dropout events: "The scISR method first uses a hypothesis-testing technique to identify zero-valued entries that are most likely affected by dropout events"
- [readme] scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach.: "scISR identifies the true dropout values in the scRNA-seq dataset using hyper-geomtric testing approach."
- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
