---
name: gradient-descent-distribution-fitting
description: Use when the input is a sparsity-filtered single-cell cell-gene expression
  matrix and the goal is to detect the part of the dataset that behaves like a random
  matrix by fitting Marchenko-Pastur universality to its covariance eigenvalue spectrum.
when_to_use_negative:
- Input is the raw, unfiltered sparse expression matrix — sparsity must first be removed
  by discarding genes causing spurious eigenvector localization in the randomized
  data, or the covariance spectrum will reflect sparsity-induced effects rather than
  the random-matrix component.
- The goal is to extract or interpret the biological (low-rank) signal itself — this
  skill only characterizes the random-matrix bulk; signal analysis is a separate downstream
  step.
- The data is already denoised or reduced to a low-rank representation — the MP fit
  would be meaningless on a spectrum whose noise bulk has been removed.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Randomly
  role: Python 3 library implementing the RMT denoising workflow, including Wishart-matrix
    eigenvalue analysis used for the MP distribution fit
  repo: github.com/RabadanLab/randomly
- name: Python 3
  role: Required runtime — Randomly only works with Python 3 (not Python 2)
- name: NumPy
  role: Used for the matrix operations that compute the sample covariance (Wishart)
    matrix and its full eigenvalue spectrum
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:4e95c29e8da58d42f981bf09428091622d27c3cdfa91849401c64c25aa233f11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  tools:
  - NumPy
  - Python 3
  - Randomly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gradient-descent-distribution-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits the non-parametric Marchenko-Pastur distribution to the empirical eigenvalue density of the Wishart (sample covariance) matrix of a sparsity-filtered single-cell expression matrix, using gradient descent to locate the MP edge and thereby identify the random-matrix (noise) component of the data.

## When to use

Use when the input is a sparsity-filtered single-cell cell-gene expression matrix and the goal is to detect the part of the dataset that behaves like a random matrix by fitting Marchenko-Pastur universality to its covariance eigenvalue spectrum.

## When NOT to use

- Input is the raw, unfiltered sparse expression matrix — sparsity must first be removed by discarding genes causing spurious eigenvector localization in the randomized data, or the covariance spectrum will reflect sparsity-induced effects rather than the random-matrix component.
- The goal is to extract or interpret the biological (low-rank) signal itself — this skill only characterizes the random-matrix bulk; signal analysis is a separate downstream step.
- The data is already denoised or reduced to a low-rank representation — the MP fit would be meaningless on a spectrum whose noise bulk has been removed.

## Inputs

- sparsity-filtered single-cell cell-gene expression matrix (e.g., chained input task_001/filtered_expression_matrix)

## Outputs

- fitted Marchenko-Pastur edge location
- eigenvalue spectrum of the Wishart matrix with the fitted MP distribution, written to eigenvalue_spectrum.csv

## How to apply

Load the sparsity-filtered expression matrix produced by the upstream sparsity-filter step (the output of the feature selection that discards genes responsible for localized eigenvectors in the randomized dataset). Compute the sample covariance (Wishart) matrix of the filtered matrix using NumPy matrix operations, then compute the full eigenvalue spectrum and build the empirical eigenvalue density. Fit the non-parametric Marchenko-Pastur distribution to this empirical density via gradient descent optimization to locate the fitted MP edge. The rationale is the paper's model in which any single-cell dataset is a random matrix (noise) plus a low-rank perturbation (signal); the bulk eigenvalue density should follow MP universality, and large deviations from these distributions indicate the presence of a signal that can be further analyzed.

## Related tools

- **Randomly** (Python 3 library implementing the RMT denoising workflow, including Wishart-matrix eigenvalue analysis used for the MP distribution fit) — github.com/RabadanLab/randomly
- **Python 3** (Required runtime — Randomly only works with Python 3 (not Python 2))
- **NumPy** (Used for the matrix operations that compute the sample covariance (Wishart) matrix and its full eigenvalue spectrum)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- The bulk of the empirical eigenvalue density must be compatible with the fitted MP distribution support (the paper reports roughly 95% of the information in single-cell data is compatible with random matrix theory predictions).
- Gradient descent converges to a stable fitted MP edge rather than an arbitrary spectral boundary.
- eigenvalue_spectrum.csv is produced and contains both the full eigenvalue spectrum and the fitted MP edge.
- The fitted edge is computed on the Wishart matrix of the sparsity-filtered data, not the raw data — a check that the upstream filter step was actually applied.

## Limitations

- Randomly only works with Python 3 (not Python 2); no version or release information is provided for the publicly available algorithm code.
- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data — this step is only valid after the sparsity filter removes genes introducing spurious effects.
- Only about 2% of the information reflects true biological signal, so deviations from the MP fit mark a small subset of eigenvalues that require further analysis.

## Evidence
