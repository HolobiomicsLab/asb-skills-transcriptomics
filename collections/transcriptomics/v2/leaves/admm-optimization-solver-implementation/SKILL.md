---
name: admm-optimization-solver-implementation
description: Use when you must estimate a sparse, signed, weighted regulatory matrix
  from time-stamped cross-sectional (TSCS) moment data — a sample mean vector and
  covariance matrix — and need a NumPy-only solver for the covariance-restricted L1-regularized
  regression objective.
when_to_use_negative:
- The data is a true longitudinal time series with linkage across time points, or
  the goal is a network maintaining a single cell state — COSLIR targets sparse GRNs
  driving cell-state transitions from TSCS data without pseudo-time inference.
- Only single-time-point cross-sectional data is available with no consecutive time-point
  pair, since the method requires first and second moments of two consecutive time
  points.
- Pseudo-time trajectory information is already available and a trajectory-dependent
  GRN inference method is preferred — COSLIR's design bypasses pseudo-time entirely.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: COSLIR (Python version)
  role: Target implementation environment the ADMM solver module must be compatible
    with; provides the model being solved
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Only basic dependency used to implement the closed-form ADMM subproblem updates
    and generate synthetic moment inputs
  repo: github.com/Ge-lab-pku/COSLIR
- name: Python 3.6.4
  role: Language runtime in which the admm_solver.py module is written and tested
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:8e69e3ebcea8a1f8d0cc1a6bec1c75632c306d90fd4c90f3c6b22d9b8c07acb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (Python version)
  - NumPy
  - Python 3.6.4
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# admm-optimization-solver-implementation

## Summary

Implements the ADMM solver that COSLIR uses to solve its non-convex, L1-regularized covariance-restricted sparse linear regression objective (Eq. 3), estimating the sparse regulatory matrix A_t from only the first and second moments of two consecutive time points.

## When to use

Use when you must estimate a sparse, signed, weighted regulatory matrix from time-stamped cross-sectional (TSCS) moment data — a sample mean vector and covariance matrix — and need a NumPy-only solver for the covariance-restricted L1-regularized regression objective.

## When NOT to use

- The data is a true longitudinal time series with linkage across time points, or the goal is a network maintaining a single cell state — COSLIR targets sparse GRNs driving cell-state transitions from TSCS data without pseudo-time inference.
- Only single-time-point cross-sectional data is available with no consecutive time-point pair, since the method requires first and second moments of two consecutive time points.
- Pseudo-time trajectory information is already available and a trajectory-dependent GRN inference method is preferred — COSLIR's design bypasses pseudo-time entirely.

## Inputs

- Sample mean vector (first moment) of gene expression at time point t
- Sample covariance matrix (second moment) of gene expression at time point t
- Sample mean vector and covariance matrix at the next consecutive time point
- ADMM hyperparameters: sparsity (soft-thresholding) parameter and convergence tolerance
- Initial values for primal variables A, B, C and auxiliary/dual variables

## Outputs

- Sparse regulatory matrix A_t with signed, weighted entries (directed GRN edges)
- Primal and dual residual trajectories across ADMM iterations (convergence record)
- Reusable Python module admm_solver.py compatible with the COSLIR Python version

## How to apply

Reformulate the COSLIR objective by introducing auxiliary variables B and C with the constraints A+I=(B+C)/2 and B=C, splitting the non-convex problem into single-variable subproblems. Initialize the primal variables A, B, C and the ADMM auxiliary/dual variables. Each iteration solves the closed-form updates separately: least-squares-type updates for the regression blocks and a shrinkage/soft-thresholding update enforcing sparsity — because each single-variable subproblem objective is quadratic (convex), only linear equations need be solved. Iterate until the primal and dual residuals fall below the convergence tolerance. Wrap the iteration in a reusable Python module (admm_solver.py) whose only basic dependency is NumPy, per the COSLIR Python version's requirements, and run it on a synthetic mean/covariance pair while recording the residual trajectory to confirm convergence.

## Related tools

- **COSLIR (Python version)** (Target implementation environment the ADMM solver module must be compatible with; provides the model being solved) — github.com/Ge-lab-pku/COSLIR
- **NumPy** (Only basic dependency used to implement the closed-form ADMM subproblem updates and generate synthetic moment inputs) — github.com/Ge-lab-pku/COSLIR
- **Python 3.6.4** (Language runtime in which the admm_solver.py module is written and tested)

## Examples

```
python admm_solver.py --mean1 mu_t.npy --cov1 Sigma_t.npy --mean2 mu_t1.npy --cov2 Sigma_t1.npy --tol 1e-4
```

## Evaluation signals

- Primal and dual residual trajectories decrease monotonically and fall below the convergence tolerance
- Recovered matrix A_t reproduces the known regulatory structure in the synthetic oracle case (COSLIR achieved perfect accuracy in simulations in the oracle case)
- A+I=(B+C)/2 and B=C constraints hold (within tolerance) at convergence of the ADMM iterations
- Solver module runs with only NumPy as a basic dependency (tested on v1.14) and no other imports
- Inferred A_t is sparse — soft-thresholding drives most off-diagonal entries to zero, consistent with the assumption that GRNs driving cell-state transitions are sparse

## Limitations

- This is a beta version of COSLIR — some bugs may still remain, so it should be used with caution
- The underlying optimization problem is non-convex; the ADMM solver yields closed-form updates that converge to a reasonable estimate of A_t, not a guaranteed global optimum
- Only time-stamped cross-sectional (TSCS) data can be used — single-cell assays sacrifice individual cells, so true longitudinal linkage across time points is unavailable and the method relies solely on first and second moments

## Evidence

- [results] ADMM splitting with B and C constraints: [evidence span withheld — anchor status: not_found]
- [results] Quadratic subproblems give closed-form updates: [evidence span withheld — anchor status: not_found]
- [abstract] Moments-only input requirement: "utilizing only the first and second moments of samples between two consecutive time points"
- [methods] NumPy-only dependency: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)"
- [readme] Beta-version caveat: "Note that this is a beta version of COSLIR. Some bugs may still remain, so please use COSLIR with caution at present."
