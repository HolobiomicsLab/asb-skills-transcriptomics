---
name: single-cell-expression-resampling-bootstrapping
description: Use when estimating a transition-driving GRN from time-stamped cross-sectional
  (TSCS) single-cell data and you need to guard the interaction matrix against imprecision
  in estimated sample mean/covariance moments.
when_to_use_negative:
- The input is a single time point / single cell stage, where the goal is to infer
  the network maintaining the current cell state rather than the mechanism driving
  cell state transitions along the lineage
- True longitudinal time-series data with real linkage across time points is available,
  so pseudo-time-free moment-based modeling is unnecessary
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: COSLIR (Python version)
  role: Fit the covariance-restricted sparse linear regression on each bootstrap resample
    of the single-cell expression data
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy 1.14
  role: Compute the first and second (covariance) moments between consecutive time
    points and perform the resampling with replacement
- name: Python 3.6.4
  role: Runtime environment in which the bootstrapping ensemble and thresholding are
    executed
- name: COSLIR (MATLAB version)
  role: Alternative runtime for running the same covariance-restricted sparse regression
    resampling workflow
  repo: github.com/Ge-lab-pku/COSLIR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:6a963bbd2fc5271bd178026b8bb0b9c5c0e1c8d466e6c8289f3fe49def315751
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  - NumPy 1.14
  - Python 3.6.4
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-expression-resampling-bootstrapping

## Summary

Stabilizes COSLIR's estimator of the gene-gene interaction matrix by repeatedly refitting the covariance-restricted sparse linear regression on bootstrap resamples (with replacement) of single-cell expression data, then aggregating ensemble estimates and thresholding on confidence and small-magnitude clipping to yield a sparse, signed, weighted directed GRN.

## When to use

Use when estimating a transition-driving GRN from time-stamped cross-sectional (TSCS) single-cell data and you need to guard the interaction matrix against imprecision in estimated sample mean/covariance moments.

## When NOT to use

- The input is a single time point / single cell stage, where the goal is to infer the network maintaining the current cell state rather than the mechanism driving cell state transitions along the lineage
- True longitudinal time-series data with real linkage across time points is available, so pseudo-time-free moment-based modeling is unnecessary

## Inputs

- Time-stamped cross-sectional (TSCS) single-cell expression data (qPCR or single-cell RNA-seq) with samples assigned to consecutive time points
- First and second (covariance) moments between consecutive time points computed from the expression matrix

## Outputs

- Final sparse gene-by-gene interaction matrix with signed, weighted entries and zeros for all entries removed by the confidence and clipping thresholds
- thresholded_interaction_matrix.csv (square gene-by-gene matrix of signed weights)

## How to apply

First compute the first and second (covariance) moments between consecutive time points from the time-stamped cross-sectional single-cell expression data using NumPy. Then repeatedly run COSLIR's covariance-restricted sparse linear regression, each fit on a dataset resampled with replacement from the original cells, producing one coefficient matrix per resample. Aggregate the coefficient matrices across resampled runs into an ensemble estimator for each regulatory interaction, which corrects for instability of the moment-based estimator. Retain only interaction entries whose ensemble support exceeds the confidence threshold, then apply the clipping threshold that zeroes out retained entries whose magnitudes fall below the small-magnitude cutoff. Only entries surviving both filters are kept as statistically significant, and their mean value constitutes the final interaction-matrix estimator, yielding a sparse directed network with signs and weights.

## Related tools

- **COSLIR (Python version)** (Fit the covariance-restricted sparse linear regression on each bootstrap resample of the single-cell expression data) — github.com/Ge-lab-pku/COSLIR
- **NumPy 1.14** (Compute the first and second (covariance) moments between consecutive time points and perform the resampling with replacement)
- **Python 3.6.4** (Runtime environment in which the bootstrapping ensemble and thresholding are executed)
- **COSLIR (MATLAB version)** (Alternative runtime for running the same covariance-restricted sparse regression resampling workflow) — github.com/Ge-lab-pku/COSLIR

## Examples

```
import numpy as np; idx = np.random.choice(n_cells, size=n_cells, replace=True); X_boot, y_boot = X[idx], y[idx]
```

## Evaluation signals

- The output file thresholded_interaction_matrix.csv exists and is a square gene-by-gene matrix with signed weights
- The output contains zeros for all entries removed by the confidence and clipping thresholds, consistent with the sparse structure of transition-driving GRNs reported in the paper
- Only non-zero entries that survive BOTH the confidence threshold and the clipping (small-magnitude) threshold are retained
- Retained entries equal the mean value of the coefficient matrices across all resampled runs, aggregated into the ensemble estimator
- Each bootstrap run fits the model on a dataset resampled with replacement from the original cells; the number of resampled runs matches the ensemble size

## Limitations

- This is a beta version of COSLIR; some bugs may still remain, and users are advised to use COSLIR with caution at present
- COSLIR relies exclusively on the first and second moments of the measured samples, so it requires time-stamped cross-sectional data grouped into at least two consecutive time points and cannot exploit real longitudinal linkage

## Evidence

- [readme] The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence.: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [other] After bootstrapping and clipping thresholding, only non-zero entries that survive both filters are kept as statistically significant, and their mean value constitutes the final interaction-matrix estimator.: [evidence span withheld — anchor status: not_found]
- [other] Retain only interaction entries whose ensemble support exceeds the confidence threshold. Apply the clipping threshold, zeroing out retained entries whose magnitudes fall below the small-magnitude cutoff: [evidence span withheld — anchor status: not_found]
- [abstract] utilizing only the first and second moments of samples between two consecutive time points: "utilizing only the first and second moments of samples between two consecutive time points"
- [readme] COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14).: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
