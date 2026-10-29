---
name: directed-edge-ranking-by-weight
description: Use when you have a bootstrapped COSLIR coefficient matrix A plus the
  two consecutive-stage mean vectors mu1 and mu2, and need to rank directed regulator→target
  edges for early-precision evaluation.
when_to_use_negative:
- Only a single cell stage/time point is available, so the two consecutive-stage means
  mu1 and mu2 required by the rescaled-value formula cannot be computed
- The goal is to characterize the network maintaining a current cell state rather
  than the mechanism driving a transition along the cell lineage
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: COSLIR (Python version)
  role: Produces the bootstrapped regression coefficient matrix A and mean vectors
    mu1/mu2 that the rescaled-value ranking consumes
  repo: github.com/Ge-lab-pku/COSLIR
- name: NumPy
  role: Element-wise computation of |A|_ij * |mu1|_j / |(mu1)_i - (mu2)_i|, aggregation
    across bootstrap replicates, and sorting of the ranked edge table
- name: COSLIR (MATLAB version)
  role: Alternative upstream implementation producing A, mu1, and mu2 for the same
    rescaled-value ranking
  repo: github.com/Ge-lab-pku/COSLIR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-62850-1
    title: Unveiling gene regulatory networks during cellular state transitions without
      linkage across time points
  source_content_hash: sha256:d3c810084decbb0ae3fb02cfe93e2598cfaf1fd2d801ac995b44575284e577c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSLIR (MATLAB version)
  - COSLIR (Python version)
  - NumPy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# directed-edge-ranking-by-weight

## Summary

Rank directed gene regulatory interactions inferred by COSLIR by computing a rescaled value for every gene pair — the absolute bootstrapped regression weight scaled by the target gene's mean abundance and the regulator's absolute mean change between two consecutive time points — so the most influential edges can be selected for downstream early-precision evaluation.

## When to use

Use when you have a bootstrapped COSLIR coefficient matrix A plus the two consecutive-stage mean vectors mu1 and mu2, and need to rank directed regulator→target edges for early-precision evaluation.

## When NOT to use

- Only a single cell stage/time point is available, so the two consecutive-stage means mu1 and mu2 required by the rescaled-value formula cannot be computed
- The goal is to characterize the network maintaining a current cell state rather than the mechanism driving a transition along the cell lineage

## Inputs

- Bootstrapped COSLIR regression coefficient matrix A (p x p, one entry per directed gene pair)
- First-moment vector mu1 (mean gene expression at the earlier time point)
- First-moment vector mu2 (mean gene expression at the later time point)

## Outputs

- ranked_interactions.csv: one row per non-zero bootstrapped directed interaction, with regulator, target, and rescaled-value columns sorted by descending rescaled score

## How to apply

Load the bootstrapped COSLIR regression coefficient matrix A and the first-moment vectors mu1 (earlier time point) and mu2 (later time point) from the upstream COSLIR run using Python with NumPy. For every candidate edge (i, j), compute the rescaled value (Ã)_ij = (|Â|)_ij · |μ̂_1|_j / |(μ̂_1)_i − (μ̂_2)_i| — the absolute estimated interaction weight re-weighted by the target gene's mean abundance divided by the absolute difference between the i-th components of the two consecutive-stage means. Aggregate rescaled values across bootstrap replicates of the COSLIR estimator into a single score per directed interaction, consistent with the README's stated workflow of solving the model multiple times after bootstrapping and then clipping the rescaled value and confidence. Sort all candidate regulatory interactions by aggregated rescaled value in descending order and write regulator, target, and rescaled-value columns to ranked_interactions.csv.

## Related tools

- **COSLIR (Python version)** (Produces the bootstrapped regression coefficient matrix A and mean vectors mu1/mu2 that the rescaled-value ranking consumes) — github.com/Ge-lab-pku/COSLIR
- **NumPy** (Element-wise computation of |A|_ij * |mu1|_j / |(mu1)_i - (mu2)_i|, aggregation across bootstrap replicates, and sorting of the ranked edge table)
- **COSLIR (MATLAB version)** (Alternative upstream implementation producing A, mu1, and mu2 for the same rescaled-value ranking) — github.com/Ge-lab-pku/COSLIR

## Examples

```
import numpy as np; rs = np.abs(A) * np.abs(mu1)[None, :] / np.abs(mu1[:, None] - mu2[:, None]); ranked = np.dstack(np.unravel_index(np.argsort(-rs, axis=None), rs.shape))
```

## Evaluation signals

- ranked_interactions.csv exists and parses cleanly with NumPy/pandas
- Every row carries finite rescaled values (no NaN/inf from division by zero mean change)
- The file contains one row per non-zero bootstrapped interaction, with no duplicate directed edges
- Rows are sorted by aggregated rescaled value in descending order
- Top-ranked rows correspond to the highest rescaled scores available for downstream early-precision evaluation

## Limitations

- COSLIR is explicitly a beta version — 'Some bugs may still remain, so please use COSLIR with caution at present' — so rescaled-value rankings inherit any instability in the upstream estimator
- The rescaled value divides by |(mu1)_i - (mu2)_i|; a regulator whose mean is unchanged across the transition will blow up or be undefined and must be handled before ranking
- COSLIR infers transition-driving GRNs that are assumed sparse, so ranking is meaningful only for that sparse transition mechanism, not for dense single-state networks

## Evidence

- [other] rescaled-value formula definition: [evidence span withheld — anchor status: not_found]
- [readme] README bootstrap workflow: "The general workflow of COSLIR is to solve the model multiple times after bootstrapping and then make a clipping on the rescaled value and the confidence."
- [abstract] moments-only input requirement: "utilizing only the first and second moments of samples between two consecutive time points"
- [methods] runtime environment for implementing the computation: "COSLIR is tested on Python v3.6.4 (Windows), with the only basic requirement of Numpy (tested on v1.14)"
