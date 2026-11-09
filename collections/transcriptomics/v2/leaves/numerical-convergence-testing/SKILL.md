---
name: numerical-convergence-testing
description: Use when solving the COSLIR Eq. 3 objective from only the first and second
  moments (sample mean vectors and covariance matrices) of two consecutive TSCS time
  points, and you must demonstrate numerically that the ADMM iteration converged before
  trusting the sparse A_t estimate.
when_to_use_negative:
- Input is true longitudinal single-cell data where the same cells are linked across
  time points — COSLIR is designed for time-stamped cross-sectional (TSCS) data with
  no linkage across time, so moment-based inference is unnecessary.
- The goal is to infer the dense GRN that maintains a single cell state at one time
  point — COSLIR targets the sparse GRN driving cell state transitions between consecutive
  time points, not single-stage networks.
- Per-cell expression profiles or a constructed pseudo-time trajectory are available
  and preferred — the ADMM moment-based solver intentionally bypasses single-cell
  temporal ordering and cannot exploit per-cell linkage information.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: COSLIR (Python version)
  role: Hosts the ADMM solver for Eq. 3; the convergence-tested module is written
    to be compatible with it
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Only basic dependency; used to build the synthetic mean/covariance pair and
    implement the closed-form ADMM updates
- name: Python 3.6.4
  role: Tested runtime for the NumPy-based COSLIR solver and convergence test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:8ac55e3de7ac6ea95fda8db7f0a67856429133197f8953f2336bffd9da297c51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-lab-pku/COSLIR
  tools:
  - COSLIR (Python version)
  - NumPy
  - Python 3.6.4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# numerical-convergence-testing

## Summary

Verifies that the ADMM solver for COSLIR's non-convex, L1-regularized covariance-restricted sparse linear regression (Eq. 3) converges by monitoring primal and dual residuals across the closed-form A, B, C subproblem updates, yielding a defensible estimate of the sparse regulatory matrix A_t.

## When to use

Use when solving the COSLIR Eq. 3 objective from only the first and second moments (sample mean vectors and covariance matrices) of two consecutive TSCS time points, and you must demonstrate numerically that the ADMM iteration converged before trusting the sparse A_t estimate.

## When NOT to use

- Input is true longitudinal single-cell data where the same cells are linked across time points — COSLIR is designed for time-stamped cross-sectional (TSCS) data with no linkage across time, so moment-based inference is unnecessary.
- The goal is to infer the dense GRN that maintains a single cell state at one time point — COSLIR targets the sparse GRN driving cell state transitions between consecutive time points, not single-stage networks.
- Per-cell expression profiles or a constructed pseudo-time trajectory are available and preferred — the ADMM moment-based solver intentionally bypasses single-cell temporal ordering and cannot exploit per-cell linkage information.

## Inputs

- Sample mean vector at time point t (first moment of TSCS samples)
- Sample covariance matrix at time point t (second moment of TSCS samples)
- Sample mean vector and covariance matrix at the consecutive time point t+1
- ADMM hyperparameters: sparsity (L1) penalty and convergence tolerance
- Initialized primal variables A, B, C and auxiliary/dual variables

## Outputs

- Estimated sparse regulatory matrix A_t (directed, signed, weighted edges)
- Primal residual trajectory per ADMM iteration
- Dual residual trajectory per ADMM iteration
- Reusable solver module (e.g. admm_solver.py) dependent only on NumPy

## How to apply

Generate a synthetic time-stamped cross-sectional moment pair (sample mean vector and covariance matrix) with NumPy as solver input, then initialize the primal variables A, B, C and the auxiliary/dual variables of the ADMM splitting with constraints A+I=(B+C)/2 and B=C. Iterate the closed-form subproblem updates separately: least-squares-type quadratic updates for the regression blocks and a shrinkage/soft-thresholding update enforcing L1 sparsity; because each single-variable subproblem is quadratic (convex), only linear equations are solved. Monitor primal and dual residuals each iteration and stop when both fall below the convergence tolerance, recording the residual trajectory as the convergence artifact. Wrap the iteration in a NumPy-only module (e.g. admm_solver.py, tested on NumPy v1.14 / Python 3.6.4) so it can be rerun inside COSLIR's bootstrapping workflow, which solves the model multiple times after bootstrapping and clips the rescaled value and confidence. Convergence is judged correct when the residual trajectory decays monotonically to tolerance and the recovered A_t reproduces the known sparse support in the synthetic oracle setting.

## Related tools

- **COSLIR (Python version)** (Hosts the ADMM solver for Eq. 3; the convergence-tested module is written to be compatible with it) — https://github.com/Ge-lab-pku/COSLIR
- **NumPy** (Only basic dependency; used to build the synthetic mean/covariance pair and implement the closed-form ADMM updates)
- **Python 3.6.4** (Tested runtime for the NumPy-based COSLIR solver and convergence test)

## Evaluation signals

- Primal residual trajectory decays and falls below the chosen convergence tolerance before termination
- Dual residual trajectory also falls below the same tolerance (both residuals, not just one, must satisfy the criterion)
- Each subproblem update is closed-form (linear-system solve / soft-thresholding) — no iterative inner optimizer or line search should appear in the residual log
- In the synthetic oracle case, the recovered A_t matches the known sparse regulatory matrix (perfect accuracy is the reported oracle benchmark)
- Final admm_solver.py imports only NumPy (tested on v1.14) and runs under the COSLIR Python version without other dependencies

## Limitations

- COSLIR is a beta version; some bugs may still remain, so results should be used with caution and bug reports welcomed
- Convergence of each ADMM subproblem relies on the quadratic (convex) single-variable reformulation — the full Eq. 3 objective itself remains non-convex, so the solver converges to a reasonable estimate of A_t, not a global optimum
- Method assumes only the first and second moments of samples between two consecutive time points are available and that the target GRN is sparse; performance on real data is validated only for single-cell RT-PCR and RNA-seq developmental datasets

## Evidence

- [other] ADMM splitting and closed-form convexity rationale: [evidence span withheld — anchor status: not_found]
- [other] Convergence criterion: [evidence span withheld — anchor status: not_found]
- [readme] NumPy-only runtime requirement: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
- [readme] Bootstrapping workflow the converged solver feeds into: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [abstract] Moment-only input assumption: "utilizing only the first and second moments of samples between two consecutive time points"
