---
name: scrna-seq-count-matrix-preprocessing
description: Use when a raw scRNA-seq expression matrix (genes × cells) must be normalized
  and filtered before feature selection or clustering, following the scEFSC protocol.
when_to_use_negative:
- Input is already a normalized feature-selected gene subset (e.g. output of the non-negative
  kernel autoencoder pre-selection)
- The downstream tool performs its own preprocessing, e.g. SHARP which removes all-zero
  genes and log-transforms internally by default
- Data are not a gene-by-cell scRNA-seq count/expression matrix
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEFSC
  role: Pipeline whose first stage defines and consumes this preprocessing (log2 normalization,
    2% detection filter)
  repo: https://github.com/Conan-Bian/scEFSC
- name: SHARP
  role: Downstream clustering algorithm (v1.1.0) that receives the preprocessed matrix;
    it also applies its own removal of all-zero genes and log-transform
  repo: https://github.com/shibiaowan/SHARP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:42e4a9652c888f8cc3f79c4318c6e328161779c77af67cdfbb0a6966d6a4e437
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - CIDR
  - Monocle
  - PhenoGraph
  - RaceID
  - SC3
  - SHARP
  - Seurat
  - scDHA
  - scEFSC
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# scRNA-seq-count-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess scRNA-seq expression matrices for the scEFSC ensemble-clustering pipeline: log2 normalization, removal of low-level expressed genes detected in less than 2% of cells, dropping ambiguously labeled 'abandoned' cells, and rescaling with log2 when expression values exceed 100.

## When to use

Use when a raw scRNA-seq expression matrix (genes × cells) must be normalized and filtered before feature selection or clustering, following the scEFSC protocol.

## When NOT to use

- Input is already a normalized feature-selected gene subset (e.g. output of the non-negative kernel autoencoder pre-selection)
- The downstream tool performs its own preprocessing, e.g. SHARP which removes all-zero genes and log-transforms internally by default
- Data are not a gene-by-cell scRNA-seq count/expression matrix

## Inputs

- scRNA-seq expression matrix with rows as genes and columns as samples (cell labels may contain an 'abandoned' category)

## Outputs

- Normalized, filtered expression matrix (low-expression genes removed, 'abandoned' cells dropped, values log2-rescaled where >100) ready for 5000-gene autoencoder pre-selection

## How to apply

Start from an scRNA-seq expression matrix with rows as genes and columns as samples, then apply a log2 transformation to normalize the data. Remove genes detected in the normalized data in less than 2% of the cells to filter out low-level expressed genes. Drop cells labeled ambiguously as 'abandoned'. If gene expression values are higher than 100, rescale the data with a base-2 logarithmic transformation. The cleaned matrix then feeds the non-negative kernel autoencoder that pre-selects 5000 genes for downstream scEFSC analysis.

## Related tools

- **scEFSC** (Pipeline whose first stage defines and consumes this preprocessing (log2 normalization, 2% detection filter)) — https://github.com/Conan-Bian/scEFSC
- **SHARP** (Downstream clustering algorithm (v1.1.0) that receives the preprocessed matrix; it also applies its own removal of all-zero genes and log-transform) — https://github.com/shibiaowan/SHARP

## Examples

```
dat <- log2(raw_counts + 1); keep <- rowMeans(dat > 0) >= 0.02; dat <- dat[keep, ]; dat <- dat[, colnames(dat) != "abandoned"]
```

## Evaluation signals

- Genes detected in less than 2% of cells after log2 normalization are absent from the output matrix
- No cells labeled 'abandoned' remain in the preprocessed matrix
- All expression values are ≤ 100 (values >100 have been log2-rescaled)
- The output matrix retains genes-by-cells orientation and can be passed directly to the non-negative kernel autoencoder to pre-select 5000 genes

## Limitations

- The 2% detection threshold and the >100 log2 rescaling rule are dataset-independent defaults from the scEFSC paper and may not suit very sparse UMI data
- Cells must carry explicit 'abandoned' labels for that filtering step to apply; otherwise no ambiguous-cell removal occurs

## Evidence
