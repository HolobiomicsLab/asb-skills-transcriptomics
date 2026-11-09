---
name: eigenvector-component-normality-testing
description: Use when eigenvectors of a single-cell covariance matrix must be classified
  as informative vs non-informative — e.g. eigenvalues outside the Marchenko-Pastur
  bulk whose components still look Gaussian and may be sparsity-driven artifacts.
when_to_use_negative:
- The input is already a filtered, low-rank representation rather than raw single-cell
  covariance eigenvectors — there is nothing left to discard
- The goal is feature (gene) selection for sparsity effects; the paper's sparsity
  filter discards genes causing localized eigenvectors in the randomized data, a different
  step from eigenvector normality testing
- Eigenvalue-only analysis is sufficient — this skill specifically requires eigenvector
  component distributions, not just the spectrum
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Randomly
  role: Library providing the RMT eigendecomposition and filtering pipeline in which
    the Shapiro-Wilk eigenvector normality filter is applied
  repo: github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime required to run Randomly and implement the per-eigenvector Shapiro-Wilk
    tests and CSV flag output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:c1c8d108405802eb1f38a30466fb1774114b8f2e822c2b27fa15859bc8e1e03d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  tools:
  - Python 3
  - Randomly
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# eigenvector-component-normality-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A filter from the Randomly RMT denoising pipeline that runs a Shapiro-Wilk normality test on the component distribution of each eigenvector of the single-cell covariance matrix, discarding delocalized (Gaussian-profile) eigenvectors as non-informative even when their eigenvalues fall outside the Marchenko-Pastur bulk, since only localized eigenvectors reflect correlated groups of cells.

## When to use

Use when eigenvectors of a single-cell covariance matrix must be classified as informative vs non-informative — e.g. eigenvalues outside the Marchenko-Pastur bulk whose components still look Gaussian and may be sparsity-driven artifacts.

## When NOT to use

- The input is already a filtered, low-rank representation rather than raw single-cell covariance eigenvectors — there is nothing left to discard
- The goal is feature (gene) selection for sparsity effects; the paper's sparsity filter discards genes causing localized eigenvectors in the randomized data, a different step from eigenvector normality testing
- Eigenvalue-only analysis is sufficient — this skill specifically requires eigenvector component distributions, not just the spectrum

## Inputs

- Eigenvectors (component vectors) from eigendecomposition of the single-cell covariance matrix
- Corresponding eigenvalues of the covariance spectrum
- Marchenko-Pastur bulk boundaries for the covariance spectrum

## Outputs

- Per-eigenvector Shapiro-Wilk statistics and p-values
- Per-eigenvector Marchenko-Pastur bulk membership flags
- Per-eigenvector retain/discard decisions (eigenvector_normality_flags.csv)
- Signal candidate set of eigenvectors with delocalized eigenvectors removed

## How to apply

Load each eigenvector from the eigendecomposition of the single-cell covariance matrix and run a Shapiro-Wilk test on its component distribution to decide whether the components are Gaussian (delocalized) or localized. Under the RMT interpretation, localized eigenvectors correspond to groups of cells with correlated gene expression, whereas Gaussian, delocalized components indicate noise- or sparsity-driven directions. Critically, the normality verdict overrides the eigenvalue position: an eigenvalue lying outside the Marchenko-Pastur bulk is still discarded if the Shapiro-Wilk test indicates a Gaussian component profile, because sparsity can cause spurious eigenvector localization. Write per-eigenvector Shapiro-Wilk statistics, p-values, Marchenko-Pastur bulk membership, and retain/discard decisions to an eigenvector_normality_flags.csv file, and remove all flagged delocalized eigenvectors from the signal candidate set.

## Related tools

- **Randomly** (Library providing the RMT eigendecomposition and filtering pipeline in which the Shapiro-Wilk eigenvector normality filter is applied) — github.com/luisaparicio/randomly
- **Python 3** (Runtime required to run Randomly and implement the per-eigenvector Shapiro-Wilk tests and CSV flag output)

## Examples

```
from scipy.stats import shapiro; W, p = shapiro(eigenvector_components)  # discard eigenvector if Gaussian per Shapiro-Wilk, even if eigenvalue lies outside the MP bulk
```

## Evaluation signals

- eigenvector_normality_flags.csv contains, for every eigenvector, the Shapiro-Wilk statistic, p-value, MP-bulk membership, and a retain/discard decision
- Every eigenvector flagged Gaussian (delocalized) by Shapiro-Wilk is discarded, including those with eigenvalues outside the Marchenko-Pastur bulk
- No eigenvector retained in the signal candidate set has a Gaussian component profile
- The retained eigenvectors show localized (non-Gaussian) component distributions consistent with correlated groups of cells
- Randomly runs under Python 3 only; the pipeline must not be attempted under Python 2

## Limitations

- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data, so localization alone is not sufficient evidence of biological signal
- The paper lacks a complete quantitative understanding of single-cell noise sources and does not assume a specific expression distribution or i.i.d. structure, so normality-based decisions rely on RMT universality rather than an explicit noise model
- No version or release information is provided for the publicly available algorithm code

## Evidence

- [other] Delocalized eigenvectors discarded despite eigenvalue outside MP bulk: [evidence span withheld — anchor status: not_found]
- [abstract] Spurious localization from sparsity: [evidence span withheld — anchor status: not_found]
