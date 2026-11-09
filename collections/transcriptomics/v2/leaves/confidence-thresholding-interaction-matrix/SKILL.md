---
name: confidence-thresholding-interaction-matrix
description: Use when reconstructing sparse directed GRNs from time-stamped cross-sectional
  (TSCS) single-cell expression data with COSLIR and you must suppress unstable or
  spurious interaction estimates via bootstrapped confidence and magnitude clipping.
when_to_use_negative:
- The goal is to infer the dense GRN maintaining the current cell stage rather than
  the sparse network driving a transition between consecutive time points.
- Only a single cell stage/time point is available, so no first and second moments
  between consecutive time points can be computed.
- The data are true longitudinal time series with linked cells across time and pseudo-time
  trajectory methods are appropriate.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: COSLIR (Python version)
  role: Refit repeatedly on bootstrap-resampled cells to produce the coefficient matrices
    that form the ensemble estimator
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Compute the first and second (covariance) moments between consecutive time
    points; only basic dependency of the Python version
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: Python
  role: Runtime for the Python version of COSLIR used in the bootstrapping ensemble
    workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:cfcd5c6ef9851737eb2efe717ee4021988801d84be8baf191f699880d8fed556
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-lab-pku/COSLIR
  tools:
  - COSLIR (Python version)
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# confidence-thresholding-interaction-matrix

## Summary

Stabilizes the COSLIR estimator of the gene-gene interaction matrix by bootstrapping: the covariance-restricted sparse linear regression is refit on resampled cells, coefficients are aggregated into an ensemble, and only entries surviving both a confidence threshold and a small-magnitude clipping threshold are retained as significant.

## When to use

Use when reconstructing sparse directed GRNs from time-stamped cross-sectional (TSCS) single-cell expression data with COSLIR and you must suppress unstable or spurious interaction estimates via bootstrapped confidence and magnitude clipping.

## When NOT to use

- The goal is to infer the dense GRN maintaining the current cell stage rather than the sparse network driving a transition between consecutive time points.
- Only a single cell stage/time point is available, so no first and second moments between consecutive time points can be computed.
- The data are true longitudinal time series with linked cells across time and pseudo-time trajectory methods are appropriate.

## Inputs

- Time-stamped cross-sectional (TSCS) single-cell expression data (qPCR or single-cell RNA-seq) at two or more consecutive time points
- First and second (covariance) moments between consecutive time points, computed with NumPy
- Confidence threshold for ensemble support and a small-magnitude clipping cutoff

## Outputs

- thresholded_interaction_matrix.csv — a square gene-by-gene matrix with signed weights
- Sparse directed gene regulatory network in which entries removed by the confidence or clipping thresholds are zero

## How to apply

Compute the first and second (covariance) moments between consecutive time points from the TSCS expression data using NumPy. Repeatedly run COSLIR, each time fitting the covariance-restricted sparse linear regression on a dataset resampled with replacement from the original cells. Aggregate the coefficient matrices across resampled runs into an ensemble estimator for each regulatory interaction; retain only entries whose ensemble support exceeds the confidence threshold. Then apply the clipping threshold, zeroing out retained entries whose magnitudes fall below the small-magnitude cutoff, so that only non-zero entries surviving both filters are kept as statistically significant. The rationale is to guard against imprecision from estimating the sample mean and covariance, exploiting the expected sparsity of GRNs that drive cell-state transitions.

## Related tools

- **COSLIR (Python version)** (Refit repeatedly on bootstrap-resampled cells to produce the coefficient matrices that form the ensemble estimator) — https://github.com/Ge-lab-pku/COSLIR
- **NumPy** (Compute the first and second (covariance) moments between consecutive time points; only basic dependency of the Python version) — https://github.com/Ge-lab-pku/COSLIR
- **Python** (Runtime for the Python version of COSLIR used in the bootstrapping ensemble workflow)

## Examples

```
python COSLIR.py  # run the Python version per the repo's DESCRIPTION file; consult the corresponding folder's DESCRIPTION for usage details
```

## Evaluation signals

- The output file thresholded_interaction_matrix.csv exists.
- The output is a square gene-by-gene matrix with signed weights.
- All entries removed by the confidence and clipping thresholds are zeros in the output, consistent with the sparse transition-driving GRN structure.
- Only non-zero entries that survive both the confidence and clipping filters are retained as statistically significant.
- COSLIR's Python version runs with NumPy as the only basic requirement (tested on v1.14, Python v3.6.4).

## Limitations

- This is a beta version of COSLIR; some bugs may still remain, so it should be used with caution and bug reports are welcomed.
- COSLIR relies exclusively on the first and second moments of the measured samples, so imprecision in estimating the sample mean and covariance propagates into the estimator; bootstrapping with confidence and clipping thresholds is the paper's mitigation.
- The method assumes GRNs driving cell state transitions are sparse; it is not designed for dense state-maintenance networks.

## Evidence

- [readme] bootstrapping with confidence and clipping: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [abstract] moments-only inputs: "utilizing only the first and second moments of samples between two consecutive time points"
- [intro] sparse transition networks: "GRNs that drive cell state transitions are considered sparse"
- [methods] runtime environment: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
