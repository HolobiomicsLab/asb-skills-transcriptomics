---
name: tracy-widom-threshold-estimation
description: Use when analyzing sparse scRNA-seq count matrices (e.g., Z8eq.csv.gz)
  and a data-driven signal-vs-noise cutoff for dimensionality reduction is needed
  instead of manual or heuristic PC selection.
when_to_use_negative:
- When the goal is analysis on a low-specification machine with a large dataset and
  whole genes — computing all eigenvalues of the large cell-to-cell similarity matrix
  requires significant memory resources and a computer with high specifications
- When input is already a reduced embedding or a feature-selected (highly variable
  gene) matrix, since the method expects the whole-gene normalized count matrix and
  variance-based feature selection can compromise low-variance local structure
- When a subjective, user-defined dimension threshold is acceptable and RMT fitting
  overhead is unnecessary
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: 'Julia package implementing the RMT noise-filtering workflow: preprocessing
    (L2 normalization), similarity-matrix eigenvalue decomposition, MP distribution
    fitting, and Tracy-Widom thresholding'
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python/PyTorch distribution of scLENS with conda environment setup for running
    the RMT-based signal detection
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:efd265b029650e7e500d9df91f921f9c12f041923fd64271e3b030eb5d7623d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - scLENS
  - scLENSpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tracy-widom-threshold-estimation

## Summary

Random matrix theory (RMT)-based noise filtering, as implemented in scLENS, fits a Marchenko-Pastur (MP) distribution to the eigenvalue spectrum of the cell-to-cell similarity matrix G = XX^T/N and uses a Tracy-Widom (TW) threshold to objectively separate biological signal eigenvalues from noise eigenvalues without user-tuned parameters.

## When to use

Use when analyzing sparse scRNA-seq count matrices (e.g., Z8eq.csv.gz) and a data-driven signal-vs-noise cutoff for dimensionality reduction is needed instead of manual or heuristic PC selection.

## When NOT to use

- When the goal is analysis on a low-specification machine with a large dataset and whole genes — computing all eigenvalues of the large cell-to-cell similarity matrix requires significant memory resources and a computer with high specifications
- When input is already a reduced embedding or a feature-selected (highly variable gene) matrix, since the method expects the whole-gene normalized count matrix and variance-based feature selection can compromise low-variance local structure
- When a subjective, user-defined dimension threshold is acceptable and RMT fitting overhead is unnecessary

## Inputs

- Raw scRNA-seq count matrix as CSV (rows = cells, columns = genes, e.g., data/Z8eq.csv.gz) or JLD2 DataFrame with a :cell column
- Preprocessed gene-scaled, L2-normalized cell-by-gene matrix X after QC filtering (cells with < 200 expressed genes and genes expressed in < 15 cells removed)

## Outputs

- Set of signal eigenvalues exceeding the Tracy-Widom 0.05 threshold and their signal eigenvectors V_sig
- n-dimensional PC-score-equivalent embedding (sclens_embedding[:pca_n1], savable as pca.csv)
- Diagnostic plots: MP distribution fit (plot_mpdist) and clustering stability (plot_stability)
- AnnData (.h5ad), CSV, or UMAP embedding exports of the denoised result

## How to apply

First preprocess the raw count matrix per the scLENS pipeline: log normalization with scaling factor L=1, gene (median) scaling, and L2 normalization of cell vectors to unit length to prevent signal distortion. Compute the cell-to-cell similarity matrix G = XX^T/N (N = number of genes) and perform eigenvalue decomposition to obtain all eigenvalues and eigenvectors. Fit the Marchenko-Pastur distribution to the eigenvalue spectrum following the Aparicio et al. fitting procedure; eigenvalues conforming to the MP fit are noise. Derive the Tracy-Widom threshold from the fitted MP parameters as the critical point beyond which an eigenvalue has only a 0.05 probability under the TW distribution of a random matrix; eigenvalues exceeding it, together with their eigenvectors (V_sig), are retained as potential biological signals. Because the eigenvector basis is equivalent to PCA, sqrt(eigenvalue) * eigenvector products reproduce the n-dimensional PC score matrix exactly, so results can be verified against PCA; scLENS additionally applies a signal robustness test to remove low-quality signals from stochastic dropout without imputation.

## Related tools

- **scLENS** (Julia package implementing the RMT noise-filtering workflow: preprocessing (L2 normalization), similarity-matrix eigenvalue decomposition, MP distribution fitting, and Tracy-Widom thresholding) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python/PyTorch distribution of scLENS with conda environment setup for running the RMT-based signal detection) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS; pre_df = scLENS.preprocess(scLENS.read_file("data/Z8eq.csv.gz")); emb = scLENS.sclens(pre_df, device_="cpu"); scLENS.plot_mpdist(emb)
```

## Evaluation signals

- Fitted MP distribution visually matches the bulk of the eigenvalue spectrum (plot_mpdist shows noise eigenvalues conforming to Marchenko-Pastur)
- Eigenvalues flagged as signals all exceed the Tracy-Widom critical value at the 0.05 probability level derived from the fitted MP parameters
- Reconstruction check: sqrt(signal eigenvalue) * signal eigenvector products equal the n-PC score matrix from PCA of the normalized data (near-zero deviation)
- plot_stability shows the retained signal dimensions are stable under the signal robustness test
- QC/preprocessing invariants hold: cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells were filtered, and cell vectors are unit L2 norm

## Limitations

- The RMT-based approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources; operation on large data with whole genes still requires a computer with high specifications
- Denoising methods based on RMT alone did not show substantial performance improvements over existing approaches in earlier work (Aparicio et al.), motivating scLENS's additional signal robustness test
- CUDA support is optional; without an NVIDIA GPU the package falls back to CPU, which may be slow for large matrices

## Evidence

- [discussion] RMT-based noise filtering to provide a data-driven signal/noise threshold: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [abstract] RMT-based noise filtering and signal robustness test enable data-driven threshold: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [intro] Prior RMT denoising (Aparicio et al.) automatically distinguishes signal from noise: [evidence span withheld — anchor status: not_found]
- [readme] scLENS README: RMT noise filtering enables automatic signal detection: "By incorporating L2 normalization and leveraging random matrix theory (RMT) for noise filtering, scLENS automatically detects biologically meaningful signals"
