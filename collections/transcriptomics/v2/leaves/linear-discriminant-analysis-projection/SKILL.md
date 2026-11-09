---
name: linear-discriminant-analysis-projection
description: Use when single-cell data (scRNA-seq PCs, mass cytometry markers) has
  trusted a priori class labels and the goal is a non-stochastic, linear, label-aligned
  visualization of heterogeneity or trajectory, e.g., reconstructing villus differentiation.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable (supervised
  LDA imposes label structure)
- Input exceeds ~100 dimensions and HSS feature selection would be needed — use PC-reduced
  inputs or skip HSS
- Preserving local data structure or capturing non-linear relationships is the goal;
  LDA trades local structure for class separability
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: MASS
  role: Fits the LDA model (MASS::lda) on predictors and class labels, providing the
    scaling matrix of linear discriminants
- name: hsslda
  role: R package whose makeAxes() projects data onto LD axes and whose runHSS() performs
    feature-selection-optimized LDA
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Renders scatter plots of cells on LD1/LD2 colored by class labels or landmark
    gene expression
- name: HSS-LDA
  role: Overall supervised dimensionality-reduction method combining hybrid subset
    selection with LDA for single-cell visualization
  repo: https://github.com/mamouzgar/hsslda
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:7e412d78b531a01bb4eabcca22131578e42ffbcc3a5c20ac92079a33d587c0c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - HSS-LDA
  - MASS
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# linear-discriminant-analysis-projection

## Summary

Applies linear discriminant analysis (LDA) as supervised dimensionality reduction for single-cell data, projecting cells onto linear-discriminant axes that optimally separate a priori class labels (e.g., villus zone, cell type, time point) for interpretable visualization.

## When to use

Use when single-cell data (scRNA-seq PCs, mass cytometry markers) has trusted a priori class labels and the goal is a non-stochastic, linear, label-aligned visualization of heterogeneity or trajectory, e.g., reconstructing villus differentiation.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable (supervised LDA imposes label structure)
- Input exceeds ~100 dimensions and HSS feature selection would be needed — use PC-reduced inputs or skip HSS
- Preserving local data structure or capturing non-linear relationships is the goal; LDA trades local structure for class separability

## Inputs

- Cells-by-features expression or PC-score dataframe (e.g., first 50 principal components of scRNA-seq counts)
- Vector of a priori class labels aligned to rows (e.g., train.y villus-zone labels from the Moor et al. enterocyte dataset)
- Optional: model scaling matrix (lda.res$scaling) for projecting new data via makeAxes

## Outputs

- Merged dataframe of input features, class labels, and linear-discriminant coordinates (LD1, LD2, ...) from makeAxes
- MASS lda model object with scaling coefficients and proportion-of-trace values
- ggplot2 scatter plots of cells on LD1/LD2 colored by labels or landmark gene expression (ADA, SLC2A2)

## How to apply

Prepare predictors as a cells-by-features dataframe (e.g., the first 50 principal components of the Moor et al. enterocyte scRNA-seq matrix, since HSS is not feasible above ~100 dimensions) and a matching vector of a priori labels (villus zone classes). Fit the model with MASS::lda(x = PCs, grouping = zone.labels) when running LDA without HSS feature selection, or supply custom predictors/score methods via hsslda::runHSS() when the feature count permits HSS. Generate LD coordinates for all cells with hsslda::makeAxes(df, co = lda.res$scaling), which returns a merged dataframe of input features, labels, and LD axes. Visualize cells on LD1/LD2 with ggplot2, coloring by class label and by landmark genes (e.g., ADA at the villus tip, SLC2A2 in the mid-villus), and build a UMAP embedding of the same PCs for side-by-side comparison. Rationale: LDA maximizes between-class separability while minimizing within-class variance, yielding deterministic axes — unlike stochastic UMAP — that order labeled populations linearly along the discriminants.

## Related tools

- **MASS** (Fits the LDA model (MASS::lda) on predictors and class labels, providing the scaling matrix of linear discriminants)
- **hsslda** (R package whose makeAxes() projects data onto LD axes and whose runHSS() performs feature-selection-optimized LDA) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Renders scatter plots of cells on LD1/LD2 colored by class labels or landmark gene expression)
- **HSS-LDA** (Overall supervised dimensionality-reduction method combining hybrid subset selection with LDA for single-cell visualization) — https://github.com/mamouzgar/hsslda

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = hsslda::makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- LD axes are deterministic (non-stochastic): re-running LDA on the same PCs and labels reproduces the same trajectory, unlike UMAP
- Zone labels order linearly along the LD axes, matching the published Figure 5B/5C comparison against UMAP
- Landmark gene expression is spatially coherent: ADA peaks toward the villus-tip end and SLC2A2 toward the mid-villus along LD1/LD2
- Proportion-of-trace output from the lda model shows LD1/LD2 capture the dominant discriminant variance
- makeAxes returns a merged dataframe containing all originally inputted features, labels, and the new LD columns

## Limitations

- LDA, by definition, maximizes class separability and comes at the expense of local data structure
- LDA does not capture non-linear relationships, which may limit performance on some datasets
- HSS is not feasible for more than 100 dimensions; in such cases use PCs as LDA predictors without HSS
- HSS-LDA may perform poorly when a large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) or with an increasing number of populations
- Relevant colinear features may not all be selected by HSS, and HSS may disregard features only useful for separating a single class

## Evidence

- [readme] LDA without HSS via MASS: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [abstract] Non-stochastic interpretable axes: "We implement feature selection by hybrid subset selection (HSS) and demonstrate that this computationally efficient approach generates non-stochastic, interpretable axes"
- [abstract] LDA separates a priori classes: "LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity."
- [methods] makeAxes for new data: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] LDA loses local structure: "LDA, by definition, maximizes class separability and minimizes variance within each projected class, and by doing so will come at the expense of local data structure."
