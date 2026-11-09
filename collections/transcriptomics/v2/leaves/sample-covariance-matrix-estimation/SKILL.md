---
name: sample-covariance-matrix-estimation
description: Use when the input is a sparsity-filtered cell-gene expression matrix
  and the goal is to expose its eigenvalue density for comparison against random matrix
  theory universality (Marchenko-Pastur) predictions.
when_to_use_negative:
- The expression matrix has not yet been sparsity-filtered — spurious eigenvector
  localization from sparsity (~3% of information per the paper) would contaminate
  the Wishart spectrum and bias the MP fit
- The goal is to extract true biological signal (only ~2% of information); this step
  characterizes the random-matrix noise bulk, not the low-rank signal departures from
  MP support
- The input is already a covariance matrix or a small matrix where Marchenko-Pastur
  universality assumptions are inappropriate
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: Library implementing the random matrix theory denoising workflow that includes
    the Wishart matrix computation and Marchenko-Pastur fitting
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime environment; Randomly only works with Python 3 (not Python 2), and
    NumPy matrix operations compute the sample covariance matrix
- name: pip
  role: Installs the randomly package via `pip install randomly`
- name: Jupyter Notebook
  role: Recommended interactive environment for running the Randomly analysis pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:035deb5fd632f0cfd7e791c4ce7b8b6b2b497b8d87b52519142018f2cb4d001a
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sample-covariance-matrix-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the sample covariance (Wishart) matrix of a sparsity-filtered single-cell expression matrix and its eigenvalue spectrum, so that the random-matrix (noise) component of the data can be identified by fitting a Marchenko-Pastur distribution via gradient descent.

## When to use

Use when the input is a sparsity-filtered cell-gene expression matrix and the goal is to expose its eigenvalue density for comparison against random matrix theory universality (Marchenko-Pastur) predictions.

## When NOT to use

- The expression matrix has not yet been sparsity-filtered — spurious eigenvector localization from sparsity (~3% of information per the paper) would contaminate the Wishart spectrum and bias the MP fit
- The goal is to extract true biological signal (only ~2% of information); this step characterizes the random-matrix noise bulk, not the low-rank signal departures from MP support
- The input is already a covariance matrix or a small matrix where Marchenko-Pastur universality assumptions are inappropriate

## Inputs

- Sparsity-filtered single-cell cell-gene expression matrix (output of the upstream sparsity-filter step, e.g. filtered_expression_matrix from task_001)

## Outputs

- eigenvalue_spectrum.csv containing the full eigenvalue spectrum of the Wishart matrix and the fitted Marchenko-Pastur edge
- Empirical eigenvalue density of the sample covariance matrix
- Fitted Marchenko-Pastur distribution edge location

## How to apply

Load the sparsity-filtered expression matrix produced upstream (e.g. task_001/filtered_expression_matrix) and compute its sample covariance (Wishart) matrix with NumPy matrix operations. Compute the full eigenvalue spectrum of this Wishart matrix and build the empirical eigenvalue density. Fit the non-parametric Marchenko-Pastur distribution to the empirical density via gradient descent optimization to locate the fitted MP edge. Write the fitted MP edge together with the eigenvalue spectrum to eigenvalue_spectrum.csv. The rationale is the paper's model of single-cell data as a random matrix (noise) plus a low-rank perturbation (signal), so the bulk of the Wishart eigenvalue density should follow the MP law — the paper reports roughly 95% of the information in single-cell data is compatible with random matrix theory predictions.

## Related tools

- **Randomly** (Library implementing the random matrix theory denoising workflow that includes the Wishart matrix computation and Marchenko-Pastur fitting) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime environment; Randomly only works with Python 3 (not Python 2), and NumPy matrix operations compute the sample covariance matrix)
- **pip** (Installs the randomly package via `pip install randomly`)
- **Jupyter Notebook** (Recommended interactive environment for running the Randomly analysis pipeline)

## Examples

```
pip install randomly; import randomly  # then load filtered_expression_matrix and compute the Wishart covariance matrix with NumPy to fit the Marchenko-Pastur edge
```

## Evaluation signals

- The bulk of the empirical eigenvalue density must be compatible with the fitted Marchenko-Pastur distribution support (the paper finds ~95% of information in single-cell data is compatible with RMT predictions)
- The fitted MP edge locates the boundary between the random-matrix bulk and low-rank signal departures, matching the paper's noise/signal decomposition
- eigenvalue_spectrum.csv is produced containing both the eigenvalue spectrum and the fitted MP edge
- Gradient descent on the non-parametric MP fit converges to an edge consistent with the empirical density; eigenvalue counts inside the MP support should reflect the ~95% RMT-compatible fraction

## Limitations

- Requires Python 3; Randomly does not work with Python 2
- Fitting the Marchenko-Pastur distribution is sensitive to prior removal of sparsity-induced signal — the paper notes about 3% of information is spurious signal induced by sparsity and that sparsity can cause spurious eigenvector localization
- Only the departures beyond the fitted MP edge carry biological signal (~2% of information); the Wishart/MP analysis itself cannot interpret that signal

## Evidence
