---
name: linear-programming-formulation
description: Use when selecting a small, non-redundant gene-marker panel from labeled
  scRNA-seq data and the combinatorial marker problem must be relaxed to a solvable
  LP with a sparsity budget ||alpha||_1 <= s.
when_to_use_negative:
- The analysis requires nonlinear dimension reduction or the cell type classes are
  not linearly separable, since scGeneFit assumes linear separability of labels
- The goal is placing unlabeled cells on trajectories or spatial locations rather
  than categorical label discrimination — the method is limited to categorical labeling
- Each gene must be tested separately for discriminatory value rather than jointly
  recovering a set of genes of a given size
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Provides the label-aware compressive classification formulation whose axis-aligned
    constraint yields the sparse linear program (Eq. 4) for marker selection
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: scipy.optimize.linprog solves the assembled slack LP (constraints sum_j alpha_j
    z_j^2 >= Delta^2 - beta_z, budget ||alpha||_1 <= s, slack vector beta)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:dd5d3badf262db1c3eae14522e7f85ca3cd4198a50b7b37e3c43db997c30183e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# linear-programming-formulation

## Summary

Reformulates scGeneFit's combinatorial marker-selection objective as a slack linear program (Eq. 4) by constraining each projection dimension to align with a gene coordinate axis, so the optimization becomes tractable and each marker is a single gene solved via scipy.optimize.linprog.

## When to use

Use when selecting a small, non-redundant gene-marker panel from labeled scRNA-seq data and the combinatorial marker problem must be relaxed to a solvable LP with a sparsity budget ||alpha||_1 <= s.

## When NOT to use

- The analysis requires nonlinear dimension reduction or the cell type classes are not linearly separable, since scGeneFit assumes linear separability of labels
- The goal is placing unlabeled cells on trajectories or spatial locations rather than categorical label discrimination — the method is limited to categorical labeling
- Each gene must be tested separately for discriminatory value rather than jointly recovering a set of genes of a given size

## Inputs

- Small labeled scRNA-seq expression matrix (cells x genes) with an explicit label vector, e.g. the spectral toy dataset from the scGeneFit release
- Label hierarchy / pairwise label relationships used to encode the constraint set Z(D)
- Target marker set size s (sparsity budget) and margin parameter Delta

## Outputs

- Fitted alpha weight vector from the LP solution
- Selected marker indices (alpha entries nonzero/above threshold) written to CSV with explicit column names
- Alpha weight vector written to CSV with explicit column names

## How to apply

Load the labeled expression matrix (e.g., the spectral toy dataset shipped at github.com/solevillar/scGeneFit) with numpy/pandas and build the constraint set Z(D) from pairwise label relationships, forming vectors z whose squared components enter the constraints. Assemble Eq. 4 as a linear program: for each z in Z(D), constrain sum_j alpha_j z_j^2 >= Delta^2 - beta_z, impose the sparsity budget ||alpha||_1 <= s via auxiliary sign variables, and include the slack vector beta to keep the program feasible. Solve with scipy.optimize.linprog, then derive the selected marker set as the indices whose alpha weights are nonzero (above threshold). Rationale: constraining the label-aware projection so each subspace dimension is aligned with a coordinate axis in the original gene space makes each marker a single gene rather than a weighted combination, which is what renders the optimization a linear program rather than a combinatorially difficult search.

## Related tools

- **scGeneFit** (Provides the label-aware compressive classification formulation whose axis-aligned constraint yields the sparse linear program (Eq. 4) for marker selection) — github.com/solevillar/scGeneFit
- **SciPy** (scipy.optimize.linprog solves the assembled slack LP (constraints sum_j alpha_j z_j^2 >= Delta^2 - beta_z, budget ||alpha||_1 <= s, slack vector beta))

## Examples

```
from scipy.optimize import linprog; res = linprog(c=beta_costs, A_ub=Z2_constraints, b_ub=margin_rhs, bounds=[(0,1)]*n_genes, method='highs')
```

## Evaluation signals

- The emitted LP solution satisfies the feasibility budget ||alpha||_1 <= s
- The selected markers recover the toy labels (in the spectral toy model the two circle-coordinate features should be selected whenever the target marker set size is >= 2)
- Each selected marker corresponds to exactly one gene index (alpha support is axis-aligned), not a weighted combination of genes
- The LP converges to a feasible solution with the slack vector beta absorbing any violated margin constraints sum_j alpha_j z_j^2 >= Delta^2 - beta_z
- Output CSVs contain the alpha weight vector and marker indices with explicit column names

## Limitations

- A key underlying assumption is that cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; the linear separability assumption is planned future work to relax
- Marker selection is limited to categorical labeling; no support for placing unlabeled cells on trajectories or in spatial assays (a future manifold constraint is envisioned)
- Optimal marker choice given a hierarchical partition of labels is combinatorially difficult, so the LP is a tractable relaxation rather than the exact combinatorial optimum

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [intro] we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels.: "we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels."
- [results] scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
