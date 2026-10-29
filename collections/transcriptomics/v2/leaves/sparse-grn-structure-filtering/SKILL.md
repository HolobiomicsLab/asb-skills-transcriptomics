---
name: sparse-grn-structure-filtering
description: Use when you have time-stamped cross-sectional (TSCS) single-cell qPCR
  or RNA-seq expression data from consecutive time points and need a stable, sparse,
  signed, directed gene regulatory network despite imprecision in estimating sample
  mean and covariance.
when_to_use_negative:
- The goal is a network maintaining the current cell state at a single time point/stage
  rather than the mechanism driving a transition along the cell lineage (the article
  notes such GRNs are dense rather than sparse).
- You do not have at least two consecutive time points of TSCS data — COSLIR relies
  exclusively on the first and second moments of samples between two consecutive time
  points.
- A single un-bootstrapped COSLIR fit is acceptable and the extra bootstrap/ensemble
  runtime is not warranted.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COSLIR (Python version)
  role: Fits the covariance-restricted sparse linear regression on each bootstrap-resampled
    dataset to produce the ensemble of coefficient matrices being thresholded
  repo: github.com/Ge-lab-pku/COSLIR
- name: Python 3.6.4
  role: Runtime environment in which the Python COSLIR bootstrapping and thresholding
    workflow is executed
- name: NumPy 1.14
  role: Computes the first and second (covariance) moments between consecutive time
    points and aggregates the bootstrapped coefficient matrices
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:a18ce584c43c79cedbdb8c816bec546eb2b549ba1568275d552ae3dde9c76b93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (Python version)
  - NumPy
  - NumPy 1.14
  - Python 3.6.4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sparse-grn-structure-filtering

## Summary

Stabilizes COSLIR's estimator of the gene-gene interaction matrix by bootstrapping the covariance-restricted sparse linear regression, then retaining only non-zero entries that survive both a confidence threshold and a clipping threshold, yielding the final sparse directed GRN with signs and weights.

## When to use

Use when you have time-stamped cross-sectional (TSCS) single-cell qPCR or RNA-seq expression data from consecutive time points and need a stable, sparse, signed, directed gene regulatory network despite imprecision in estimating sample mean and covariance.

## When NOT to use

- The goal is a network maintaining the current cell state at a single time point/stage rather than the mechanism driving a transition along the cell lineage (the article notes such GRNs are dense rather than sparse).
- You do not have at least two consecutive time points of TSCS data — COSLIR relies exclusively on the first and second moments of samples between two consecutive time points.
- A single un-bootstrapped COSLIR fit is acceptable and the extra bootstrap/ensemble runtime is not warranted.

## Inputs

- time-stamped cross-sectional (TSCS) single-cell qPCR or RNA-seq expression data from two consecutive time points
- bootstrap-resampled coefficient (interaction) matrices produced by repeated COSLIR covariance-restricted sparse linear regression runs
- a confidence threshold on ensemble support and a small-magnitude clipping threshold

## Outputs

- thresholded_interaction_matrix.csv — a square gene-by-gene matrix of signed interaction weights, with zeros for all entries removed by the confidence and clipping thresholds

## How to apply

Load the time-stamped cross-sectional single-cell expression data and compute the first and second (covariance) moments between consecutive time points using NumPy. Repeatedly run the Python version of COSLIR, each run fitting the covariance-restricted sparse linear regression (COSLIR model) on a dataset resampled with replacement from the original cells. Aggregate the coefficient matrices from all resampled runs into an ensemble estimator for each regulatory interaction, then retain only interaction entries whose ensemble support exceeds the confidence threshold. Apply the clipping threshold, zeroing out retained entries whose magnitudes fall below the small-magnitude cutoff — per the README, COSLIR's workflow is 'to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence.' This double filtering is justified because GRNs driving cell state transitions are sparse, and only entries surviving both filters are treated as statistically significant and kept with their mean value as the final interaction-matrix estimator.

## Related tools

- **COSLIR (Python version)** (Fits the covariance-restricted sparse linear regression on each bootstrap-resampled dataset to produce the ensemble of coefficient matrices being thresholded) — github.com/Ge-lab-pku/COSLIR
- **Python 3.6.4** (Runtime environment in which the Python COSLIR bootstrapping and thresholding workflow is executed)
- **NumPy 1.14** (Computes the first and second (covariance) moments between consecutive time points and aggregates the bootstrapped coefficient matrices)

## Evaluation signals

- thresholded_interaction_matrix.csv exists and is a square gene-by-gene matrix with signed weights
- all entries removed by the confidence and clipping thresholds are exactly zero in the output
- the retained (non-zero) entries are only those whose ensemble support across bootstrap runs exceeds the confidence threshold
- retained entries have magnitudes above the small-magnitude clipping cutoff; no sub-cutoff values survive
- the resulting network is sparse, consistent with the sparse structure of transition-driving GRNs reported in the paper

## Limitations

- The repository README states this is a beta version of COSLIR: 'Some bugs may still remain, so please use COSLIR with caution at present.'
- Because only first and second moments between two consecutive time points are used, no linkage across time points (pseudo-time ordering of individual cells) is exploited or recoverable.
- Bootstrap ensemble stability depends on the accuracy of the estimated sample mean and covariance from TSCS data; entries near the confidence or clipping thresholds are sensitive to sampling noise.

## Evidence

- [readme] readme workflow quote: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [readme] readme install/env quote: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
- [intro] sparsity motivation: "GRNs that drive cell state transitions are considered sparse"
- [abstract] moments-only basis: "utilizing only the first and second moments of samples between two consecutive time points"
- [readme] beta-version caveat: [evidence span withheld — anchor status: external_doc]
