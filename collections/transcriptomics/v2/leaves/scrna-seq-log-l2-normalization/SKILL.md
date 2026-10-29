---
name: scrna-seq-log-l2-normalization
description: Use when preprocessing raw scRNA-seq count matrices (CSV or JLD2 with
  rows=cells, columns=genes) before dimensionality reduction, especially for sparse
  data where conventional log normalization would distort signals via unequal cell
  vector lengths.
when_to_use_negative:
- The data are already normalized or already a feature/gene-selected table — re-normalizing
  would not address the log-normalization distortion scLENS targets
- The goal is to repair dropout zeros before analysis — scLENS deliberately avoids
  imputation, which can bias data and generate false signals
- The data are not single-cell count matrices (e.g., bulk RNA-seq or spatial statistics)
  where the cell-vector-length distortion mechanism does not apply
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3577
tools:
- name: scLENS
  role: Julia package whose preprocess function performs QC (cells <200 genes and
    genes <15 cells filtered) and the log/L2 normalization pipeline
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python environment distribution of scLENS providing the conda environment
    (environment.yml) and PyTorch GPU/CPU backend
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:791f8dbcb81668945904b5764a073763ced71106b082ca81b3063ed3dac844d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrna-seq-log-l2-normalization

## Summary

scLENS's two-step preprocessing for scRNA-seq count matrices: log normalization with a scaling factor of 1, followed by L2 normalization that uniformizes cell vector lengths. This combination addresses the false variability that conventional log normalization introduces by amplifying gaps between zero and non-zero values.

## When to use

Use when preprocessing raw scRNA-seq count matrices (CSV or JLD2 with rows=cells, columns=genes) before dimensionality reduction, especially for sparse data where conventional log normalization would distort signals via unequal cell vector lengths.

## When NOT to use

- The data are already normalized or already a feature/gene-selected table — re-normalizing would not address the log-normalization distortion scLENS targets
- The goal is to repair dropout zeros before analysis — scLENS deliberately avoids imputation, which can bias data and generate false signals
- The data are not single-cell count matrices (e.g., bulk RNA-seq or spatial statistics) where the cell-vector-length distortion mechanism does not apply

## Inputs

- Raw scRNA-seq count matrix as CSV (.csv/.csv.gz, rows=cells, columns=genes, first row gene names, first column cell IDs)
- 10x format data (matrix.mtx.gz, features.tsv.gz, barcodes.tsv.gz) convertible via tenx2jld2
- JLD2 file containing a DataFrame under key 'data' with a :cell column of cell IDs

## Outputs

- Preprocessed DataFrame (QC-filtered, log-normalized with scaling factor 1, L2-normalized) ready for scLENS.sclens()
- Converted JLD2 data file (out_jld2/out.jld2) when starting from 10x gz input

## How to apply

Load the raw count matrix (e.g., via scLENS.read_file for CSV, or tenx2jld2 to convert 10x gz files to JLD2), then run scLENS QC filtering cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells (scLENS.preprocess). Apply log normalization with the scaling factor set to 1, which avoids inflating absolute expression values. Then apply L2 normalization to uniformize cell vector lengths — the paper identifies the primary cause of signal distortion during log normalization as non-uniform cell vector lengths, which L2 normalization effectively corrects. The resulting preprocessed matrix feeds downstream gene scaling (median scaling instead of z-score) and L2 normalization prior to eigendecomposition of the cell similarity matrix G = XX^T/N. No imputation of zeros is performed; zeros are left intact so stochastic dropout-driven low-quality signals can be filtered later by the signal robustness test.

## Related tools

- **scLENS** (Julia package whose preprocess function performs QC (cells <200 genes and genes <15 cells filtered) and the log/L2 normalization pipeline) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python environment distribution of scLENS providing the conda environment (environment.yml) and PyTorch GPU/CPU backend) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS; ndf = scLENS.read_file("data/Z8eq.csv.gz"); pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_="cpu")
```

## Evaluation signals

- QC filter removes cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells before normalization
- All cell vectors in the preprocessed matrix have uniform (L2 norm = 1) lengths, i.e., no cell dominates the similarity matrix G = XX^T/N
- Log normalization used scaling factor 1 (not the typical 10,000), so downstream PCA/signal vectors are not inflated by large-scale factors
- Preprocessed matrix retains original zeros (no imputed values), which is required for the later robustness test to catch dropout-driven low-quality signals
- Running scLENS.sclens() on the preprocessed output should yield RMT-based noise filtering with a Marchenko-Pastur distribution fit for subsequent signal detection

## Limitations

- RMT-based downstream steps require computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory; scLENS on large data with whole genes still requires a computer with high specifications
- Conventional z-score gene scaling is memory-heavy and is replaced by median scaling in scLENS
- L2 normalization addresses distortion introduced during log normalization but does not repair false signals already generated by imputation approaches

## Evidence

- [abstract] Identifies primary cause of signal distortion during log normalization and corrects it via uniformizing cell vector lengths with L2 normalization: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [discussion] Scaling factor of 1 is used during log normalization: "we set a scaling factor of 1 during the first step of log normalization"
- [intro] Conventional log normalization amplifies gaps between zero and non-zero values: "conventional log normalization can introduce false variability by amplifying gaps between zero and non-zero values and not uniformly normalizing genes across different expression levels"
- [readme] README: QC filters cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
- [discussion] Median scaling replaces z-score gene scaling to reduce memory demand: "we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling"
