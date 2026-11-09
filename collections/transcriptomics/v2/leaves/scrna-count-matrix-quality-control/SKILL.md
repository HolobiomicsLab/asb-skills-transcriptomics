---
name: scrna-count-matrix-quality-control
description: 'Use when a raw scRNA-seq count matrix (CSV or JLD2) must be cleaned
  before embedding: sparse genes, low-content cells, or high mitochondrial-read cells
  would otherwise distort downstream PCA/UMAP signal detection.'
when_to_use_negative:
- Input is not a raw count matrix (e.g., already normalized, imputed, or feature-selected
  data) — scLENS preprocessing assumes raw counts and the paper warns high-variance
  gene selection and imputation can distort or falsify signals
- Data is single-cell multiomic or spatial beyond scRNA-seq gene-by-cell count matrices
  — scLENS.read_file expects CSV or jld2 count tables
- A different QC scheme is mandated (e.g., required cell/gene thresholds differ) —
  the 200-gene / 15-cell cutoffs are hardcoded defaults of scLENS.preprocess
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Julia package whose preprocess/read_file/tenx2jld2 functions implement count-matrix
    loading and QC filtering before dimensionality reduction
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python companion providing the conda environment (environment.yml + PyTorch
    cu128) used to run scLENS with GPU or CPU execution
  repo: https://github.com/Mathbiomed/scLENSpy
- name: CUDA
  role: 'Device selection: CUDA.has_cuda() decides whether scLENS runs on ''gpu''
    or falls back to ''cpu'''
- name: CSV / DataFrames
  role: Used to read and write count matrices and results (e.g., CSV.write of pca.csv/umap.csv)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:1411e7f767717559eb294ac9589ef7ebd0ed7eeb7495f77d78b3dd09d6787740
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - CSV
  - CSV / DataFrames
  - CUDA
  - DataFrames
  - SparseArrays.jl
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scrna-count-matrix-quality-control

## Summary

Quality control of a scRNA-seq count matrix via the scLENS preprocess step, which filters low-quality cells and sparsely expressed genes (cells with <200 expressed genes, genes expressed in <15 cells, and, in benchmark workflows, cells with >5% mitochondrial reads) before dimensionality reduction.

## When to use

Use when a raw scRNA-seq count matrix (CSV or JLD2) must be cleaned before embedding: sparse genes, low-content cells, or high mitochondrial-read cells would otherwise distort downstream PCA/UMAP signal detection.

## When NOT to use

- Input is not a raw count matrix (e.g., already normalized, imputed, or feature-selected data) — scLENS preprocessing assumes raw counts and the paper warns high-variance gene selection and imputation can distort or falsify signals
- Data is single-cell multiomic or spatial beyond scRNA-seq gene-by-cell count matrices — scLENS.read_file expects CSV or jld2 count tables
- A different QC scheme is mandated (e.g., required cell/gene thresholds differ) — the 200-gene / 15-cell cutoffs are hardcoded defaults of scLENS.preprocess

## Inputs

- scRNA-seq count matrix as CSV (rows=cells, columns=gene names/IDs, first row gene names, first column cell IDs)
- scRNA-seq count matrix as JLD2 file containing a DataFrame with a :cell column of cell IDs
- 10x GZ files (matrix.mtx.gz, features.tsv.gz, barcodes.tsv.gz) convertible via scLENS.tenx2jld2

## Outputs

- QC-filtered count DataFrame (pre_df) with low-content cells and sparse genes removed
- Downstream scLENS embedding object from which PCA/UMAP CSVs, MP distribution and stability plots, and AnnData (.h5ad) files can be exported

## How to apply

Load the count matrix with scLENS.read_file (CSV with cells as rows and genes as columns, first column = cell IDs, or a JLD2 DataFrame with a :cell column), then call scLENS.preprocess on the DataFrame. The preprocess step filters out cells expressing fewer than 200 genes and genes expressed in fewer than 15 cells; benchmark workflows for the scLENS-med comparison additionally drop cells with more than 5% mitochondrial reads. Rationale: skewed, dropout-heavy distributions (>90% zeros are common) make QC essential before log normalization and RMT-based noise filtering in scLENS. The cleaned DataFrame feeds directly into scLENS.sclens(), with device selection via CUDA.has_cuda() choosing 'gpu' when available and 'cpu' otherwise. For 10x gz data, first convert matrix.mtx.gz/features.tsv.gz/barcodes.tsv.gz to JLD2 with scLENS.tenx2jld2 before QC.

## Related tools

- **scLENS** (Julia package whose preprocess/read_file/tenx2jld2 functions implement count-matrix loading and QC filtering before dimensionality reduction) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python companion providing the conda environment (environment.yml + PyTorch cu128) used to run scLENS with GPU or CPU execution) — https://github.com/Mathbiomed/scLENSpy
- **CUDA** (Device selection: CUDA.has_cuda() decides whether scLENS runs on 'gpu' or falls back to 'cpu')
- **CSV / DataFrames** (Used to read and write count matrices and results (e.g., CSV.write of pca.csv/umap.csv))

## Examples

```
using scLENS; ndf = scLENS.read_file("data/Z8eq.csv.gz"); pre_df = scLENS.preprocess(ndf); emb = scLENS.sclens(pre_df, device_=(CUDA.has_cuda() ? "gpu" : "cpu"))
```

## Evaluation signals

- preprocess returns a DataFrame whose cells each express ≥200 genes and whose genes are each expressed in ≥15 cells
- QC output retains a :cell column with cell IDs as required for downstream scLENS.sclens()
- Cells with >5% mitochondrial reads are excluded in benchmark settings (scLENS-med workflow)
- Downstream scLENS run completes with device selected by CUDA.has_cuda() ('gpu' if CUDA present, else 'cpu'), confirming the QC'd matrix is well-formed
- Saved results (pca.csv, umap.csv, or .h5ad via save_anndata) have row counts matching the QC-filtered cell set

## Limitations

- The preprocess defaults (cells <200 expressed genes, genes <15 cells) are fixed; cells with >5% mitochondrial reads filtering is a workflow-level addition, not a README-documented default
- Highly sparse datasets (>90% zeros may be dropout, not true absence) mean QC thresholds cannot distinguish biological zeros from technical dropouts
- Feature selection of highly variable genes (an alternative QC-style step) is discouraged because it can compromise local structure in low-variance genes
- scLENS on large data with all genes still requires a high-specification computer; RMT filtering computes all eigenvalues of a large cell-to-cell similarity matrix

## Evidence

- [readme] README QC defaults: "During this process, cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality."
- [readme] preprocess usage in README example: "# Perform Quality Control (QC)
pre_df = scLENS.preprocess(ndf)"
- [discussion] median scaling memory-reduction rationale: "we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling"
- [discussion] log normalization scaling factor: "we set a scaling factor of 1 during the first step of log normalization"
- [discussion] RMT memory demand: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
