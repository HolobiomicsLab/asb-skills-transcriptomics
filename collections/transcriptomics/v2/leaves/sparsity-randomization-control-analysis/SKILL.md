---
name: sparsity-randomization-control-analysis
description: Use when a highly sparse (~95%) cell-gene expression matrix shows eigenvalues
  deviating from Marchenko-Pastur/Tracy-Widom predictions and you must decide which
  deviations are sparsity artifacts versus real biological signal.
when_to_use_negative:
- The expression matrix is dense or only mildly sparse, so sparsity-induced spurious
  localization is unlikely to dominate departures from RMT universality
- The goal is imputation or zero-inflated modeling (e.g. MAGIC, scImpute, ZIFA) rather
  than diagnosing which eigen-spectrum deviations are sparsity artifacts
- Only Python 2 is available — the Randomly library explicitly does not work with
  Python 2
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Randomly
  role: Python library used to compute the covariance eigen-spectrum, fit Marchenko-Pastur/Tracy-Widom
    universality predictions, and run the per-gene cell-label randomization control
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Required runtime — Randomly only works with Python 3, not Python 2
- name: Jupyter Notebook
  role: Convenient interactive environment for running the Randomly analysis workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:4569401b78d4d312aa33a4da16ad03ce4d72dad4a2d231bc3d68cd5fa0338f3b
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
  - Jupyter Notebook
  - Python 3
  - Randomly
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Sparsity-randomization control analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A randomization-based null control that permutes cell labels independently for each gene to isolate sparsity-induced (non-biological) eigenvector localization, enabling the partition of single-cell eigen-spectrum departures from RMT universality into ~95% random, ~3% sparsity-spurious, and ~2% true biological signal.

## When to use

Use when a highly sparse (~95%) cell-gene expression matrix shows eigenvalues deviating from Marchenko-Pastur/Tracy-Widom predictions and you must decide which deviations are sparsity artifacts versus real biological signal.

## When NOT to use

- The expression matrix is dense or only mildly sparse, so sparsity-induced spurious localization is unlikely to dominate departures from RMT universality
- The goal is imputation or zero-inflated modeling (e.g. MAGIC, scImpute, ZIFA) rather than diagnosing which eigen-spectrum deviations are sparsity artifacts
- Only Python 2 is available — the Randomly library explicitly does not work with Python 2

## Inputs

- Sparse cell-gene expression matrix (e.g. ~95% sparse matrix of 6,573 human PBMCs from Kang et al.)
- Covariance-matrix eigen-spectrum of the original data
- Randomized control eigen-spectrum and eigenvector localization statistics after per-gene cell-label permutation

## Outputs

- Classification of eigenvalues as RMT-compatible, sparsity-induced (non-biological), or true biological signal
- Three fractions summarizing the decomposition (target: ~95% RMT-compatible, ~3% sparsity-spurious, ~2% biological)
- Set of genes responsible for localized eigenvectors in the randomized (sparsity-only) case, for removal by the feature-selection filter

## How to apply

Compute the covariance-matrix eigen-spectrum of the sparse cell-gene matrix (e.g. the 95% sparse matrix of 6,573 human PBMCs from Kang et al.) with the Randomly library in Python 3, fitting the Marchenko-Pastur bulk and Tracy-Widom edge statistics to classify eigenvalues as compatible or deviating. Build the control by permuting cell labels independently for each gene, destroying biological correlations while preserving each gene's sparsity, then recompute the randomized eigen-spectrum and eigenvector localization statistics. Eigenvalues that depart from RMT universality in the original data but are reproduced by the sparsity-only randomized control are classified as spurious sparsity-induced signal; residual deviating eigenvalues are interpreted as true biological signal. Apply the companion feature-selection filter that discards genes responsible for localized eigenvectors in the randomized case, increasing power to identify potentially interesting biological signals. Report the three fractions — within RMT universality, explained by the sparsity-only control, and residual biological — expecting roughly 95% / 3% / 2% on the Kang et al. PBMC dataset.

## Related tools

- **Randomly** (Python library used to compute the covariance eigen-spectrum, fit Marchenko-Pastur/Tracy-Widom universality predictions, and run the per-gene cell-label randomization control) — https://github.com/luisaparicio/randomly
- **Python 3** (Required runtime — Randomly only works with Python 3, not Python 2)
- **Jupyter Notebook** (Convenient interactive environment for running the Randomly analysis workflow)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- The three computed fractions reproduce the reported decomposition (~95% compatible with RMT universality, ~3% sparsity-spurious, ~2% true biological) within a small tolerance of the Figure 2E / abstract values
- Eigenvector localization observed in the original deviating eigenvalues is reproduced by the randomized sparsity-only control for the spurious component
- Per-gene cell-label permutation preserves each gene's zero (sparsity) statistics while destroying cross-gene cell correlations — verify the randomized matrix has the same sparsity level (~95%)
- The feature-selection filter removes exactly the genes responsible for localized eigenvectors in the randomized case, and downstream biological signal increases after their removal

## Limitations

- Sparsity alone can cause spurious eigenvector localization, falsely identifying meaningful directions in the data — without the randomization control these are easily mistaken for biology
- The method provides no predictive quantitative model of gene expression; the correct distribution and the i.i.d. assumption are unclear
- No version or release information is provided for the publicly available algorithm code, complicating reproducibility

## Evidence
