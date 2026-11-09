---
name: pca-signal-equivalence-verification
description: Use when reconstructing scLENS's RMT noise-filtering module and you must
  prove that eigen-decomposition of the cell similarity matrix G = XX^T/N yields a
  signal subspace equivalent to PCA scores.
when_to_use_negative:
- The input has not been L2-normalized after log normalization — signal distortion
  from unequal cell vector lengths invalidates the PCA equivalence comparison
- You need only standard PCA output and have no requirement to validate the RMT-selected
  signal subspace; plain PCA suffices and avoids the full eigenvalue decomposition
  memory cost
tools:
- name: scLENS
  role: Julia package providing preprocess (QC filtering cells <200 genes, genes <15
    cells), sclens embedding, and MP-distribution plotting used to generate and inspect
    the signal eigenvalues being verified against PCA
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python environment (Anaconda + PyTorch) for running scLENS when GPU/CUDA execution
    of the eigen-decomposition is required
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:8c7ee0b901d3a726ced1a712c163243d0abe8291d46b3b71aaed6e2261906b26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - scLENS
  - scLENSpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pca-signal-equivalence-verification

## Summary

Verification step for scLENS's RMT-based noise filtering that mathematically checks the product of the square roots of the n signal eigenvalues and their eigenvectors equals the n-dimensional PC score matrix from PCA on the L2-normalized scRNA-seq data, ensuring the RMT signal subspace is PCA-consistent.

## When to use

Use when reconstructing scLENS's RMT noise-filtering module and you must prove that eigen-decomposition of the cell similarity matrix G = XX^T/N yields a signal subspace equivalent to PCA scores.

## When NOT to use

- The input has not been L2-normalized after log normalization — signal distortion from unequal cell vector lengths invalidates the PCA equivalence comparison
- You need only standard PCA output and have no requirement to validate the RMT-selected signal subspace; plain PCA suffices and avoids the full eigenvalue decomposition memory cost

## Inputs

- Raw scRNA-seq count matrix (e.g. package example data Z8eq.csv.gz, rows = cells, columns = genes)
- Preprocessed normalized data matrix X (log normalization with L=1, gene scaling, L2-normalized cell vectors)
- Cell-to-cell similarity matrix G = XX^T/N with full eigenvalue decomposition
- Marchenko-Pastur distribution fit parameters and Tracy-Widom 0.05 threshold

## Outputs

- signal_eigenvalues.csv listing all computed eigenvalues with those above the TW 0.05-threshold flagged as signals
- equivalence_report.csv recording the maximum deviation between the n-signal reconstruction and the n-PC score matrix
- Signal eigenvectors V_sig defining the data-driven low-dimensional embedding

## How to apply

After preprocessing the scRNA-seq matrix (log normalization with scaling factor L=1, gene scaling, L2 normalization of cell vectors to unit length), compute the cell-to-cell similarity matrix G = XX^T/N where N is the number of genes, and perform eigenvalue decomposition. Fit the Marchenko-Pastur (MP) distribution to the eigenvalue spectrum (Aparicio et al. procedure) and derive the Tracy-Widom threshold at the critical point where an eigenvalue has 0.05 probability under the TW distribution of a random matrix. Select signal eigenvalues exceeding the TW threshold with their eigenvectors (signal vectors V_sig). Then reconstruct the n-dimensional scores as sqrt(lambda_i) * V_sig for the n signal eigenvalues and compare to the score matrix obtained by multiplying n PCs to the normalized data; record the maximum deviation in an equivalence report. Near-zero deviation validates that the RMT-selected signal subspace is mathematically identical to the leading PCA scores.

## Related tools

- **scLENS** (Julia package providing preprocess (QC filtering cells <200 genes, genes <15 cells), sclens embedding, and MP-distribution plotting used to generate and inspect the signal eigenvalues being verified against PCA) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python environment (Anaconda + PyTorch) for running scLENS when GPU/CUDA execution of the eigen-decomposition is required) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS; pre_df = scLENS.preprocess(scLENS.read_file("data/Z8eq.csv.gz")); sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.plot_mpdist(sclens_embedding)
```

## Evaluation signals

- signal_eigenvalues.csv exists and contains the complete eigenvalue set with every eigenvalue exceeding the Tracy-Widom 0.05-threshold flagged as signal
- equivalence_report.csv shows maximum deviation between the sqrt(lambda_i)*V_sig reconstruction and the n-PC score matrix that is near zero
- Number of signal dimensions n equals the count of eigenvalues exceeding the TW threshold and matches the number of PCs used in the comparison score matrix
- Eigenvalues classified as noise conform to the fitted Marchenko-Pastur distribution, while flagged signals deviate from it
- plot_mpdist output of the sclens embedding visually confirms the MP fit used to set the TW threshold

## Limitations

- The RMT-based approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources
- scLENS operation on large data with whole genes still requires a computer with high specifications
- GPU (CUDA) support is optional; without CUDA the package automatically falls back to CPU, which slows full eigenvalue decomposition

## Evidence

- [other] confirm that the product of the square root of the n signal eigenvalues with their eigenvectors is mathematically equivalent to the n-dimensional PC score matrix: [evidence span withheld — anchor status: not_found]
- [discussion] RMT-based noise filtering provides a data-driven threshold: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [readme] By incorporating L2 normalization and leveraging random matrix theory (RMT) for noise filtering, scLENS automatically detects biologically meaningful signals: "By incorporating L2 normalization and leveraging random matrix theory (RMT) for noise filtering, scLENS automatically detects biologically meaningful signals"
- [readme] Plot the distribution of eigenvalues and clustering stability: "Plot the distribution of eigenvalues and clustering stability"
