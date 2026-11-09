---
name: marchenko-pastur-distribution-fitting
description: Use when a sparsity-filtered cell-gene expression matrix is available
  and you need to identify the bulk eigenvalue spectrum predicted by random matrix
  theory so that departures (localized eigenvectors, outlier eigenvalues) can be read
  as signal.
when_to_use_negative:
- The dataset has not yet had the sparsity filter applied — the paper's first algorithm
  step is to remove genes introducing spurious sparsity effects before fitting the
  MP distribution, since sparsity can cause spurious eigenvector localization.
- The data is a randomized (label-permuted) matrix used only to identify sparsity-induced
  effects — in that case the MP fit describes the null, not biological signal.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Randomly
  role: Python 3 library implementing the RMT denoising workflow, including MP distribution
    fitting on single-cell data
  repo: https://rabadan.c2b2.columbia.edu/html/randomly/
- name: Python 3
  role: Runtime language; Randomly only works with Python 3 (not Python 2)
- name: Jupyter Notebook
  role: Recommended environment for running Randomly interactively
- name: pip
  role: 'Installation channel: pip install randomly'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:0fb092d4df46d7243b45e744239dfb9152b9e2b5e8dc6c0c4cd339adb9a4eebb
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
  - pip
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# marchenko-pastur-distribution-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits the Marchenko-Pastur (MP) distribution to the eigenvalue density of the Wishart (sample covariance) matrix of a sparsity-filtered single-cell expression matrix via gradient descent, to locate the MP edge and separate random-matrix noise from biological signal.

## When to use

Use when a sparsity-filtered cell-gene expression matrix is available and you need to identify the bulk eigenvalue spectrum predicted by random matrix theory so that departures (localized eigenvectors, outlier eigenvalues) can be read as signal.

## When NOT to use

- The dataset has not yet had the sparsity filter applied — the paper's first algorithm step is to remove genes introducing spurious sparsity effects before fitting the MP distribution, since sparsity can cause spurious eigenvector localization.
- The data is a randomized (label-permuted) matrix used only to identify sparsity-induced effects — in that case the MP fit describes the null, not biological signal.

## Inputs

- sparsity-filtered single-cell expression matrix (chained output of the sparsity-filter step, e.g. filtered_expression_matrix from task_001)
- cell-gene expression matrix (e.g. 6,573 human PBMCs from Kang et al., ~95% sparse)

## Outputs

- eigenvalue_spectrum.csv containing the fitted MP edge and the eigenvalue spectrum of the Wishart matrix
- empirical eigenvalue density of the Wishart matrix
- fitted Marchenko-Pastur edge (threshold separating noise from signal)

## How to apply

Load the sparsity-filtered single-cell expression matrix produced by the upstream sparsity-filter step (task_001/filtered_expression_matrix). Compute the sample covariance (Wishart) matrix with NumPy matrix operations, then compute its full eigenvalue spectrum and build the empirical eigenvalue density. Fit the non-parametric Marchenko-Pastur distribution to the empirical density via gradient descent to locate the fitted MP edge, and write the fitted edge and spectrum to eigenvalue_spectrum.csv. The rationale is the paper's decomposition of single-cell data into a random matrix (noise), a sparsity-induced non-biological signal, and a biological signal: eigenvalues within the MP support are noise, and large deviations from the MP/TW universality distributions indicate signal. Validation requires the bulk of the empirical eigenvalue density to be compatible with the fitted MP support, consistent with the finding that roughly 95% of the information in single-cell data matches random matrix theory predictions.

## Related tools

- **Randomly** (Python 3 library implementing the RMT denoising workflow, including MP distribution fitting on single-cell data) — https://rabadan.c2b2.columbia.edu/html/randomly/
- **Python 3** (Runtime language; Randomly only works with Python 3 (not Python 2))
- **Jupyter Notebook** (Recommended environment for running Randomly interactively)
- **pip** (Installation channel: pip install randomly)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- The bulk of the empirical eigenvalue density must be compatible with the fitted MP distribution support (paper reports ~95% of information compatible with RMT predictions).
- The fitted MP edge should lie where the empirical eigenvalue density drops off; eigenvalues beyond the edge are candidate signal directions.
- The eigenvalue spectrum and fitted MP edge are correctly serialized in eigenvalue_spectrum.csv.
- The Wishart matrix is computed from the sparsity-filtered (not raw) expression matrix, so sparsity-induced localized eigenvectors (~3% spurious signal) are not mistaken for signal.

## Limitations

- Requires the upstream sparsity filter first: sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data.
- Randomly only works with Python 3 (not Python 2).
- No version or release information is provided for the publicly available algorithm code, complicating reproducibility.

## Evidence
