---
name: sparse-binary-matrix-perturbation
description: Use when a scRNA-seq count matrix contains RMT-detected signal dimensions
  possibly inflated by stochastic dropout events, and a data-driven, imputation-free
  criterion is needed to decide which signal eigenvectors to keep.
when_to_use_negative:
- When the goal is denoising via imputation — scLENS deliberately avoids imputation
  because imputation can bias data and generate false signals
- When the data are not sparse scRNA-seq counts (the perturbation sparsity calibration
  assumes high dropout; zeros can exceed 90%)
- When a manual signal-noise threshold is acceptable and RMT/robustness automation
  is unnecessary
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Implements preprocessing (QC, log normalization with scaling factor 1, L2
    normalization), RMT noise filtering, and the signal robustness test with sparse
    binary perturbation
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python distribution of scLENS providing the conda environment (environment.yml)
    and PyTorch backend used to run the pipeline
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:121a2cabbab252c08e194932ec034e0a4d45f5f1c2f3e7a3b50aa1a6da044eec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Sparse binary matrix perturbation

## Summary

A signal robustness test from scLENS that perturbs the raw count matrix by adding a binary random matrix with sparsity ≥0.97, then retains signals whose eigenvectors remain highly aligned with perturbed-data eigenvectors — filtering low-quality dropout-driven signals without imputation or altering the original data.

## When to use

Use when a scRNA-seq count matrix contains RMT-detected signal dimensions possibly inflated by stochastic dropout events, and a data-driven, imputation-free criterion is needed to decide which signal eigenvectors to keep.

## When NOT to use

- When the goal is denoising via imputation — scLENS deliberately avoids imputation because imputation can bias data and generate false signals
- When the data are not sparse scRNA-seq counts (the perturbation sparsity calibration assumes high dropout; zeros can exceed 90%)
- When a manual signal-noise threshold is acceptable and RMT/robustness automation is unnecessary

## Inputs

- Raw scRNA-seq count matrix (CSV rows=cells, columns=genes; or JLD2 with a 'data' DataFrame and a :cell column)
- Signal eigenvector set V_sig from RMT-based noise filtering of the cell similarity matrix G = XX^T/N

## Outputs

- Per-signal robustness scores and selected robust signal vectors (robustness_scores.csv)
- Reduced data reconstructed from selected robust eigenvalues/vectors (out/pca.csv)
- UMAP embedding of robust signals (15 neighbors, min_dist 0.01, cosine metric; out/umap.csv)

## How to apply

First preprocess the raw count matrix with scLENS QC and log normalization (scaling factor 1), gene scaling, L2 normalization, compute G = XX^T/N, and run RMT-based noise filtering (Marchenko-Pastur fit, Tracy-Widom threshold at 0.05) to get the signal eigenvector set V_sig. Calibrate the perturbation sparsity iteratively: binarize the data (non-zero values set to ones), perturb with a binary random matrix starting at sparsity 0.999, eigendecompose, and decrease sparsity until the minimum correlation between eigenvector sets falls below the average correlation between eigenvector sets of two random matrices at the data's original sparsity (typically ≥0.97). Generate 10 perturbed datasets by adding the binary perturbation matrix to the original counts, compute per-signal column-wise maximum absolute inner products against each perturbed eigenvector set (Eq. 3), average the 10 maxima into a robustness value (Eq. 4), and retain signals with robustness >0.5.

## Related tools

- **scLENS** (Implements preprocessing (QC, log normalization with scaling factor 1, L2 normalization), RMT noise filtering, and the signal robustness test with sparse binary perturbation) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python distribution of scLENS providing the conda environment (environment.yml) and PyTorch backend used to run the pipeline) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.plot_stability(sclens_embedding)
```

## Evaluation signals

- Selected perturbation sparsity must be above 0.97, having been decreased iteratively from 0.999 until the minimum inter-set eigenvector correlation drops below the random-matrix baseline correlation at the data's original sparsity
- Signals with mean robustness value (average of 10 column-wise maximum absolute inner products) greater than 0.5 are retained
- On the paper's example data, 6 RMT-detected signals must reduce to 3 robust signals
- The resulting 2D UMAP embedding must show accurate, non-overlapping distinction of the three clusters
- robustness_scores.csv must contain per-signal robustness scores and the selected signal vectors

## Limitations

- RMT-based filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources; operation on large data with whole genes still requires a computer with high specifications
- Feature selection of highly variable genes (an alternative strategy) can compromise local structure in low-variance genes; scLENS avoids this but inherits its own compute cost
- GPU/CUDA is optional but if unavailable the package falls back to CPU, which may be slow
