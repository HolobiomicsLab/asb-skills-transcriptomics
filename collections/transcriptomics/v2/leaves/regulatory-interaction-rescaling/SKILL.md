---
name: regulatory-interaction-rescaling
description: Use when you have bootstrapped COSLIR coefficient matrices plus first-moment
  vectors (mu1, mu2) from two consecutive time points and need to rank directed gene-gregulatory
  interactions by influence.
when_to_use_negative:
- You need GRNs that maintain a single cell stage rather than the mechanism driving
  a cell-state transition — COSLIR targets sparse transition-driving networks from
  TSCS data
- Only a single time point / cell stage is available, so no consecutive-stage means
  (mu1, mu2) exist to compute the denominator |(mu1)_i − (mu2)_i|
- Expression values are not summarized as first and second moments per time point
  (e.g., raw unaggregated counts without time stamps)
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: COSLIR (Python version)
  role: Produces the bootstrapped coefficient matrix A and mean vectors mu1/mu2 that
    the rescaled-value formula consumes; the workflow is implemented in Python with
    NumPy
  repo: https://github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Elementwise computation of |A|_ij * |mu1|_j / |(mu1)_i − (mu2)_i| and sorting/aggregation
    of rescaled scores
- name: COSLIR (MATLAB version)
  role: Alternative implementation that can supply the same bootstrapped coefficient
    matrices and moment vectors for rescaling
  repo: https://github.com/Ge-lab-pku/COSLIR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:b3729e4be6415d95fc867edf18a6be969ab873c381d4213af336f921a2329c92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-lab-pku/COSLIR
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# regulatory-interaction-rescaling

## Summary

Rescales each bootstrapped COSLIR regression coefficient (edge weight) by the target gene's mean abundance and the regulator's absolute mean change across a two-stage transition — (A~)_ij = (|Â|)_ij · |μ̂_1|_j / |(μ̂_1)_i − (μ̂_2)_i| — so influential directed regulatory interactions can be ranked for early-precision evaluation.

## When to use

Use when you have bootstrapped COSLIR coefficient matrices plus first-moment vectors (mu1, mu2) from two consecutive time points and need to rank directed gene-gregulatory interactions by influence.

## When NOT to use

- You need GRNs that maintain a single cell stage rather than the mechanism driving a cell-state transition — COSLIR targets sparse transition-driving networks from TSCS data
- Only a single time point / cell stage is available, so no consecutive-stage means (mu1, mu2) exist to compute the denominator |(mu1)_i − (mu2)_i|
- Expression values are not summarized as first and second moments per time point (e.g., raw unaggregated counts without time stamps)

## Inputs

- Bootstrapped COSLIR regression coefficient matrix A (NumPy array of edge weights)
- First-moment vector mu1 (mean expression at the earlier time point)
- First-moment vector mu2 (mean expression at the later time point)
- Multiple bootstrap replicates of the COSLIR estimator

## Outputs

- ranked_interactions.csv: table of regulator, target, and aggregated rescaled value sorted in descending order
- One row per non-zero bootstrapped directed interaction with finite rescaled values

## How to apply

Load the bootstrapped COSLIR regression coefficient matrix A and the consecutive-stage mean vectors mu1 (earlier) and mu2 (later) using Python with NumPy. For every candidate edge (i,j) compute the rescaled value |A|_ij * |mu1|_j / |(mu1)_i − (mu2)_i|, re-weighting each absolute regression coefficient by the target gene's mean abundance and the regulator's absolute mean change across the transition. Aggregate rescaled values across bootstrap replicates into a single score per directed interaction, consistent with COSLIR's general workflow of solving the model multiple times after bootstrapping and then making a clipping on the rescaled value and the confidence. Sort all candidate interactions by aggregated rescaled value in descending order and write regulator, target, and rescaled-value columns to ranked_interactions.csv for downstream early-precision evaluation.

## Related tools

- **COSLIR (Python version)** (Produces the bootstrapped coefficient matrix A and mean vectors mu1/mu2 that the rescaled-value formula consumes; the workflow is implemented in Python with NumPy) — https://github.com/Ge-lab-pku/COSLIR
- **NumPy** (Elementwise computation of |A|_ij * |mu1|_j / |(mu1)_i − (mu2)_i| and sorting/aggregation of rescaled scores)
- **COSLIR (MATLAB version)** (Alternative implementation that can supply the same bootstrapped coefficient matrices and moment vectors for rescaling) — https://github.com/Ge-lab-pku/COSLIR

## Examples

```
import numpy as np; R = np.abs(A) * np.abs(mu1)[None,:] / np.abs(mu1 - mu2)[:,None]; np.savetxt('ranked_interactions.csv', R, delimiter=',')
```

## Evaluation signals

- ranked_interactions.csv exists and parses cleanly with NumPy/pandas
- The file contains one row per non-zero bootstrapped interaction and every rescaled value is finite (no NaN/Inf; the denominator |(mu1)_i − (mu2)_i| is non-zero for included rows)
- Rows are sorted by aggregated rescaled value in descending order
- Top-ranked rows correspond to the highest rescaled scores available and carry forward to downstream early-precision evaluation
- Spot-check a sampled edge (i,j) by recomputing |A|_ij · |mu1|_j / |(mu1)_i − (mu2)_i| independently and matching the tabulated score

## Limitations

- COSLIR is a beta version and some bugs may still remain, so results should be used with caution (per the README)
- The rescaling denominator requires two consecutive stages with distinct means; edges with |(mu1)_i − (mu2)_i| near zero are ill-conditioned
- The final filtering step involves a clipping on the rescaled value and the confidence whose exact thresholds are detailed in the repo's DESCRIPTION files, not in the README

## Evidence

- [other] COSLIR defines a rescaled value for every gene pair (i,j) ... as (Ã)_ij = (|Â|)_ij · |μ̂_1|_j / |(μ̂_1)_i − (μ̂_2)_i|: [evidence span withheld — anchor status: not_found]
- [readme] The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence.: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [abstract] utilizing only the first and second moments of samples between two consecutive time points: "utilizing only the first and second moments of samples between two consecutive time points"
- [readme] COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14).: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)."
