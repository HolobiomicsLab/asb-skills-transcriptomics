---
name: covariance-restricted-sparse-linear-regression
description: Use when you have TSCS single-cell expression data (qPCR or RNA-seq)
  sampled at consecutive time points and need to infer the transition-driving GRN
  without pseudo-time trajectory construction.
when_to_use_negative:
- Cells are truly linked across time (longitudinal single-cell trajectories available)
  — COSLIR is designed for TSCS data where linkage across time points is absent
- Goal is the dense GRN that maintains a single current cell state rather than the
  sparse GRN driving a transition between consecutive stages
- Input is only a single time point / cell stage with no consecutive-time moment pair,
  since the method requires first and second moments between two consecutive time
  points
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COSLIR (Python version)
  role: 'Primary implementation: solves the covariance-restricted sparse linear regression
    from moment inputs and includes the bootstrapping/clipping ensemble'
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: COSLIR (MATLAB version)
  role: Alternative implementation of the same inference for MATLAB users (tested
    on R2018a)
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Computing first/second moments, bootstrap resampling, and the only basic dependency
    of the Python solver (tested on v1.14)
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:12239d304273e4ef254eaccec4b85fdd76c7098178301474b6e208effa8d97cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-lab-pku/COSLIR
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# covariance-restricted-sparse-linear-regression

## Summary

COSLIR reconstructs the directed, signed, weighted GRN driving a cell-state transition from time-stamped cross-sectional single-cell data by solving a covariance-restricted sparse linear regression (L1-regularized, non-convex) on only the first and second moments of two consecutive time points, stabilized by bootstrapping plus confidence and clipping thresholds.

## When to use

Use when you have TSCS single-cell expression data (qPCR or RNA-seq) sampled at consecutive time points and need to infer the transition-driving GRN without pseudo-time trajectory construction.

## When NOT to use

- Cells are truly linked across time (longitudinal single-cell trajectories available) — COSLIR is designed for TSCS data where linkage across time points is absent
- Goal is the dense GRN that maintains a single current cell state rather than the sparse GRN driving a transition between consecutive stages
- Input is only a single time point / cell stage with no consecutive-time moment pair, since the method requires first and second moments between two consecutive time points

## Inputs

- Time-stamped cross-sectional single-cell expression data (qPCR or scRNA-seq) at two consecutive time points
- Sample mean vector and sample covariance matrix between consecutive time points (the only required inputs)
- Bootstrap settings: confidence threshold and clipping threshold

## Outputs

- Sparse directed, signed, weighted gene-by-gene interaction matrix A_t (GRN driving the cell-state transition)
- Thresholded interaction matrix (e.g. thresholded_interaction_matrix.csv) with zeros for entries removed by confidence and clipping thresholds
- Optional precision/recall curves vs. gene number, cell number, and confidence threshold (e.g. precision_recall_curves.png)

## How to apply

Compute the sample mean vector and covariance matrix of gene expression between two consecutive time points — these moment estimates are the only inputs COSLIR requires. Solve the L1-regularized covariance-restricted optimization (Eq. 3) with the provided ADMM splitting: introduce B and C with constraints A+I=(B+C)/2 and B=C, so each single-variable subproblem is quadratic and admits closed-form updates (least-squares-type updates for the regression blocks, soft-thresholding for sparsity); iterate until primal and dual residuals fall below tolerance. Because moment estimation from finite cells is imprecise, repeat the fit on bootstrap resamples (drawn with replacement from the original cells), aggregate into an ensemble estimator, retain only entries whose ensemble support exceeds the confidence threshold, then apply the clipping threshold zeroing small-magnitude entries — surviving signed entries constitute the final interaction matrix. Use the Python version (NumPy only, tested on NumPy 1.14 / Python 3.6.4) or MATLAB version (R2018a); check the DESCRIPTION file in the version folder for usage details.

## Related tools

- **COSLIR (Python version)** (Primary implementation: solves the covariance-restricted sparse linear regression from moment inputs and includes the bootstrapping/clipping ensemble) — https://github.com/Ge-lab-pku/COSLIR
- **COSLIR (MATLAB version)** (Alternative implementation of the same inference for MATLAB users (tested on R2018a)) — https://github.com/Ge-lab-pku/COSLIR
- **NumPy** (Computing first/second moments, bootstrap resampling, and the only basic dependency of the Python solver (tested on v1.14))

## Examples

```
import numpy as np; mu, Sigma = X_t0.mean(0), np.cov(X_t0, X_t1.T); A = coslir_admm(mu, Sigma, lam, tol=1e-6)
```

## Evaluation signals

- Oracle simulation should attain perfect accuracy in recovering the true directed, signed interaction matrix
- Precision and recall of recovered edges should increase as the number of cells grows (Fig. 2C trend)
- Convergence of the ADMM solver: primal and dual residuals must fall below the convergence tolerance over iterations
- Final matrix is square (gene-by-gene), signed, sparse, with zeros exactly at entries removed by the confidence and clipping thresholds
- Running time should remain nearly independent of the number of cells

## Limitations

- The distributed COSLIR is a beta version; the README states 'Some bugs may still remain, so please use COSLIR with caution at present'
- Performance depends on the quality of estimated sample mean and covariance; the bootstrap + clipping ensemble is the built-in safeguard against this imprecision
- Runtime is nearly cell-number independent, but performance varies with gene count, cell count, and confidence threshold (Fig. 2B-D)

## Evidence

- [abstract] abstract: "utilizing only the first and second moments of samples between two consecutive time points"
- [methods] methods: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)"
- [readme] readme: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
