---
name: eigenvector-correlation-robustness-scoring
description: Use when scRNA-seq count data have passed RMT-based noise filtering but
  a data-driven decision is still needed on which signal eigenvectors are robust versus
  dropout-driven, prior to dimensionality reduction/embedding.
when_to_use_negative:
- Input is not a sparse count matrix — the perturbation sparsity calibration relies
  on the data's original sparsity (binarizing non-zero values), so dense or heavily
  transformed inputs are inappropriate
- The analysis goal is imputation or denoising of individual gene expression values
  — this test only scores signal robustness and does not replace zeros with imputed
  values
- A manual/user-defined dimensionality threshold is already fixed — scLENS is designed
  to avoid subjective threshold selection
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scLENS
  role: 'Implements the full robustness workflow: preprocessing, RMT-based noise filtering,
    sparse random perturbation generation, eigenvector correlation scoring, and UMAP
    embedding of robust signals'
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python distribution of scLENS providing the conda environment and PyTorch
    GPU/CPU execution for running the robustness test
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:887f2754f8dacc0e1a5acb6a248d2e4577bb7930139dc089f75519a03c0d7c7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# eigenvector-correlation-robustness-scoring

## Summary

A signal robustness test from scLENS that removes low-quality signals arising from stochastic dropout by scoring the stability of RMT-derived signal eigenvectors against sparse random perturbations of the count matrix, without imputation or altering the original data.

## When to use

Use when scRNA-seq count data have passed RMT-based noise filtering but a data-driven decision is still needed on which signal eigenvectors are robust versus dropout-driven, prior to dimensionality reduction/embedding.

## When NOT to use

- Input is not a sparse count matrix — the perturbation sparsity calibration relies on the data's original sparsity (binarizing non-zero values), so dense or heavily transformed inputs are inappropriate
- The analysis goal is imputation or denoising of individual gene expression values — this test only scores signal robustness and does not replace zeros with imputed values
- A manual/user-defined dimensionality threshold is already fixed — scLENS is designed to avoid subjective threshold selection

## Inputs

- Raw scRNA-seq count matrix (rows = cells, columns = genes; CSV with gene names in the first row and cell IDs in the first column, or JLD2 with a 'data' DataFrame containing a :cell column)
- RMT-derived signal eigenvector set V_sig from scLENS noise filtering

## Outputs

- Per-signal robustness scores and selected robust signal vectors written to robustness_scores.csv
- Set of robust signals (robustness value > 0.5) used for reconstruction of the reduced data

## How to apply

First preprocess the raw count matrix with scLENS QC, log normalization (scaling factor 1), gene scaling, and L2 normalization, compute the cell similarity matrix G = XX^T/N, and obtain the signal eigenvector set V_sig via RMT-based noise filtering (Marchenko-Pastur fit with Tracy-Widom threshold at 0.05 probability). Determine the perturbation sparsity iteratively: start with a binary random perturbation matrix at sparsity 0.999, perturb, preprocess, and eigendecompose, decreasing sparsity until the minimum correlation between the two eigenvector sets drops below the average correlation between eigenvector sets from two random matrices at the data's original sparsity (typically selected above 0.97). Generate 10 perturbed datasets by adding the binary random matrix to the original count matrix, then for each signal vector compute the column-wise maximum absolute inner product against each of the 10 perturbed eigenvector sets (Eq. 3) and average the 10 maxima into a robustness value (Eq. 4). Retain signals with robustness value greater than 0.5; this filters dropout-driven signals without imputation because the perturbation merely probes sensitivity of the original eigenvectors.

## Related tools

- **scLENS** (Implements the full robustness workflow: preprocessing, RMT-based noise filtering, sparse random perturbation generation, eigenvector correlation scoring, and UMAP embedding of robust signals) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python distribution of scLENS providing the conda environment and PyTorch GPU/CPU execution for running the robustness test) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS; sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.plot_stability(sclens_embedding)
```

## Evaluation signals

- Every retained signal has a mean robustness value (average of 10 column-wise maximum absolute inner products) greater than 0.5
- The selected perturbation-matrix sparsity is 0.97 or greater and satisfies the calibration criterion: minimum inter-set eigenvector correlation below the average random-matrix correlation at the data's original sparsity
- Per-signal robustness scores and selected signal vectors are present in robustness_scores.csv
- On the paper's example data, 6 RMT-detected signals reduce to 3 robust signals, and the 2D UMAP embedding (15 nearest neighbors, min_dist 0.01, cosine metric) shows three accurately, non-overlapping separated clusters
- scLENS.plot_stability can be called on the embedding to visualize the signal stability used for threshold determination

## Limitations

- scLENS's RMT-based approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources
- Operation on large data with whole genes still requires a computer with high specifications
- 10 repeated perturbations are needed per dataset, multiplying preprocessing and eigendecomposition cost

## Evidence

- [abstract] Robustness test retains signals above 0.5: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [discussion] Robustness test removes dropout signals without imputation: "scLENS removes low-quality signals arising from stochastic dropout events using a signal robustness test which does not require imputation"
- [discussion] RMT filtering provides the data-driven threshold the robustness test builds on: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [readme] Plotting stability of signals is supported in the Julia package: "scLENS.plot_stability(sclens_embedding)"
