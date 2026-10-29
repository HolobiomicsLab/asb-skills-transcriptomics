---
name: sparsity-constrained-optimization-with-slack-variables
description: Use when you need to select exactly s individual genes as markers from
  labeled scRNA-seq data by solving a linear program with L1 sparsity and slack constraints,
  rather than weighted gene combinations.
when_to_use_negative:
- The goal is a nonlinear or weighted combination of genes per marker — scGeneFit
  deliberately restricts each marker to a single coordinate axis in gene space
- Cell labels are unavailable or continuous (e.g., trajectory/pseudotime or spatial
  positions); the discussion notes categorical labeling and linear separability are
  assumed, with manifold constraints left as future work
- The cell-type classes are not linearly separable in the reduced space — the paper
  states this assumption 'is a reasonable assumption in high dimensions, but may not'
  hold in lower dimensions
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Defines the label-aware compressive-classification LP formulation (Eq. 4)
    with axis-aligned markers, L1 sparsity budget, and slack variables; its release
    supplies the toy spectral dataset and label structure
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: scipy.optimize.linprog solves the assembled slack linear program with sign-variable-encoded
    L1 budget to obtain the alpha weight vector
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:42f92db90aea2a879ed39dff837c1efa56e9b098c4d0997628ed508592d4a887
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# sparsity-constrained-optimization-with-slack-variables

## Summary

Reconstructs scGeneFit's sparse linear-program formulation (slack LP, Eq. 4) in SciPy, in which label-aware projection weights alpha are constrained per-coordinate (one marker = one gene) under an L1 sparsity budget ||alpha||_1 <= s with slack variables beta absorbing constraint violations.

## When to use

Use when you need to select exactly s individual genes as markers from labeled scRNA-seq data by solving a linear program with L1 sparsity and slack constraints, rather than weighted gene combinations.

## When NOT to use

- The goal is a nonlinear or weighted combination of genes per marker — scGeneFit deliberately restricts each marker to a single coordinate axis in gene space
- Cell labels are unavailable or continuous (e.g., trajectory/pseudotime or spatial positions); the discussion notes categorical labeling and linear separability are assumed, with manifold constraints left as future work
- The cell-type classes are not linearly separable in the reduced space — the paper states this assumption 'is a reasonable assumption in high dimensions, but may not' hold in lower dimensions

## Inputs

- Small labeled gene expression matrix (e.g., scGeneFit spectral toy dataset from github.com/solevillar/scGeneFit) as numpy array or pandas DataFrame
- Vector of cell/sample labels encoding pairwise label relationships used to build Z(D)
- Target marker set size s (sparsity budget) and separation margin Delta^2

## Outputs

- Fitted alpha weight vector from the solved LP
- Selected marker gene indices (alpha entries above threshold / nonzero)
- CSV files with explicit column names containing the alpha weights and selected marker indices

## How to apply

Load a small labeled expression matrix (e.g., the spectral toy dataset shipped with github.com/solevillar/scGeneFit) plus its cell labels with numpy/pandas. Build the constraint set Z(D) from the pairwise label relationships encoded in the label vector, forming the vectors z whose squared components z_j^2 enter the linear constraints sum_j alpha_j z_j^2 >= Delta^2 - beta_z. Assemble the LP for Eq. 4 with auxiliary sign variables to implement the sparsity budget ||alpha_1|| <= s and a slack vector beta, and solve with scipy.optimize.linprog. Because scGeneFit constrains each subspace dimension to align with a single coordinate axis in gene space, the optimization is a linear program and each nonzero alpha index maps to exactly one marker gene; extract markers as the above-threshold alpha indices and write alpha and marker indices to CSV with explicit column names.

## Related tools

- **scGeneFit** (Defines the label-aware compressive-classification LP formulation (Eq. 4) with axis-aligned markers, L1 sparsity budget, and slack variables; its release supplies the toy spectral dataset and label structure) — github.com/solevillar/scGeneFit
- **SciPy** (scipy.optimize.linprog solves the assembled slack linear program with sign-variable-encoded L1 budget to obtain the alpha weight vector)

## Examples

```
from scipy.optimize import linprog; res = linprog(c=beta_cost, A_ub=-Zsq, b_ub=-delta2, bounds=bounds_alpha_sign, method='highs')
```

## Evaluation signals

- The LP solution must satisfy the feasibility budget ||alpha||_1 <= s, including contributions from the auxiliary sign variables
- Marker indices extracted from above-threshold alpha weights must each correspond to a single coordinate axis (gene), not a weighted combination
- On the spectral toy dataset, the selected markers must recover the toy sample labels; specifically the two circle-coordinate features should be selected whenever the target marker set size is >= 2
- All LP constraints sum_j alpha_j z_j^2 >= Delta^2 - beta_z must hold for every z in Z(D), with slack beta_z absorbing only allowed violations
- Emitted CSV files must contain the alpha weight vector and marker indices with explicit column names

## Limitations

- A key underlying assumption is linear separability of cell type classes, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is planned future work
- Marker selection is limited to categorical labeling; support for placing unlabeled cells on trajectories or spatial locations via a manifold constraint is only envisioned future work

## Evidence

- [abstract] axis-aligned sparsity renders Eq. 4 a linear program: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] toy spectral validation recovers circle coordinates: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [intro] motivation for jointly recovering an optimal gene set: "Where traditional approaches test the discriminatory value of each gene separately, our approach jointly recovers the optimal set of genes of a given size that allows robust partitioning of the given"
- [discussion] linear separability caveat from discussion: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
