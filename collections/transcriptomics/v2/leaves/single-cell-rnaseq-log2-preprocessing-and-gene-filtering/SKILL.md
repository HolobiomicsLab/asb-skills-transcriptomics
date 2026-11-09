---
name: single-cell-rnaseq-log2-preprocessing-and-gene-filtering
description: Use when raw or low-level-normalized scRNA-seq count/TPM matrices feed
  an ensemble clustering or feature-selection pipeline and need normalization plus
  removal of low-level expressed genes.
when_to_use_negative:
- Data are single-cell ATAC-seq, proteomics, or bulk RNA-seq — the <2% cell-detection
  threshold is calibrated for scRNA-seq gene dropout structure
- The matrix is already log2-normalized with low-detection genes removed; re-applying
  the transformation and filter would double-normalize and discard no additional genes
- A supervised feature selection or differential-expression-driven gene filter is
  required — this is an unsupervised pre-filter
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scEFSC
  role: Pipeline that implements the log2 preprocessing, <2% gene filtering, and non-negative
    kernel autoencoder pre-selection of 5000 genes before ensemble feature selection
    and clustering
  repo: https://github.com/Conan-Bian/scEFSC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:086588a7a6bc9d632125bd6012af7686e6fb308e792c4baa9ccd3231b6259acb
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
  - RaceID
  - Rphenograph
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - pcaReduce
  - scEFSC
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# single-cell-rnaseq-log2-preprocessing-and-gene-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-processing step from scEFSC that log2-normalizes an n × d scRNA-seq expression matrix (rescaling with log2 only when expression values exceed 100) and removes genes detected in fewer than 2% of cells, yielding a denoised matrix ready for downstream feature selection and clustering.

## When to use

Use when raw or low-level-normalized scRNA-seq count/TPM matrices feed an ensemble clustering or feature-selection pipeline and need normalization plus removal of low-level expressed genes.

## When NOT to use

- Data are single-cell ATAC-seq, proteomics, or bulk RNA-seq — the <2% cell-detection threshold is calibrated for scRNA-seq gene dropout structure
- The matrix is already log2-normalized with low-detection genes removed; re-applying the transformation and filter would double-normalize and discard no additional genes
- A supervised feature selection or differential-expression-driven gene filter is required — this is an unsupervised pre-filter

## Inputs

- scRNA-seq expression matrix (rows = genes, columns = cells/samples), e.g. CSV of counts or TPM values from datasets such as GSE36552, GSE83139, GSE81608
- Optional known cell-type label file for later validation

## Outputs

- Log2-normalized gene × cell matrix with genes detected in <2% of cells removed
- Pre-selected matrix of 5000 genes (non-negative kernel autoencoder) suitable for downstream feature selection

## How to apply

Load the scRNA-seq data with rows as genes and columns as samples/cells. If gene expression values are higher than 100, rescale the data with a base-2 logarithmic transformation to normalize the data; otherwise leave values as-is. After normalization, compute per-gene detection rates and remove any gene detected in less than 2% of the cells to filter out low-level expressed genes. Optionally, as scEFSC does next, pre-select 5000 genes with a single-layer non-negative kernel autoencoder, retaining genes with large encoder-weight variances, before handing the filtered matrix to unsupervised feature selectors (Low Variance, Laplacian Score, SPEC, MCFS). The rationale is that clustering algorithms assume all features are equally significant, so low-detection and insignificant genes must be removed early to avoid degrading the ensemble consensus clustering.

## Related tools

- **scEFSC** (Pipeline that implements the log2 preprocessing, <2% gene filtering, and non-negative kernel autoencoder pre-selection of 5000 genes before ensemble feature selection and clustering) — https://github.com/Conan-Bian/scEFSC

## Examples

```
data.norm <- log2(raw); keep <- rowMeans(data.norm > 0) >= 0.02; filtered <- data.norm[keep, ]
```

## Evaluation signals

- After filtering, every retained gene is detected in at least 2% of the cells (no gene with detection rate < 0.02 remains)
- Matrix values are on log2 scale only where the original expression values exceeded 100 — check max(raw values) before transformation
- Pre-selection yields exactly 5000 genes with large encoder-weight variances from the non-negative kernel autoencoder
- Downstream feature selection (Low Variance, Laplacian Score, SPEC, MCFS) runs without errors on the filtered matrix
- Final scEFSC labels achieve non-degenerate NMI/ARI against published cell-type labels (e.g. best NMI on 11 of 14 datasets in the paper)

## Limitations

- The 2% detection cutoff and 5000-gene pre-selection were tuned on fourteen real scRNA-seq datasets; other scRNA-seq data types or ultra-sparse technologies may need adjusted thresholds
- Cells labeled ambiguously as 'abandoned' must be removed separately — the gene filter does not handle ambiguous cell labels
- The log2 transformation is conditional (only when values exceed 100), so blindly applying log2 can distort already-normalized low-value matrices

## Evidence
