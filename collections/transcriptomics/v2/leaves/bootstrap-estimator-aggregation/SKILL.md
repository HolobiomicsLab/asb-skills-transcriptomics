---
name: bootstrap-estimator-aggregation
description: Use when you have bootstrapped COSLIR regression coefficient matrices
  plus first-moment vectors μ̂_1 and μ̂_2 from two consecutive time points and must
  rank which interactions drive the cell-state transition.
when_to_use_negative:
- Data consist of a single cell stage/time point — COSLIR's rescaled value requires
  two consecutive-stage means μ̂_1 and μ̂_2, and such data describe networks maintaining
  the current state rather than driving transitions
- Raw single-cell expression counts without computed first and second moments — COSLIR
  relies exclusively on first and second moments of measured samples, not on pseudo-time-ordered
  trajectories
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: COSLIR (Python version)
  role: Produces the bootstrapped regression coefficient matrix A and moment vectors
    mu1/mu2 that feed the rescaled-value computation
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Computes the elementwise rescaled values |A|_ij * |mu1|_j / |(mu1)_i - (mu2)_i|
    and aggregates them across bootstrap replicates
- name: COSLIR (MATLAB version)
  role: Alternative implementation for generating the bootstrapped estimator before
    rescaled-value ranking
  repo: github.com/Ge-lab-pku/COSLIR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:85120d7ec50eb45baa0f90735ae9b3db524203cf4698e9fb6fa6a38ddcf455e6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bootstrap-estimator-aggregation

## Summary

Aggregates COSLIR regression coefficients across bootstrap replicates by computing each gene pair's rescaled value (Ã)_ij = (|Â|)_ij · |μ̂_1|_j / |(μ̂_1)_i − (μ̂_2)_i|, then ranking directed regulatory interactions for downstream early-precision evaluation.

## When to use

Use when you have bootstrapped COSLIR regression coefficient matrices plus first-moment vectors μ̂_1 and μ̂_2 from two consecutive time points and must rank which interactions drive the cell-state transition.

## When NOT to use

- Data consist of a single cell stage/time point — COSLIR's rescaled value requires two consecutive-stage means μ̂_1 and μ̂_2, and such data describe networks maintaining the current state rather than driving transitions
- Raw single-cell expression counts without computed first and second moments — COSLIR relies exclusively on first and second moments of measured samples, not on pseudo-time-ordered trajectories

## Inputs

- Bootstrapped COSLIR regression coefficient matrix A (one matrix per replicate)
- First-moment vector mu1 from the earlier time point
- First-moment vector mu2 from the later consecutive time point
- Single-cell TSCS-derived first and second moments (qPCR or RNA-seq data from the COSLIR repo)

## Outputs

- ranked_interactions.csv with one row per non-zero bootstrapped interaction, containing regulator, target, and aggregated rescaled-value columns
- Ranked table of the most influential regulatory interactions for early-precision evaluation

## How to apply

Load the bootstrapped COSLIR coefficient matrix A and the mean vectors mu1 (earlier stage) and mu2 (later stage) with Python/NumPy. For every candidate edge (i, j), compute the rescaled value |A|_ij * |mu1|_j / |(mu1)_i - (mu2)_i|, re-weighting each absolute regression coefficient by the target gene's mean abundance and the regulator's absolute mean change across the transition. Aggregate the rescaled values across bootstrap replicates into a single score per directed interaction, following the COSLIR workflow of solving the model repeatedly after bootstrapping and then clipping on the rescaled value and the confidence. Sort all candidate interactions by aggregated rescaled value in descending order and write regulator, target, and rescaled-value columns to ranked_interactions.csv.

## Related tools

- **COSLIR (Python version)** (Produces the bootstrapped regression coefficient matrix A and moment vectors mu1/mu2 that feed the rescaled-value computation) — github.com/Ge-lab-pku/COSLIR
- **NumPy** (Computes the elementwise rescaled values |A|_ij * |mu1|_j / |(mu1)_i - (mu2)_i| and aggregates them across bootstrap replicates)
- **COSLIR (MATLAB version)** (Alternative implementation for generating the bootstrapped estimator before rescaled-value ranking) — github.com/Ge-lab-pku/COSLIR

## Examples

```
rescaled = np.abs(A) * np.abs(mu1) / np.abs(mu1 - mu2)[:, None]
```

## Evaluation signals

- ranked_interactions.csv exists and parses cleanly with NumPy/pandas
- Table contains one row per non-zero bootstrapped interaction and every rescaled value is finite (no NaN/Inf, i.e. |(mu1)_i - (mu2)_i| > 0 for retained rows)
- Top-ranked rows correspond to the highest aggregated rescaled scores available for downstream early-precision evaluation
- Rescaled values match the formula |A|_ij * |mu1|_j / |(mu1)_i - (mu2)_i| when recomputed independently from the saved estimator outputs

## Limitations

- COSLIR is a beta version; some bugs may still remain and users are cautioned to use it carefully
- Non-finite rescaled values can arise for genes whose mean is unchanged between the two consecutive stages ((μ̂_1)_i ≈ (μ̂_2)_i), so a clipping step on the rescaled value and the confidence is applied
- COSLIR was tested on Python v3.6.4 with NumPy v1.14 (Windows) and MATLAB 9.4 (R2018a); other environments are unverified

## Evidence

- [readme] The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence.: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [abstract] utilizing only the first and second moments of samples between two consecutive time points: "utilizing only the first and second moments of samples between two consecutive time points"
- [intro] GRNs that drive cell state transitions are considered sparse: "GRNs that drive cell state transitions are considered sparse"
