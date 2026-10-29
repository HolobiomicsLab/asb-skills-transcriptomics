---
name: random-matrix-theory-noise-filtering
description: Use when analyzing sparse scRNA-seq count data and a data-driven signal/noise
  cutoff on PCA-like dimensions is needed instead of subjective elbow-inspection of
  eigenvalues.
when_to_use_negative:
- Input data has already been feature-selected to highly variable genes — the article
  warns this compromises local structure in low-variance genes and would bias the
  RMT eigenvalue spectrum
- Data has been imputed — the article notes imputation can bias data and generate
  false signals that RMT filtering cannot distinguish from true biology
- Only a small random-matrix-like synthetic matrix without a meaningful MP regime
  is available — the MP/TW fit assumes a large noise eigenvalue spectrum to fit against
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scLENS
  role: 'Julia package implementing the RMT-based noise filtering: QC preprocess,
    similarity-matrix EVD, Marchenko-Pastur fit, and Tracy-Widom thresholding within
    the sclens() pipeline'
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python distribution of scLENS providing the conda environment (environment.yml)
    plus PyTorch GPU/CPU backend used to run the same RMT filtering pipeline
  repo: Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:5afc8a5ecc5af8a5f34ae1adb74509eb71f936071eda989e99c1ef806c9efd4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# random-matrix-theory-noise-filtering

## Summary

A data-driven noise-filtering step from scLENS that classifies eigenvalues of the cell-to-cell similarity matrix G = XX^T/N as noise or biological signal by fitting the Marchenko-Pastur distribution and applying a Tracy-Widom threshold, eliminating manual signal-dimension tuning.

## When to use

Use when analyzing sparse scRNA-seq count data and a data-driven signal/noise cutoff on PCA-like dimensions is needed instead of subjective elbow-inspection of eigenvalues.

## When NOT to use

- Input data has already been feature-selected to highly variable genes — the article warns this compromises local structure in low-variance genes and would bias the RMT eigenvalue spectrum
- Data has been imputed — the article notes imputation can bias data and generate false signals that RMT filtering cannot distinguish from true biology
- Only a small random-matrix-like synthetic matrix without a meaningful MP regime is available — the MP/TW fit assumes a large noise eigenvalue spectrum to fit against

## Inputs

- Raw scRNA-seq count matrix as CSV (rows = cells, columns = genes, header row of gene IDs, first column of cell IDs) or JLD2 file with a DataFrame under key "data" and a :cell column (e.g., data/Z8eq.csv.gz)
- Optional gene-ID mapping CSV with columns "gene" and "gene_ID" for renaming genes

## Outputs

- Signal eigenvalues/eigenvector set V_sig (e.g., signal_eigenvalues.csv) with eigenvalues above the Tracy-Widom 0.05 threshold flagged as signals
- MP-fit diagnostics (e.g., MP distribution plot via scLENS.plot_mpdist) and an equivalence report comparing the n-signal reconstruction to the n-PC score matrix
- Normalized/filtered reduced data feeding downstream signal robustness testing and PCA/UMAP outputs (out/pca.csv, out/umap.csv)

## How to apply

Load the raw scRNA-seq count matrix (e.g., scLENS example data Z8eq.csv.gz) and preprocess with scLENS QC (drop cells with <200 expressed genes and genes expressed in <15 cells), log normalization with scaling factor L=1, gene scaling, and L2 normalization of each cell vector to unit length to prevent log-normalization-induced artificial signals. Compute the cell-to-cell similarity matrix G = XX^T/N (N = number of genes), perform eigenvalue decomposition, and fit the Marchenko-Pastur distribution to the eigenvalue spectrum following the Aparicio et al. fitting procedure. Eigenvalues conforming to the MP distribution are classified as noise; eigenvalues deviating from the MP fit and exceeding the Tracy-Widom critical point (0.05 probability under the TW distribution of a random matrix) are retained as signal eigenvectors V_sig. Verify the reduced reconstruction is equivalent to PCA by checking that the n signal eigenvalue/eigenvector products match the n-PC score matrix with near-zero deviation; scLENS's plot_mpdist visualizes the MP fit for inspection.

## Related tools

- **scLENS** (Julia package implementing the RMT-based noise filtering: QC preprocess, similarity-matrix EVD, Marchenko-Pastur fit, and Tracy-Widom thresholding within the sclens() pipeline) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python distribution of scLENS providing the conda environment (environment.yml) plus PyTorch GPU/CPU backend used to run the same RMT filtering pipeline) — Mathbiomed/scLENSpy

## Examples

```
ndf = scLENS.read_file("data/Z8eq.csv.gz"); pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.plot_mpdist(sclens_embedding)
```

## Evaluation signals

- signal_eigenvalues.csv exists and contains the full eigenvalue spectrum with values exceeding the Tracy-Widom 0.05-probability threshold flagged as signals
- PCA-equivalence check: maximum deviation between the product of sqrt(signal eigenvalues) with eigenvectors and the n-PC score matrix is near zero (recorded in equivalence_report.csv)
- On the Z8eq example data with log normalization plus L2 normalization, the detected signal count drops from 33 to 6 (Fig. 2 validation pattern)
- On a TGC-biased sparse synthetic matrix (first 400 rows halved), log normalization alone yields ~400 artificial signals, while adding L2 normalization yields 0 signals
- scLENS.plot_mpdist(sclens_embedding) shows eigenvalues conforming to the fitted Marchenko-Pastur distribution classified as noise

## Limitations

- The RMT-based approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources
- Even with median scaling replacing z-score gene scaling, scLENS operation on large data with whole genes still requires a computer with high specifications
- The preceding Aparicio et al. RMT denoising alone did not show substantial performance improvements over existing approaches — scLENS adds L2 normalization and a signal robustness test for gains

## Evidence

- [abstract] we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [discussion] provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [intro] Aparicio et al. introduced a denoising algorithm based on random matrix theory (RMT), which automatically distinguishes signal from noise in scRNA-seq data.: [evidence span withheld — anchor status: not_found]
- [readme] During this process, cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality.: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
- [abstract] we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
