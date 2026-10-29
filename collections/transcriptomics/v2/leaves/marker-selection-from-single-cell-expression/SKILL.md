---
name: marker-selection-from-single-cell-expression
description: Use when you have scRNA-seq expression data plus cell type labels (optionally
  a label hierarchy) and must choose a minimal gene panel that jointly discriminates
  the labels rather than testing genes one-vs-all.
when_to_use_negative:
- Cells are unlabeled or labels are continuous (e.g., trajectory pseudotime or spatial
  location) — the method only supports categorical labeling
- Cell type classes are expected to be nonlinearly separable in low dimensions — the
  method assumes linear separability, reasonable in high dimensions
- The goal is one-vs-all per-gene differential testing rather than jointly recovering
  an optimal marker set for label recovery
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Marker selection method; its spectral toy dataset supplies the labeled expression
    matrix and its linear-program formulation (slack LP, Eq. 4) defines the optimization
    implemented in this skill
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: scipy.optimize.linprog solves the assembled slack linear program for the alpha
    weight vector
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:99bdcdcb50c11e9c256826646330e3899b733c232bee3bd5c5973054fb8ba013
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# marker-selection-from-single-cell-expression

## Summary

Selects a small, jointly optimized set of gene markers from labeled single-cell RNA-seq data using scGeneFit's label-aware compressive classification, in which the sparse label-aware projection is solved as a slack linear program (Eq. 4) so each marker corresponds to a single gene.

## When to use

Use when you have scRNA-seq expression data plus cell type labels (optionally a label hierarchy) and must choose a minimal gene panel that jointly discriminates the labels rather than testing genes one-vs-all.

## When NOT to use

- Cells are unlabeled or labels are continuous (e.g., trajectory pseudotime or spatial location) — the method only supports categorical labeling
- Cell type classes are expected to be nonlinearly separable in low dimensions — the method assumes linear separability, reasonable in high dimensions
- The goal is one-vs-all per-gene differential testing rather than jointly recovering an optimal marker set for label recovery

## Inputs

- Labeled scRNA-seq cell-by-gene expression matrix (e.g., the spectral toy dataset from github.com/solevillar/scGeneFit)
- Vector of cell/sample labels (optionally a hierarchical partition of cell type labels)
- Target marker set size s (sparsity budget)

## Outputs

- Fitted alpha weight vector (CSV with explicit column names)
- Selected marker gene indices with nonzero alpha weights (CSV)

## How to apply

Load the labeled expression matrix (e.g., the spectral toy dataset shipped at github.com/solevillar/scGeneFit) with numpy/pandas and build the constraint set Z(D) from the pairwise label relationships in the label vector. Assemble the linear program for Eq. 4: constraints sum_j alpha_j z_j^2 >= Delta^2 - beta_z for each z in Z(D), a sparsity budget ||alpha||_1 <= s implemented with auxiliary sign variables, and a slack vector beta, then solve it with scipy.optimize.linprog. Extract the fitted alpha weight vector and take markers as indices with nonzero (above-threshold) alpha weights, writing the alpha vector and selected marker indices to CSV with explicit column names. The formulation is tractable because each subspace dimension is constrained to align with a coordinate axis in gene space, making each marker a single gene and rendering the optimization a linear program rather than a combinatorial search.

## Related tools

- **scGeneFit** (Marker selection method; its spectral toy dataset supplies the labeled expression matrix and its linear-program formulation (slack LP, Eq. 4) defines the optimization implemented in this skill) — github.com/solevillar/scGeneFit
- **SciPy** (scipy.optimize.linprog solves the assembled slack linear program for the alpha weight vector)

## Examples

```
from scipy.optimize import linprog; res = linprog(c, A_ub=A_constraints, b_ub=b_constraints, bounds=bounds, method='highs')
```

## Evaluation signals

- Emitted solution satisfies the feasibility budget ||alpha||_1 <= s
- Selected markers recover the toy labels: in the spectral toy model the two circle-coordinate features are selected whenever the target marker set size is >= 2
- Each selected marker corresponds to a single gene (coordinate-axis-aligned projection), not a weighted combination of genes
- Alpha weight vector and marker index CSVs have explicit column names and match the LP solution
- Marker set is less redundant than one-vs-all alternatives and recovers the cell type hierarchy with fewer markers

## Limitations

- Assumes linear separability of cell type classes, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods implemented; relaxing the linear separability assumption is planned future work
- Limited to categorical labeling; no support for placing unlabeled cells on trajectories or in spatial assays (a manifold constraint is envisioned future work)
- Optimal marker choice over a label hierarchy is combinatorially difficult in general; tractability relies on the LP relaxation via the axis-aligned sparsity formulation

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] Spectral toy model marker behavior: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] Linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
- [discussion] Categorical labeling limitation: "we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at, for example, time points along cell trajectories"
- [methods] LP formulation from source card: [evidence span withheld — anchor status: not_found]
