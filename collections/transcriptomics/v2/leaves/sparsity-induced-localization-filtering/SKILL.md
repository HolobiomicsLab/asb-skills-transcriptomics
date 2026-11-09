---
name: sparsity-induced-localization-filtering
description: Use when a sparse cell-gene expression matrix yields covariance eigenvectors
  that localize outside Marchenko-Pastur expectations, and genes driving spurious
  (sparsity-induced) directions must be discarded before signal gene selection.
when_to_use_negative:
- The input is already a filtered feature table with sparsity-induced effects removed
  (the randomization/localization filter would be redundant)
- Dense or non-sparse data where sparsity does not drive eigenvector localization
  — the sparsity null has nothing to expose
- The goal is imputation or zero-inflated modeling (MAGIC, scImpute, ZIFA, ZIMB-WaVE,
  scVI) rather than RMT-based denoising/feature selection
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Randomly
  role: Python library that performs the RMT eigendecomposition, dataset randomization,
    eigenvector localization/normality assessment, and signal-like gene selection
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime for Randomly; the library only works with Python 3 (not Python 2)
- name: Jupyter Notebook
  role: Recommended environment for running the Randomly filtering workflow
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:ed637e3624c4977871e10b2a194e078ee70a35b592eeaca843ed43c4f46b649f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# sparsity-induced-localization-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An RMT-based filtering skill from the Randomly paper that separates sparsity-driven spurious eigenvector localization from true biological signal in single-cell covariance spectra, then selects signal-like genes via chi-squared variance testing with FDR correction.

## When to use

Use when a sparse cell-gene expression matrix yields covariance eigenvectors that localize outside Marchenko-Pastur expectations, and genes driving spurious (sparsity-induced) directions must be discarded before signal gene selection.

## When NOT to use

- The input is already a filtered feature table with sparsity-induced effects removed (the randomization/localization filter would be redundant)
- Dense or non-sparse data where sparsity does not drive eigenvector localization — the sparsity null has nothing to expose
- The goal is imputation or zero-inflated modeling (MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI) rather than RMT-based denoising/feature selection

## Inputs

- Sparse cell-gene expression count matrix (e.g. 6,573 human PBMCs from Kang et al. at ~95% sparsity, or 3,005 mouse cortex cells from Zeisel et al.)
- Eigendecomposition of the single-cell expression covariance matrix (eigenvalues and eigenvector components)
- Randomized (label-permuted) version of the expression matrix used as the sparsity-only null

## Outputs

- eigenvector_normality_flags.csv with per-eigenvector Shapiro-Wilk statistics, p-values, Marchenko-Pastur bulk membership, and retain/discard decisions
- signal_like_genes.csv with gene identifier, chi-squared statistic, Benjamini-Hochberg corrected p-value, and rank for each tested gene, with exactly 1,500 genes selected as signal-like
- A low-rank denoised representation of the single-cell data for further interpretation

## How to apply

First randomize the dataset by permuting the cell labels for each gene independently; this exposes sparsity-only effects, and genes responsible for localized eigenvectors in the randomized data are discarded as the first algorithm step. For the real covariance eigendecomposition, run a Shapiro-Wilk test on each eigenvector's component distribution: Gaussian (delocalized) profiles are non-informative and must be discarded even when their eigenvalues lie outside the Marchenko-Pastur bulk, since only localized eigenvectors reflect correlated groups of cells. Partition remaining eigenvectors into the noise bulk (RMT-universality-consistent) and signal outlier subsets, then project every gene onto both subsets to get per-gene variance contributions. Apply a chi-squared test for variance to each gene's signal-versus-noise projections, correct all p-values with Benjamini-Hochberg FDR, and rank genes by corrected significance of signal-like variance (the paper's reported selection is the top 1,500 signal-like genes). Rationale: roughly 95% of the information in single-cell data is RMT-compatible noise, ~3% is spurious signal induced by sparsity, and only ~2% is true biological signal, so both the randomization filter and the delocalized-eigenvector check are needed to avoid mistaking sparsity artifacts for biology.

## Related tools

- **Randomly** (Python library that performs the RMT eigendecomposition, dataset randomization, eigenvector localization/normality assessment, and signal-like gene selection) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime for Randomly; the library only works with Python 3 (not Python 2))
- **Jupyter Notebook** (Recommended environment for running the Randomly filtering workflow)

## Examples

```
pip install randomly; python -c "import randomly"
```

## Evaluation signals

- eigenvector_normality_flags.csv exists and contains one row per eigenvector with Shapiro-Wilk statistic, p-value, MP-bulk membership, and a retain/discard decision
- Eigenvectors with Gaussian component profiles (Shapiro-Wilk) are discarded even when their eigenvalues lie outside the Marchenko-Pastur bulk
- signal_like_genes.csv contains one row per tested gene and exactly 1,500 genes flagged/selected as signal-like, matching the paper's reported selection
- The eigenvector/eigenvalue partition of the real data is contrasted against the randomized data: genes driving localization only in the randomized case are removed
- Most of the covariance spectrum should fall in the RMT universality bulk, consistent with the finding that roughly 95% of single-cell information is random-matrix compatible

## Limitations

- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data — localization alone is not proof of biology without the randomized-data control (~3% of information is spurious signal induced by sparsity)
- Randomly only works with Python 3 (not Python 2)
- No version or release information is provided for the publicly available algorithm code

## Evidence
