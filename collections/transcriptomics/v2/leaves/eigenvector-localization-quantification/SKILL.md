---
name: eigenvector-localization-quantification
description: Use when a highly sparse single-cell expression matrix (e.g. ~95% zeros,
  6,573 PBMCs x genes) shows localized eigenvectors or an unexpected elbow in a PCA
  scree plot, and you must decide whether these are sparsity artifacts or biological
  signal.
when_to_use_negative:
- Data are dense or near-dense, so sparsity cannot drive eigenvector localization
  and the sparsity-control comparison is unnecessary
- Goal is cell-type clustering or imputation (e.g. MAGIC/scImpute) rather than deciding
  which spectral components are noise, sparsity artifacts, or signal
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: Python 3 library used to compute covariance eigenspectra, eigenvector localization
    statistics, and the label-permutation randomization control
  repo: https://github.com/luisaparicio/randomly
- name: Jupyter Notebook
  role: Recommended interactive environment for running the Randomly eigenspectrum
    and localization analysis
  repo: http://jupyter.org/
- name: pip
  role: Command-line installer for the randomly package
  repo: https://pypi.python.org/pypi/randomly
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:0208e84688f5a8631755ff51e0497efcba201b6337fbdb03ddc5feeb3cde7f3c
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Eigenvector localization quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies eigenvector localization and upper-edge eigenvalue deviations in single-cell covariance spectra, separating sparsity-induced artifacts from true biological signal by benchmarking against a label-permuted randomization and RMT universality (Marchenko-Pastur/Tracy-Widom).

## When to use

Use when a highly sparse single-cell expression matrix (e.g. ~95% zeros, 6,573 PBMCs x genes) shows localized eigenvectors or an unexpected elbow in a PCA scree plot, and you must decide whether these are sparsity artifacts or biological signal.

## When NOT to use

- Data are dense or near-dense, so sparsity cannot drive eigenvector localization and the sparsity-control comparison is unnecessary
- Goal is cell-type clustering or imputation (e.g. MAGIC/scImpute) rather than deciding which spectral components are noise, sparsity artifacts, or signal

## Inputs

- Sparse cell-gene expression matrix (e.g. 95% sparse counts from 6,573 human PBMCs, Kang et al.)
- Randomized control matrix produced by independently permuting cell labels per gene
- RMT universality reference distributions (Marchenko-Pastur bulk, Tracy-Widom edge)

## Outputs

- Covariance eigenspectrum with classification of eigenvalues as RMT-compatible or deviating
- Eigenvector localization statistics for original and randomized matrices
- Scree plot exposing the artifactual elbow in randomized sparse data
- eigenvalue_fraction_report.csv with the three spectral fractions (~95% RMT / ~3% sparsity / ~2% biological)

## How to apply

Load the sparse cell-gene matrix (e.g. the 95% sparse Kang et al. PBMC matrix of 6,573 cells) and compute the covariance-matrix eigenspectrum with the Randomly library in Python 3. Compare the bulk of the spectrum against the Marchenko-Pastur bulk and Tracy-Widom edge statistics, flagging eigenvalues that depart from universality. Randomize the dataset by permuting cell labels independently for each gene — this destroys biological correlation while preserving each gene's sparsity profile — and recompute eigenvalues and eigenvector localization statistics; deviating eigenvalues that persist in the randomized control are sparsity-induced, not biological. Localized eigenvectors in the original data that vanish after removing the genes responsible for randomized-case localization can then be interpreted as groups of correlated cells (biological signal). Partition the spectrum into three fractions: compatible with RMT universality (~95%), explained by the sparsity-only control (~3%), and residual true biological signal (~2%), writing them to eigenvalue_fraction_report.csv.

## Related tools

- **Randomly** (Python 3 library used to compute covariance eigenspectra, eigenvector localization statistics, and the label-permutation randomization control) — https://github.com/luisaparicio/randomly
- **Jupyter Notebook** (Recommended interactive environment for running the Randomly eigenspectrum and localization analysis) — http://jupyter.org/
- **pip** (Command-line installer for the randomly package) — https://pypi.python.org/pypi/randomly

## Examples

```
pip install randomly; python -c "import randomly"
```

## Evaluation signals

- Reproduce the three-way decomposition: ~95% of eigenvalues compatible with RMT universality, ~3% spurious sparsity-induced signal, ~2% true biological signal (Figure 2E / abstract tolerance)
- Randomized control (cell labels permuted per gene) still shows localized eigenvectors, upper-edge deviations from TW/MP, and an artifactual scree elbow (Figure 2A-2C), proving the artifacts are sparsity-only
- Genes responsible for localization in the randomized case, once discarded, remove localization in the original data attributable to sparsity while retaining biological deviating eigenvalues
- Eigenspectrum computed with the randomly package in Python 3 runs without Python 2 compatibility errors

## Limitations

- Sparsity alone can cause spurious eigenvector localization, falsely identifying meaningful directions in the data, so localized eigenvectors must never be read as biological without the randomized control
- Predictive quantitative models of gene expression are lacking, so the correct expression distribution and i.i.d. assumptions are unclear; RMT universality is used precisely to avoid distribution-specific assumptions
- No version or release information is provided for the publicly available algorithm code, complicating reproducibility

## Evidence
