---
name: scrna-seq-dataset-preprocessing
description: 'Use when raw or published scRNA-seq count/TPM matrices must be normalized
  before clustering: values exceed 100, low-level expressed genes are present, and
  downstream feature selection needs a comparable scale.'
when_to_use_negative:
- Input is already log-normalized with all expression values ≤ 100 and low-expression
  genes already filtered — reapplying log2 would distort the data
- Analysis goal is full-length transcript/trajectory quantification rather than producing
  a matrix for ensemble feature selection and meta-clustering
- Input is single-cell ATAC or other non-gene-expression data where the 2% cell-detection
  gene filter is not meaningful
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEFSC
  role: Pipeline whose preprocessing stage performs the log2 transformation and <2%
    gene-detection filtering on the fourteen scRNA-seq datasets
  repo: https://github.com/Conan-Bian/scEFSC
- name: SHARP
  role: Example downstream consumer whose own preprocessing removes all-zero genes
    and applies log-transform; contrasts with scEFSC's fixed thresholds
  repo: github.com/shibiaowan/SHARP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:86389ce70c12c135e0db8c24f73955e0414c953d60d85c313c62b8d5d2b5998f
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
  - scEFSC
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# scrna-seq-dataset-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess published scRNA-seq expression matrices (e.g. the fourteen GSE/E-MTAB datasets used by scEFSC) by log2 rescaling and low-expression gene filtering so they are suitable for downstream ensemble feature selection and clustering.

## When to use

Use when raw or published scRNA-seq count/TPM matrices must be normalized before clustering: values exceed 100, low-level expressed genes are present, and downstream feature selection needs a comparable scale.

## When NOT to use

- Input is already log-normalized with all expression values ≤ 100 and low-expression genes already filtered — reapplying log2 would distort the data
- Analysis goal is full-length transcript/trajectory quantification rather than producing a matrix for ensemble feature selection and meta-clustering
- Input is single-cell ATAC or other non-gene-expression data where the 2% cell-detection gene filter is not meaningful

## Inputs

- scRNA-seq gene-by-cell expression matrix (rows = genes, columns = samples/cells), e.g. from GEO/ArrayExpress accessions such as GSE36552, GSE83139, GSE81861, E-MTAB-5061
- Optional cell label file for removing cells annotated ambiguously as 'abandoned'

## Outputs

- Log2-normalized expression matrix with genes detected in <2% of cells removed
- Preprocessed matrix ready for non-negative kernel autoencoder 5000-gene pre-selection and ensemble feature selection

## How to apply

Start from an scRNA-seq expression matrix with rows as genes and columns as cells/samples. If gene expression values are higher than 100, apply a logarithmic transformation (base 2) to rescale the data. Then remove genes detected in the normalized data in fewer than 2% of the cells, to filter out low-level expressed genes. This normalized, filtered matrix is the input to downstream steps such as pre-selecting 5000 genes with a non-negative kernel autoencoder and running the Low Variance, Laplacian Score, SPEC, and MCFS unsupervised feature selections. Cells labeled ambiguously as 'abandoned' should be removed before evaluation against published labels.

## Related tools

- **scEFSC** (Pipeline whose preprocessing stage performs the log2 transformation and <2% gene-detection filtering on the fourteen scRNA-seq datasets) — https://github.com/Conan-Bian/scEFSC
- **SHARP** (Example downstream consumer whose own preprocessing removes all-zero genes and applies log-transform; contrasts with scEFSC's fixed thresholds) — github.com/shibiaowan/SHARP

## Examples

```
R: scExp <- readRDS("counts.rds"); idx <- colMeans(scExp > 0) >= 0.02; scExp <- log2(scExp[, idx])
```

## Evaluation signals

- Every gene in the output matrix is detected in ≥ 2% of cells (no low-level expressed genes remain)
- All expression values ≤ 100 after preprocessing (values > 100 trigger a log2 rescale during the run)
- Matrix orientation preserved: rows are genes, columns are samples/cells
- Cells ambiguously labeled 'abandoned' are absent from the preprocessed matrix used for ARI/NMI evaluation
- Downstream step can proceed: 5000 genes can be pre-selected from the filtered matrix with the non-negative kernel autoencoder

## Limitations

- The article does not report per-dataset preprocessing diagnostics in the results/discussion sections; the 2%-detection and log2(>100) rules are stated without sensitivity analysis in those sections
- SHARP's README cautions that log-transform is not always beneficial (it is skipped when the silhouette index of the non-log case is ≥ 0.75 and larger), so a fixed log2 rule may not suit every dataset
- Filtering genes detected in <2% of cells may remove rare cell-type markers expressed in very few cells

## Evidence
