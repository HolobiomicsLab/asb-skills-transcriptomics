---
name: toy-dataset-validation-of-optimization-solver
description: Use when implementing or debugging a marker-selection optimization (e.g.,
  scGeneFit's slack LP from Eq. 4) and you need a minimal labeled toy expression matrix
  plus a concrete expected marker output to verify solver correctness.
when_to_use_negative:
- The goal is validating scGeneFit on real scRNA-seq datasets (e.g., CBMC or mouse
  somatosensory cortex) rather than verifying the solver logic on a small controlled
  toy model
- Cell type classes are not linearly separable or the task requires nonlinear dimension
  reduction — the paper notes this linear separability assumption underlies the formulation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGeneFit
  role: Source of the spectral toy dataset and the Eq. 4 slack-LP formulation (coordinate-axis-aligned
    projection, sparsity budget ||alpha||_1 <= s) being validated
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: scipy.optimize.linprog solves the assembled linear program with the label-aware
    constraints, auxiliary sign variables, and slack vector beta
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:e141bdb09952a1334fbf5a77b6643d46ff691a83e60822a49b5299a4bedf6521
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# toy-dataset-validation-of-optimization-solver

## Summary

A validation harness that reconstructs scGeneFit's sparse linear-program (slack LP, Eq. 4) marker solver with scipy.optimize.linprog and tests it on the small labeled spectral toy dataset, where the two circle-coordinate features must be selected whenever the target marker set size is >=2.

## When to use

Use when implementing or debugging a marker-selection optimization (e.g., scGeneFit's slack LP from Eq. 4) and you need a minimal labeled toy expression matrix plus a concrete expected marker output to verify solver correctness.

## When NOT to use

- The goal is validating scGeneFit on real scRNA-seq datasets (e.g., CBMC or mouse somatosensory cortex) rather than verifying the solver logic on a small controlled toy model
- Cell type classes are not linearly separable or the task requires nonlinear dimension reduction — the paper notes this linear separability assumption underlies the formulation

## Inputs

- Small labeled toy expression matrix (numpy array or pandas DataFrame), e.g., the spectral toy dataset shipped with the scGeneFit release
- Vector of cell/sample labels for the toy dataset
- Target marker set size s (sparsity budget) and separation parameter Delta^2 for the Eq. 4 constraints

## Outputs

- Fitted alpha weight vector from the solved linear program (CSV with explicit column names)
- Selected marker indices (indices with nonzero, above-threshold alpha weights; CSV with explicit column names)

## How to apply

Load the small labeled toy expression matrix (the spectral toy dataset shipped with the scGeneFit release at github.com/solevillar/scGeneFit) with numpy/pandas, together with its cell/sample labels. Build the constraint set Z(D) from the pairwise label relationships encoded in the label vector, forming the vectors z whose squared components enter the linear constraints. Assemble the linear program for Eq. 4 — constraints sum_j alpha_j z_j^2 >= Delta^2 - beta_z per z in Z(D), the sparsity budget ||alpha||_1 <= s implemented via auxiliary sign variables, and the slack vector beta — and solve it with scipy.optimize.linprog. Extract alpha, derive the selected marker set as indices with nonzero (above-threshold) alpha weights, and write alpha and the marker indices to CSV files with explicit column names. The rationale from the paper: scGeneFit constrains each subspace dimension to align with a coordinate axis in gene space, so each marker is a single gene rather than a weighted combination, rendering the optimization a linear program that the toy LP can faithfully reproduce.

## Related tools

- **scGeneFit** (Source of the spectral toy dataset and the Eq. 4 slack-LP formulation (coordinate-axis-aligned projection, sparsity budget ||alpha||_1 <= s) being validated) — github.com/solevillar/scGeneFit
- **SciPy** (scipy.optimize.linprog solves the assembled linear program with the label-aware constraints, auxiliary sign variables, and slack vector beta)

## Examples

```
from scipy.optimize import linprog; res = linprog(c=alpha_costs, A_ub=constraint_matrix, b_ub=beta_rhs, bounds=bounds_with_sign_vars, method='highs')
```

## Evaluation signals

- The emitted solution satisfies the feasibility budget ||alpha||_1 <= s
- In the spectral toy model, the two circle-coordinate features are selected whenever the target marker set size is >= 2
- The selected markers recover the toy sample labels
- Each selected marker corresponds to exactly one gene (a single coordinate axis in the original gene space), not a weighted combination
- The alpha weight vector and selected marker indices are written to CSV files with explicit column names

## Limitations

- A key underlying assumption is that cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; future work plans to relax the linear separability assumption
- Marker selection is limited to categorical labeling; no support for placing unlabeled cells on trajectories or in spatial assays

## Evidence

- [results] spectral toy expectation: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [abstract] coordinate-axis formulation: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [intro] joint recovery rationale: "our approach jointly recovers the optimal set of genes of a given size that allows robust partitioning of the given"
- [discussion] linear separability caveat: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
