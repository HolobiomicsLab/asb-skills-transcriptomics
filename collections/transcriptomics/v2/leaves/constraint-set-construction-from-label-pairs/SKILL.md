---
name: constraint-set-construction-from-label-pairs
description: Use when you have a labeled expression matrix (e.g., the scGeneFit spectral
  toy dataset) and need to translate pairwise label relationships into linear constraints
  for a marker-selection LP solved with scipy.optimize.linprog.
when_to_use_negative:
- The data are unlabeled or lack a categorical cell-type/label partition — scGeneFit's
  constraint construction requires given labels; placing unlabeled cells on trajectories
  or in spatial assays is not supported (only envisioned future work)
- The marker problem needs nonlinear projection or nonlinear dimension reduction —
  the formulation assumes linear separability and each subspace dimension aligned
  with a coordinate axis, which the authors plan to relax only in future work
- Each gene is being tested separately in a one-vs-all manner (e.g., as with COMET)
  — this skill instead jointly recovers an optimal set of genes via shared constraints
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGeneFit
  role: Source of the label-aware constraint formulation Z(D) that turns marker selection
    into a linear program; the toy datasets used to exercise the solver ship with
    its release
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: scipy.optimize.linprog solves the assembled LP with the Z(D) constraints,
    sparsity budget ||alpha||_1 <= s, and slack vector beta
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:d242866f12a10b90990adbac7500396ba787df7be9ce0b7bd3977c93910d69d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# constraint-set-construction-from-label-pairs

## Summary

Builds the pairwise constraint set Z(D) from the cell/sample label vector for scGeneFit's slack linear program (Eq. 4), forming the vectors z whose squared components z_j^2 enter the linear constraints. This encodes which cell labels must be discriminated so marker selection can be cast as a single tractable LP.

## When to use

Use when you have a labeled expression matrix (e.g., the scGeneFit spectral toy dataset) and need to translate pairwise label relationships into linear constraints for a marker-selection LP solved with scipy.optimize.linprog.

## When NOT to use

- The data are unlabeled or lack a categorical cell-type/label partition — scGeneFit's constraint construction requires given labels; placing unlabeled cells on trajectories or in spatial assays is not supported (only envisioned future work)
- The marker problem needs nonlinear projection or nonlinear dimension reduction — the formulation assumes linear separability and each subspace dimension aligned with a coordinate axis, which the authors plan to relax only in future work
- Each gene is being tested separately in a one-vs-all manner (e.g., as with COMET) — this skill instead jointly recovers an optimal set of genes via shared constraints

## Inputs

- Labeled cell/sample expression matrix (e.g., the spectral toy dataset shipped with the scGeneFit release), as a numpy array or pandas DataFrame
- Cell/sample label vector encoding the partition of labels (optionally hierarchical) to discriminate

## Outputs

- Constraint set Z(D): vectors z derived from pairwise label relationships, with squared components z_j^2 used in LP constraints
- Constraint rows sum_j alpha_j z_j^2 >= Delta^2 - beta_z plus sparsity rows ||alpha||_1 <= s (with auxiliary sign variables), passed to scipy.optimize.linprog
- Fitted alpha weight vector whose nonzero indices define the selected marker gene set

## How to apply

Load the labeled toy expression matrix and its label vector with numpy/pandas, then enumerate the pairwise label relationships and build the constraint set Z(D): each constraint vector z encodes a pair of labels that must be separated. For each z in Z(D), the squared components enter constraints of the form sum_j alpha_j z_j^2 >= Delta^2 - beta_z, where beta_z is a slack variable and Delta is the discrimination margin. The scGeneFit formulation constrains the label-aware projection so each subspace dimension is aligned with a coordinate axis in the original gene space, which makes each marker a single gene and renders the overall problem a linear program. The resulting LP is solved with scipy.optimize.linprog together with the sparsity budget ||alpha||_1 <= s (implemented via auxiliary sign variables) and the slack vector beta; nonnegative/above-threshold alpha entries identify the selected marker genes.

## Related tools

- **scGeneFit** (Source of the label-aware constraint formulation Z(D) that turns marker selection into a linear program; the toy datasets used to exercise the solver ship with its release) — github.com/solevillar/scGeneFit
- **SciPy** (scipy.optimize.linprog solves the assembled LP with the Z(D) constraints, sparsity budget ||alpha||_1 <= s, and slack vector beta)

## Examples

```
from scipy.optimize import linprog; res = linprog(c, A_ub=A_from_Z, b_ub=b_from_Z, bounds=alpha_bounds, method='highs')
```

## Evaluation signals

- The emitted LP solution satisfies the feasibility budget ||alpha||_1 <= s
- All constraints from Z(D) hold: sum_j alpha_j z_j^2 + beta_z >= Delta^2 for each z in Z(D)
- Selected markers are single genes (coordinate-axis aligned alpha), i.e., each nonzero weight corresponds to exactly one gene index
- On the spectral toy dataset, the two circle-coordinate features are selected whenever the target marker set size is >= 2, and the selected markers recover the toy labels
- Selected markers are fewer and less redundant than one-vs-all baselines (e.g., COMET) at equal label-recovery performance

## Limitations

- Relies on the assumption that cell type classes are linearly separable, which is reasonable in high dimensions but may fail in lower dimensions
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is planned future work
- Marker selection is limited to categorical labeling; no support for placing unlabeled cells on trajectories or spatial locations (a manifold constraint is envisioned for the future)

## Evidence

- [methods] coordinate-axis constraint yielding LP: [evidence span withheld — anchor status: not_found]
- [methods] LP structure (Eq. 4): [evidence span withheld — anchor status: not_found]
- [intro] joint marker recovery motivation: "Where traditional approaches test the discriminatory value of each gene separately, our approach jointly recovers the optimal set of genes of a given size that allows robust partitioning of the given"
- [results] toy validation expectation: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] linear separability caveat: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
