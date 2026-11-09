---
name: sc-rna-seq-data-preprocessing
description: Use when loading raw scRNA-seq count data (e.g. .h5 matrices with cells×genes
  layout) as input to scCAD or its HVG/HIG ensemble feature selection, and the matrix
  has not yet been gene-filtered or log-normalized.
when_to_use_negative:
- Input is already normalized or gene-filtered expression data — passing normalization=True
  again would double-transform the log2-scaled values
- Bulk RNA-seq, proteomics, or spatial transcriptomics data where the 'expressed in
  at least three cells' filter and cell-specific size factors do not apply
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAD
  role: Consumes the preprocessed cells×genes matrix; its normalization flag (default
    True) performs the log-normalization with cell-specific size factors internally
  repo: https://github.com/xuyp-csu/scCAD
- name: Seurat
  role: Reference implementation of the log-normalization procedure (cell-specific
    size factors) and of the mean-vs-dispersion HVG strategy applied downstream
- name: Scanpy
  role: Pinned prerequisite (v1.9.1) in the scCAD environment, providing standard
    preprocessing utilities (gene filtering, normalization) in the Python stack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:9e46da05cf4cd3f9a683da7d60df0a3bb681476840b4b963a0d2fdebc3b7a6df
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - PyTorch
  - SAMap
  - Scanpy
  - Seurat
  - TACTiCS
  - scCAD
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- sc-rna-seq-preprocessing
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# sc-rna-seq-data-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw scRNA-seq count matrices the way scCAD expects: retain only genes expressed in at least three cells, then log-normalize with cell-specific size factors and a log2(x+1) transform so downstream rare-cell detection (and ensemble feature selection) operates on comparable expression scales.

## When to use

Use when loading raw scRNA-seq count data (e.g. .h5 matrices with cells×genes layout) as input to scCAD or its HVG/HIG ensemble feature selection, and the matrix has not yet been gene-filtered or log-normalized.

## When NOT to use

- Input is already normalized or gene-filtered expression data — passing normalization=True again would double-transform the log2-scaled values
- Bulk RNA-seq, proteomics, or spatial transcriptomics data where the 'expressed in at least three cells' filter and cell-specific size factors do not apply

## Inputs

- Raw scRNA-seq count matrix, cells in rows and genes in columns (e.g. 'X' array inside an .h5 file such as 1%Jurkat.h5)
- Optional gene-name and cell-name arrays ('gn', 'cn' entries in the .h5 file)

## Outputs

- Gene-filtered, log2(x+1)-normalized expression matrix (cells × genes) suitable for PCA (top 40 PCs), KNN graph (k=15), Louvain clustering, and HVG/HIG ensemble feature selection

## How to apply

Load the expression matrix with genes in columns and cells in rows (e.g. read the 'X' dataset from an .h5 file via h5py, as in the scCAD demo). First filter out genes expressed in fewer than three cells; scCAD retains 'genes that are expressed in at least three cells for downstream analysis'. Next apply log-normalization: compute cell-specific size factors, scale counts, add a pseudo-count of 1, and take the log2 transform (log2(x+1)). These steps match the default normalization flag in scCAD.scCAD (normalization=True), so pass raw counts and let the function normalize, or pre-normalize identically and pass normalization=False. Keep gene and cell name arrays alongside the matrix, since downstream HVG selection (top 2000 mean-vs-dispersion) and random-forest HIG selection (top 2000 Gini importance on cluster labels) operate on gene names.

## Related tools

- **scCAD** (Consumes the preprocessed cells×genes matrix; its normalization flag (default True) performs the log-normalization with cell-specific size factors internally) — https://github.com/xuyp-csu/scCAD
- **Seurat** (Reference implementation of the log-normalization procedure (cell-specific size factors) and of the mean-vs-dispersion HVG strategy applied downstream)
- **Scanpy** (Pinned prerequisite (v1.9.1) in the scCAD environment, providing standard preprocessing utilities (gene filtering, normalization) in the Python stack)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('raw.h5ad'); sc.pp.filter_genes(adata, min_cells=3); sc.pp.normalize_total(adata); sc.pp.log1p(adata, base=2)
```

## Evaluation signals

- After filtering, every retained gene is detected in ≥3 cells; genes expressed in 1–2 cells are absent from the matrix
- Normalized values are non-negative and follow log2(x+1) scaling (0 for zero counts, bounded by log2(max count per cell + 1))
- Each cell's normalized profile reflects size-factor scaling, so total normalized counts are comparable across cells
- Downstream reproducibility check: the pipeline reproduces scCAD's Supplementary Table 8 average OR1 overlap rates (~86.75% scCAD, ~67.80% HVG, ~80.20% HIG) on benchmark datasets
- Demo smoke test: scCAD on the bundled 1%Jurkat.h5 (1540 293T + 16 Jurkat cells) completes in ~40 seconds with normalization=True

## Limitations

- The 'at least three cells' gene filter can remove very rare marker genes; scCAD's ensemble feature selection is designed to compensate by preserving rare cell type-specific genes (average OR1 86.75%), but extremely sparse signatures may still be lost
- Double normalization risk: scCAD.scCAD normalizes by default (normalization=True); passing already-normalized data without setting normalization=False distorts the log2(x+1) scale
- Requires in-memory matrix operations; very large datasets need enough RAM on a standard computer

## Evidence
