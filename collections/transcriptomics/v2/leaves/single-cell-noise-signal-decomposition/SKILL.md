---
name: single-cell-noise-signal-decomposition
description: Use when a sparse single-cell expression matrix (e.g. scRNA-seq counts
  with ~95% zeros) must be split into noise, sparsity-driven artifacts, and true biological
  signal without assuming i.i.d. noise distributions.
when_to_use_negative:
- Input is not a single-cell expression matrix (the Randomly library targets single-cell
  sequencing data specifically).
- The analysis requires explicit parametric assumptions about gene expression distributions
  or i.i.d. noise — this method is designed to avoid such assumptions, not to provide
  them.
- The goal is imputation or zero-inflated dimensionality reduction (e.g. MAGIC, scImpute,
  ZIFA, ZIMB-WaVE) rather than RMT-based noise/signal decomposition.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Randomly
  role: Python 3 library that performs the RMT-based eigendecomposition, sparsity
    filtering, and noise/signal decomposition of single-cell data
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime environment; Randomly only works with Python 3 (not Python 2)
- name: pip
  role: Installation channel for Randomly (`pip install randomly`)
- name: Jupyter Notebook
  role: Recommended environment for running Randomly interactively
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:5171ce5317c3e6ed4251bdf254a42963754cc36d8dbbeed69525260566e46efc
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
  - pip
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-noise-signal-decomposition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Decomposes single-cell RNA-seq data into a random-matrix (noise) component, a sparsity-induced non-biological component, and a biological signal using random matrix theory (RMT) eigenvalue/eigenvector statistics, implemented via the Randomly Python library.

## When to use

Use when a sparse single-cell expression matrix (e.g. scRNA-seq counts with ~95% zeros) must be split into noise, sparsity-driven artifacts, and true biological signal without assuming i.i.d. noise distributions.

## When NOT to use

- Input is not a single-cell expression matrix (the Randomly library targets single-cell sequencing data specifically).
- The analysis requires explicit parametric assumptions about gene expression distributions or i.i.d. noise — this method is designed to avoid such assumptions, not to provide them.
- The goal is imputation or zero-inflated dimensionality reduction (e.g. MAGIC, scImpute, ZIFA, ZIMB-WaVE) rather than RMT-based noise/signal decomposition.

## Inputs

- Single-cell cell-gene expression matrix (e.g. 95% sparse, such as 6,573 human PBMCs from Kang et al. or 3,005 mouse cortex cells from Zeisel et al.)
- Eigendecomposition of the single-cell covariance matrix (eigenvalues and eigenvector components)

## Outputs

- Per-eigenvector Shapiro-Wilk statistics, p-values, Marchenko-Pastur bulk membership, and retain/discard decisions (e.g. eigenvector_normality_flags.csv)
- Genes discarded as sparsity-driven (responsible for localized eigenvectors in the randomized data)
- Set of genes responsible for potentially interesting biological signals
- Low-rank denoised representation of the single-cell data for further interpretation

## How to apply

Model the single-cell dataset as a random matrix (noise) plus a low-rank perturbation (signal), so that large deviations from RMT universality distributions (Tracy-Widom / Marchenko-Pastur) on the covariance-matrix spectrum flag signal. First suppress sparsity effects by randomizing the dataset — permuting cell labels for each gene independently — and discarding genes responsible for localized eigenvectors in the randomized (sparsity-only) case; this removes spurious localization that can falsely identify meaningful directions. Then, on the filtered data, examine eigenvalue positions relative to the Marchenko-Pastur bulk and eigenvector localization statistics: localized eigenvectors are interpreted as groups of cells whose gene expression is correlated. Additionally cross-check candidate eigenvectors with a Shapiro-Wilk normality test on their component distribution — delocalized, Gaussian-profile eigenvectors are discarded as non-informative even when their eigenvalues lie outside the MP bulk. The result is a low-rank representation of the data plus selected genes responsible for potentially interesting biological signals.

## Related tools

- **Randomly** (Python 3 library that performs the RMT-based eigendecomposition, sparsity filtering, and noise/signal decomposition of single-cell data) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime environment; Randomly only works with Python 3 (not Python 2))
- **pip** (Installation channel for Randomly (`pip install randomly`))
- **Jupyter Notebook** (Recommended environment for running Randomly interactively)

## Examples

```
pip install randomly
```

## Evaluation signals

- Approximately 95% of the information in the dataset should be compatible with random matrix theory predictions, ~3% flagged as spurious sparsity-induced signal, and ~2% as true biological signal.
- Localized eigenvectors retained as signal candidates must be non-Gaussian under the Shapiro-Wilk test on their component distribution.
- Delocalized (Gaussian-component) eigenvectors must be discarded even when their eigenvalues lie outside the Marchenko-Pastur bulk.
- Genes removed in the sparsity filter should be exactly those causing eigenvector localization in the randomized (cell-label-permuted) dataset.
- Feature selection for sparsity should increase the power for identifying potentially interesting biological signals (larger deviations from TW/MP universality after filtering).

## Limitations

- There is no complete quantitative understanding of the different sorts of noise that arise in single-cell measurements, and it is not clear what the correct expression distribution is or whether the i.i.d. assumption is reasonable.
- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data — this must be corrected via the randomized-data filter before interpreting signal.
- No version or release information is provided for the publicly available algorithm code.

## Evidence

- [results] Delocalized Gaussian eigenvectors discarded: [evidence span withheld — anchor status: not_found]
