---
name: covariance-spectrum-eigendecomposition-analysis
description: Use when a sparse cell-by-gene expression matrix has been eigendecomposed
  and you must decide which eigenvalue/eigenvector pairs are noise, sparsity artifacts,
  or true biological signal — including discarding delocalized eigenvectors whose
  eigenvalues fall outside the MP bulk.
when_to_use_negative:
- The expression matrix has already been imputed or zero-inflation-corrected (e.g.,
  MAGIC, scImpute, ZIFA, ZIMB-WaVE), since the sparsity-driven localization signal
  this skill detects depends on the raw sparse structure.
- The analysis goal is distribution-based noise modeling with explicit i.i.d. expression
  assumptions, which the paper explicitly avoids because the correct distribution
  for gene expression is unclear.
- The input is a small matrix where asymptotic RMT distributions (Marchenko-Pastur,
  Tracy-Widom) do not provide a reliable null.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: Python 3 library that performs the RMT-based eigendecomposition analysis of
    the single-cell covariance spectrum, including MP-bulk comparison and eigenvector
    localization/delocalization filtering
  repo: github.com/RabadanLab/randomly
- name: Python 3
  role: Required runtime — Randomly only works with Python 3, not Python 2
- name: Jupyter Notebook
  role: Recommended environment for running the Randomly eigendecomposition workflow
    interactively
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:f381ef3a578affee58fbe09a17e9d8e2098f989c32590532051cfa99d352321f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  tools:
  - Jupyter Notebook
  - Python 3
  - Randomly
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# covariance-spectrum-eigendecomposition-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Eigendecompose the covariance matrix of a sparse single-cell expression dataset and classify each eigenvalue/eigenvector pair against Random Matrix Theory universality (Marchenko-Pastur bulk) plus eigenvector delocalization tests, so that MP-bulk noise and Gaussian-profile (non-informative) eigenvectors are separated from localized signal-carrying directions.

## When to use

Use when a sparse cell-by-gene expression matrix has been eigendecomposed and you must decide which eigenvalue/eigenvector pairs are noise, sparsity artifacts, or true biological signal — including discarding delocalized eigenvectors whose eigenvalues fall outside the MP bulk.

## When NOT to use

- The expression matrix has already been imputed or zero-inflation-corrected (e.g., MAGIC, scImpute, ZIFA, ZIMB-WaVE), since the sparsity-driven localization signal this skill detects depends on the raw sparse structure.
- The analysis goal is distribution-based noise modeling with explicit i.i.d. expression assumptions, which the paper explicitly avoids because the correct distribution for gene expression is unclear.
- The input is a small matrix where asymptotic RMT distributions (Marchenko-Pastur, Tracy-Widom) do not provide a reliable null.

## Inputs

- Sparse cell-by-gene single-cell expression matrix (e.g., 95% sparse count matrix of 6,573 human PBMCs from Kang et al., or 3,005 mouse cortex cells from Zeisel et al.)
- Eigenvector components from the eigendecomposition of the single-cell covariance matrix
- Fitted Marchenko-Pastur bulk boundaries for the covariance spectrum

## Outputs

- eigenvector_normality_flags.csv with per-eigenvector Shapiro-Wilk statistics, p-values, MP-bulk membership, and retain/discard decisions
- Classification of the spectrum into noise (MP bulk), sparsity-induced signal (~3%), and true biological signal (~2%) components
- Set of retained localized (informative) eigenvectors for downstream low-rank representation and feature selection

## How to apply

Compute the covariance matrix of the single-cell cell-gene expression matrix (e.g., the 95%-sparse 6,573 PBMC Kang et al. or 3,005 mouse cortex Zeisel et al. datasets) and eigendecompose it. Fit the Marchenko-Pastur bulk to the spectrum; eigenvalues outside the bulk are candidate signal, consistent with the RMT model of a random matrix plus low-rank perturbation. For each candidate eigenvector, test whether its components follow a Gaussian (delocalized) profile via a Shapiro-Wilk normality test: delocalized eigenvectors are discarded as non-informative even when their eigenvalues lie outside the MP bulk, since sparsity can drive both spurious localization and delocalization. Retain localized eigenvectors, which the paper interprets as groups of correlated cells; to separate biological from sparsity-induced localization, first randomize the dataset by permuting cell labels independently per gene and note which genes drive localization in the randomized case (these are removed by the sparsity feature-selection filter). Write per-eigenvector Shapiro-Wilk statistics, p-values, MP-bulk membership, and retain/discard decisions to eigenvector_normality_flags.csv before building the low-rank signal representation.

## Related tools

- **Randomly** (Python 3 library that performs the RMT-based eigendecomposition analysis of the single-cell covariance spectrum, including MP-bulk comparison and eigenvector localization/delocalization filtering) — github.com/RabadanLab/randomly
- **Python 3** (Required runtime — Randomly only works with Python 3, not Python 2)
- **Jupyter Notebook** (Recommended environment for running the Randomly eigendecomposition workflow interactively)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- Roughly 95% of the eigenvectors/information in the covariance spectrum should be compatible with RMT predictions (noise), ~3% flagged as spurious sparsity-induced signal, and ~2% retained as true biological signal.
- eigenvector_normality_flags.csv contains a row for every eigenvector with a Shapiro-Wilk statistic, p-value, MP-bulk membership flag, and an explicit retain/discard decision.
- Every discarded eigenvector should have a Gaussian component profile (Shapiro-Wilk fails to reject normality) regardless of its eigenvalue's position relative to the MP bulk.
- Randomizing the dataset by permuting cell labels per gene should collapse the spectrum into the MP bulk, confirming that eigenvalues outside the bulk in the original data reflect correlated structure rather than sparsity.
- Genes responsible for localized eigenvectors in the randomized data should be the ones removed by the sparsity feature-selection filter.

## Limitations

- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data — localization alone is not sufficient evidence of biological signal.
- Randomly only works with Python 3 (not Python 2).
- No version or release information is provided for the publicly available algorithm code.
- The approach avoids assuming a specific expression distribution, but the paper notes it is not clear what the correct distribution is or whether the i.i.d. assumption is reasonable.

## Evidence

- [intro] localization can be interpreted as groups of cells whose gene expression is correlated.: "localization can be interpreted as groups of cells whose gene expression is correlated."
