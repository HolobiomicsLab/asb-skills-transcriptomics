---
name: rmt-marchenko-pastur-bulk-screening
description: Use when eigenvectors from an eigendecomposition of a single-cell covariance
  matrix must be classified as signal candidates or noise, especially when eigenvalues
  lie outside the Marchenko-Pastur bulk yet may still be sparsity-driven, delocalized
  noise.
when_to_use_negative:
- The input is a raw gene-expression count matrix or per-gene feature table rather
  than an eigendecomposition of the covariance matrix
- The goal is imputation of zero-inflated values (e.g., MAGIC, scImpute, ZIFA) rather
  than RMT-based classification of eigenvectors
- The analysis requires i.i.d. expression-distribution assumptions, which the paper
  explicitly avoids
tools:
- name: Randomly
  role: Python 3 library that performs the RMT eigendecomposition and whose eigenvector
    components are screened with the Shapiro-Wilk / MP-bulk decision rules
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime for Randomly (Python 3 only, not Python 2), conveniently run in a
    Jupyter Notebook
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:5ba82fa8ffb86a4af5906726276da9c3bfcb396dae3939da3e2045ae921835e5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  repo_url: https://github.com/luisaparicio/randomly
  tools:
  - Python 3
  - Randomly
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rmt-marchenko-pastur-bulk-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Random Matrix Theory (RMT) screening skill for classifying eigenvectors of the single-cell covariance matrix as informative (localized) or non-informative (delocalized), combining Marchenko-Pastur bulk membership with Shapiro-Wilk normality testing so that Gaussian-profile eigenvectors are discarded even when their eigenvalues fall outside the MP bulk.

## When to use

Use when eigenvectors from an eigendecomposition of a single-cell covariance matrix must be classified as signal candidates or noise, especially when eigenvalues lie outside the Marchenko-Pastur bulk yet may still be sparsity-driven, delocalized noise.

## When NOT to use

- The input is a raw gene-expression count matrix or per-gene feature table rather than an eigendecomposition of the covariance matrix
- The goal is imputation of zero-inflated values (e.g., MAGIC, scImpute, ZIFA) rather than RMT-based classification of eigenvectors
- The analysis requires i.i.d. expression-distribution assumptions, which the paper explicitly avoids

## Inputs

- Eigenvector components from the eigendecomposition of the single-cell covariance matrix (e.g., a 6,573-cell human PBMC or 3,005-cell mouse cortex dataset processed by Randomly)
- Eigenvalue spectrum with Marchenko-Pastur bulk boundaries

## Outputs

- eigenvector_normality_flags.csv with per-eigenvector Shapiro-Wilk statistics, p-values, MP-bulk membership, and retain/discard decisions
- A filtered signal candidate set of localized eigenvectors for downstream low-rank representation

## How to apply

Load the eigenvector components obtained from the eigendecomposition of the single-cell covariance matrix (the spectrum Randomly analyzes). For each eigenvector, compute a Shapiro-Wilk test on its component distribution to assess whether the components are Gaussian (delocalized) versus localized. Cross-check each flag against the Marchenko-Pastur bulk position of its eigenvalue so that eigenvalues outside the MP bulk are still discarded when the Shapiro-Wilk test indicates a Gaussian component profile. Mark all such sparsity- or noise-driven delocalized eigenvectors as non-informative and remove them from the signal candidate set, consistent with the RMT interpretation that only localized eigenvectors reflect correlated groups of cells. Write the per-eigenvector Shapiro-Wilk statistics, p-values, MP-bulk membership, and retain/discard decisions to eigenvector_normality_flags.csv.

## Related tools

- **Randomly** (Python 3 library that performs the RMT eigendecomposition and whose eigenvector components are screened with the Shapiro-Wilk / MP-bulk decision rules) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime for Randomly (Python 3 only, not Python 2), conveniently run in a Jupyter Notebook)

## Examples

```
pip install randomly
```

## Evaluation signals

- eigenvector_normality_flags.csv exists and contains Shapiro-Wilk statistics, p-values, MP-bulk membership, and a retain/discard decision for every eigenvector
- Every eigenvector whose component distribution passes the Shapiro-Wilk Gaussian test is discarded, even if its eigenvalue lies outside the Marchenko-Pastur bulk
- Retained eigenvectors are localized (non-Gaussian components), consistent with the RMT interpretation of correlated groups of cells
- The fraction of the spectrum compatible with RMT predictions falls near the reported ~95%, with ~3% sparsity-induced spurious signal and ~2% true biological signal

## Limitations

- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data; this is why the randomized (label-permuted) control and Gaussian-profile check are required
- No version or release information is provided for the publicly available algorithm code
- Randomly only works with Python 3 (not Python 2)

## Evidence
