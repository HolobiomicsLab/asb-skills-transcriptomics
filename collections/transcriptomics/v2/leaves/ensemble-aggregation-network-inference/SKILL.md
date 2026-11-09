---
name: ensemble-aggregation-network-inference
description: Use when reconstructing a directed, signed GRN from time-stamped cross-sectional
  single-cell expression data and the raw A_t estimate is unstable because the sample
  mean and covariance must themselves be estimated.
when_to_use_negative:
- The goal is to infer a network maintaining a single cell stage rather than the mechanism
  driving a cell-state transition — such dense, single-time-point networks are outside
  COSLIR's scope.
- True longitudinal single-cell time series (temporal linkage across time points)
  is available and pseudo-time inference is not a concern, so moment-based bootstrapped
  estimation adds unnecessary machinery.
- Input data are not time-stamped cross-sectional samples from which first and second
  moments between consecutive time points can be computed.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0749
tools:
- name: COSLIR (Python version)
  role: Core estimator run repeatedly on bootstrapped resamples; each run fits the
    covariance-restricted sparse linear regression whose coefficient matrices are
    aggregated into the ensemble
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Computes the first and second (covariance) moments between consecutive time
    points; only basic requirement of the Python COSLIR
- name: Python 3.6.4
  role: Runtime environment for the bootstrapping ensemble workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:4490d0c2f5ecf4b9c707a065c7dad643059d81c20c24287b8318d186a7c8eaad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (Python version)
  - NumPy
  - Python 3.6.4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ensemble-aggregation-network-inference

## Summary

Stabilizes the COSLIR gene-gene interaction matrix estimator by bootstrapping the covariance-restricted sparse linear regression over resampled single cells, then keeping only entries that survive both a confidence threshold and a small-magnitude clipping threshold.

## When to use

Use when reconstructing a directed, signed GRN from time-stamped cross-sectional single-cell expression data and the raw A_t estimate is unstable because the sample mean and covariance must themselves be estimated.

## When NOT to use

- The goal is to infer a network maintaining a single cell stage rather than the mechanism driving a cell-state transition — such dense, single-time-point networks are outside COSLIR's scope.
- True longitudinal single-cell time series (temporal linkage across time points) is available and pseudo-time inference is not a concern, so moment-based bootstrapped estimation adds unnecessary machinery.
- Input data are not time-stamped cross-sectional samples from which first and second moments between consecutive time points can be computed.

## Inputs

- Time-stamped cross-sectional (TSCS) single-cell expression data (qPCR or scRNA-seq) with consecutive time points
- First and second (covariance) moments between consecutive time points
- Confidence threshold and small-magnitude clipping cutoff parameters

## Outputs

- thresholded_interaction_matrix.csv — a square gene-by-gene interaction matrix with signed weights and zeros for entries removed by the confidence and clipping thresholds
- Sparse directed gene regulatory network with signs and weights

## How to apply

Compute the first and second (covariance) moments between consecutive time points from the TSCS single-cell expression data using NumPy. Repeatedly run the Python version of COSLIR, each time fitting the covariance-restricted sparse linear regression on a dataset resampled with replacement from the original cells, which hedges against imprecision in estimating the sample moments. Aggregate the per-run coefficient matrices into an ensemble estimator for each regulatory interaction. Retain only entries whose ensemble support exceeds the confidence threshold, then apply the clipping threshold by zeroing retained entries whose magnitudes fall below the small-magnitude cutoff; entries surviving both filters constitute the final sparse directed network with signs and weights, written to thresholded_interaction_matrix.csv.

## Related tools

- **COSLIR (Python version)** (Core estimator run repeatedly on bootstrapped resamples; each run fits the covariance-restricted sparse linear regression whose coefficient matrices are aggregated into the ensemble) — github.com/Ge-lab-pku/COSLIR
- **NumPy** (Computes the first and second (covariance) moments between consecutive time points; only basic requirement of the Python COSLIR)
- **Python 3.6.4** (Runtime environment for the bootstrapping ensemble workflow)

## Examples

```
python run_coslir_bootstrap.py --input expression_timeseries.csv --n_bootstrap 100 --confidence 0.8 --clip 0.01 --output thresholded_interaction_matrix.csv
```

## Evaluation signals

- thresholded_interaction_matrix.csv exists and is a square gene-by-gene matrix
- All entries carry signed weights (positive or negative), consistent with a directed network with signs
- Every entry removed by either the confidence threshold or the clipping threshold is exactly zero in the output matrix
- The surviving nonzero entries are precisely those that pass both filters (ensemble support above the confidence threshold AND magnitude above the clipping cutoff)
- The resulting network is sparse, consistent with the paper's premise that GRNs driving cell-state transitions are sparse

## Limitations

- COSLIR is a beta version; the README warns that 'Some bugs may still remain, so please use COSLIR with caution at present.'
- Stability of the ensemble depends on the choice of the confidence threshold and the small-magnitude clipping cutoff; the sources do not specify default numeric values.
- The method relies exclusively on the first and second moments of the measured samples, so information beyond moments between consecutive time points is not exploited.

## Evidence

- [results] bootstrapping and clipping thresholding: [evidence span withheld — anchor status: not_found]
- [full_text] workflow: [evidence span withheld — anchor status: not_found]
- [readme] readme-workflow: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [readme] readme-beta: "Note that this is a beta version of COSLIR. Some bugs may still remain, so please use COSLIR with caution at present."
- [methods] methods-runtime: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
