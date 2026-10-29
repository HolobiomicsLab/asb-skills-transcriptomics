---
name: single-cell-rna-seq-preprocessing-normalization
description: Use when a raw scRNA-seq count matrix (genes x cells or cells x genes,
  e.g. .h5) needs filtering and log-normalization before running rare cell detection
  methods such as scCAD, CellSIUS, EDGE, FiRE, or SCA.
when_to_use_negative:
- The input is already normalized (e.g. log-transformed data loaded via a preprocessed
  .h5ad) — re-running normalization would double-transform; pass normalization=False
  to scCAD
- The goal is bulk RNA-seq or spatial transcriptomics normalization rather than single-cell
  expression matrices
- The analysis requires raw counts, e.g. methods that model counts directly with negative
  binomial likelihoods
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scCAD
  role: Main tool whose default pipeline applies gene filtering and log-normalization
    with cell-specific size factors (normalization=True) before cluster decomposition
  repo: github.com/xuyp-csu/scCAD
- name: Seurat
  role: Used to preprocess matrices for comparator methods CellSIUS and EDGE in the
    benchmark, applying the same normalization strategy
- name: Scanpy
  role: Dependency (v1.9.1) of the scCAD environment used for single-cell preprocessing
    operations
  repo: github.com/xuyp-csu/scCAD
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:24fb18686bb687fdd8aae57b42a9f50f5f3a3174fb81dd4c439492df5433a561
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  tools:
  - CIARA
  - CellSIUS
  - EDGE
  - FiRE
  - GapClust
  - GiniClust
  - Python
  - RaceID
  - SCA
  - SCISSORS
  - Scanpy
  - Seurat
  - scCAD
  - scLearn
  - scNovel
  - scPred
  - scmap-cell
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-rna-seq-normalization-preprocessing
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# single-cell-rna-seq-preprocessing-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw scRNA-seq count matrices for rare cell type identification by filtering genes expressed in fewer than three cells and applying log-normalization with cell-specific size factors followed by a log2 transform with pseudo-count 1, matching the preprocessing used in the scCAD 25-dataset benchmark.

## When to use

Use when a raw scRNA-seq count matrix (genes x cells or cells x genes, e.g. .h5) needs filtering and log-normalization before running rare cell detection methods such as scCAD, CellSIUS, EDGE, FiRE, or SCA.

## When NOT to use

- The input is already normalized (e.g. log-transformed data loaded via a preprocessed .h5ad) — re-running normalization would double-transform; pass normalization=False to scCAD
- The goal is bulk RNA-seq or spatial transcriptomics normalization rather than single-cell expression matrices
- The analysis requires raw counts, e.g. methods that model counts directly with negative binomial likelihoods

## Inputs

- Raw scRNA-seq count matrix with genes in columns and cells in rows (e.g. numpy array read from an .h5 file with 'X', 'Y', 'gn', 'cn' fields)
- Optional gene name and cell name vectors

## Outputs

- Gene-filtered count matrix retaining only genes expressed in at least three cells
- Log-normalized expression matrix (cell-specific size factors, log2(x+1) transform) suitable for scCAD and comparator methods
- Normalized matrices for downstream PCA (top 40 PCs) and Louvain clustering

## How to apply

Start from the raw count matrix and retain only genes that are expressed in at least three cells, dropping sparsely detected genes before normalization. Normalize with the log-normalization procedure including the calculation of cell-specific size factors, then apply a log2 transform after adding a pseudo-count of 1. When calling scCAD, this normalization is applied by default (normalization=True), so set the flag accordingly only if the matrix has not already been normalized. Downstream, PCA to the top 40 principal components with a k=15 nearest-neighbor graph and Louvain clustering assumes these normalized values. The identical preprocessing was applied across all 25 real benchmark datasets so that scCAD and the ten comparator methods (Seurat-preprocessed CellSIUS/EDGE, FiRE, SCA, etc.) were evaluated on the same input.

## Related tools

- **scCAD** (Main tool whose default pipeline applies gene filtering and log-normalization with cell-specific size factors (normalization=True) before cluster decomposition) — github.com/xuyp-csu/scCAD
- **Seurat** (Used to preprocess matrices for comparator methods CellSIUS and EDGE in the benchmark, applying the same normalization strategy)
- **Scanpy** (Dependency (v1.9.1) of the scCAD environment used for single-cell preprocessing operations) — github.com/xuyp-csu/scCAD

## Examples

```
data_mat = h5py.File('./1%Jurkat.h5'); data = np.array(data_mat['X']); result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', normalization=True, save_path='./')
```

## Evaluation signals

- After filtering, every retained gene is expressed in at least three cells (genes expressed in fewer than three cells are absent from the matrix)
- Normalized values are log2-transformed with a pseudo-count of 1, so all entries are non-negative and the minimum possible value is 0
- Size factors differ across cells (cell-specific), not a single global scale factor
- Running scCAD with normalization=True on already-normalized data degrades results; verify the normalization flag matches the input state
- Benchmark reproduction: the pipeline yields scCAD F1 = 0.4172 across the 25 real datasets with rare-type success in 20/25, consistent with the reported table

## Limitations

- Filtering genes expressed in fewer than three cells discards very rare markers that may be biologically meaningful for the rarest populations
- Exact version numbers for most comparator packages (CellSIUS, CIARA, EDGE, FiRE, GapClust, GiniClust2/3, RaceID3, SCISSORS) are not stated, so exact preprocessing reproducibility across tools is not guaranteed
- scCAD is supported only on Linux and macOS and requires in-memory operations; very large matrices are limited by RAM

## Evidence

- [discussion] PCA top 40 PCs and k=15 KNN graph: [evidence span withheld — anchor status: not_found]
