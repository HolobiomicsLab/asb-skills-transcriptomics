---
name: cell-similarity-matrix-eigendecomposition
description: Use when scRNA-seq count data has been log-normalized (scaling factor
  L=1), gene-scaled, and L2-normalized, and you need a data-driven cutoff distinguishing
  signal eigenvalues from random-noise eigenvalues without manual parameter tuning.
when_to_use_negative:
- 'The data is very large: scLENS''s RMT approach requires computing all eigenvalues
  of the large cell-to-cell similarity matrix, so whole-gene analysis on large data
  still requires a computer with high specifications'
- You intend to apply imputation or highly-variable-gene feature selection first —
  imputation risks biasing data and generating false signals, and variance-based gene
  selection can compromise local structure in low-variance genes
- A subjective, user-set signal-noise threshold is acceptable or desired; this skill
  exists specifically to replace manual dimension selection with a data-driven RMT
  criterion
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Julia implementation that preprocesses data, builds the cell similarity matrix,
    runs the RMT-based noise filtering with MP fit and Tracy-Widom threshold, and
    plots the MP eigenvalue distribution
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python distribution of scLENS providing the conda environment (environment.yml,
    sclens-py) with optional CUDA/PyTorch GPU acceleration for the eigendecomposition
    pipeline
  repo: Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:7501d4535f09d6253b5562bd664f8110b7a7c95eff6ced1e1c5c37fb55eb582d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scLENS
  - scLENSpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Cell-similarity matrix eigendecomposition

## Summary

Eigendecompose the cell-to-cell similarity matrix G = XX^T/N built from L2-normalized scRNA-seq data, then separate signal from noise by fitting the Marchenko-Pastur (MP) distribution and applying a Tracy-Widom (TW) threshold, as done in scLENS's RMT-based noise filtering.

## When to use

Use when scRNA-seq count data has been log-normalized (scaling factor L=1), gene-scaled, and L2-normalized, and you need a data-driven cutoff distinguishing signal eigenvalues from random-noise eigenvalues without manual parameter tuning.

## When NOT to use

- The data is very large: scLENS's RMT approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, so whole-gene analysis on large data still requires a computer with high specifications
- You intend to apply imputation or highly-variable-gene feature selection first — imputation risks biasing data and generating false signals, and variance-based gene selection can compromise local structure in low-variance genes
- A subjective, user-set signal-noise threshold is acceptable or desired; this skill exists specifically to replace manual dimension selection with a data-driven RMT criterion

## Inputs

- Raw scRNA-seq count matrix as CSV with cells as rows, genes as columns (e.g. scLENS example data Z8eq.csv.gz), or 10x gz files converted via tenx2jld2 to JLD2
- Preprocessed matrix X: log-normalized (L=1), median gene-scaled, L2-normalized cell vectors of unit length

## Outputs

- Complete set of eigenvalues and eigenvectors of the similarity matrix G = XX^T/N
- Signal eigenvalues flagged above the Tracy-Widom 0.05-threshold (e.g. signal_eigenvalues.csv)
- Signal vectors V_sig and the equivalent n-dimensional PC score matrix / scLENS embedding (pca.csv, out/test_data.h5ad)
- MP-distribution fit plot (plot_mpdist) and equivalence report comparing signal reconstruction to PCA scores

## How to apply

First preprocess the raw count matrix (e.g. example data Z8eq.csv.gz): log normalization with scaling factor 1, gene scaling (scLENS uses median scaling in place of z-score), and L2 normalization of cell vectors to unit length to prevent log-normalization signal distortion. Compute the cell-to-cell similarity matrix G = XX^T/N, where N is the number of genes, then perform eigenvalue decomposition (EVD) of G to obtain all eigenvalues and eigenvectors. Fit the Marchenko-Pastur distribution to the eigenvalue spectrum following the Aparicio et al. fitting procedure; eigenvalues conforming to the MP distribution are classified as noise. Derive the Tracy-Widom threshold as the critical point beyond which an eigenvalue has only a 0.05 probability of being observed under the TW distribution of a random matrix; eigenvalues exceeding this threshold (with their eigenvectors V_sig) are the potential biological signals. Verify that sqrt(eigenvalue)-weighted signal eigenvectors reconstruct the n-dimensional PC score matrix (n PCs times normalized data) with near-zero deviation, confirming equivalence to PCA.

## Related tools

- **scLENS** (Julia implementation that preprocesses data, builds the cell similarity matrix, runs the RMT-based noise filtering with MP fit and Tracy-Widom threshold, and plots the MP eigenvalue distribution) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python distribution of scLENS providing the conda environment (environment.yml, sclens-py) with optional CUDA/PyTorch GPU acceleration for the eigendecomposition pipeline) — Mathbiomed/scLENSpy

## Examples

```
using scLENS; sclens_embedding = scLENS.sclens(pre_df, device_="gpu"); scLENS.plot_mpdist(sclens_embedding)
```

## Evaluation signals

- signal_eigenvalues.csv exists and contains all computed eigenvalues with those above the Tracy-Widom 0.05-threshold flagged as signals
- equivalence_report.csv shows near-zero maximum deviation between the n-signal reconstruction (sqrt-eigenvalue-weighted eigenvectors) and the n-PC score matrix
- Eigenvalues conforming to the fitted Marchenko-Pastur distribution fall below threshold and are classified as noise; MP fit is visually checkable via scLENS.plot_mpdist
- QC was applied upstream: cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells were filtered by scLENS.preprocess
- Signal robustness test (scLENS.plot_stability) confirms stability of the selected signal dimensions

## Limitations

- RMT-based filtering necessitates computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources; operation on large data with whole genes still requires a computer with high specifications
- Aparicio et al.'s earlier RMT denoising algorithm automatically distinguished signal from noise but did not show substantial performance improvements over existing approaches; scLENS adds the signal robustness test and L2 normalization
- CUDA is optional — if unavailable, the package falls back to CPU, which will be slower for the EVD step

## Evidence

- [discussion] provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [abstract] RMT filtering and signal robustness test for data-driven threshold: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [discussion] Eigenvalue cost of the RMT approach: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
- [readme] README: RMT noise filtering automatic detection: "By incorporating L2 normalization and leveraging random matrix theory (RMT) for noise filtering, scLENS automatically detects biologically meaningful signals without requiring time-consuming"
