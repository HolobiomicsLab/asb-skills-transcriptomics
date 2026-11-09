---
name: principal-component-preprocessing-for-lda
description: Use when the input feature matrix exceeds ~100 dimensions (e.g., transcriptomic
  data with thousands of genes) so HSS feature selection cannot run, yet a priori
  class labels exist and a supervised linear-discriminant visualization is wanted.
when_to_use_negative:
- When class labels are unknown or an unbiased view of the data is preferable (supervised
  LDA is inappropriate)
- When the feature count is low enough (<=~100 dimensions) that HSS feature selection
  is feasible and interpretable feature drivers are wanted
- When local data structure or non-linear relationships must be preserved (LDA maximizes
  class separability at the expense of local structure and cannot capture non-linear
  relationships)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: MASS
  role: Fits LDA on PC scores without HSS feature selection (MASS::lda)
- name: hsslda
  role: Provides makeAxes to generate LD axes from the lda$scaling matrix and project
    all cells
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Visualizes cells on LD1/LD2 colored by labels or marker gene expression
- name: R
  role: Runtime environment in which the PC computation, MASS::lda fit, and makeAxes
    projection are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:b62ab31544819dd9a4c6a65ac24fe68c7f0dd192f9f296c5eb87af04bc0fbf4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# principal-component-preprocessing-for-lda

## Summary

Preprocess high-dimensional single-cell data (e.g., scRNA-seq expression matrices) by computing principal component scores and feeding these PC scores as predictors into LDA, because HSS feature selection is not feasible above ~100 dimensions but LDA visualization is still desired.

## When to use

Use when the input feature matrix exceeds ~100 dimensions (e.g., transcriptomic data with thousands of genes) so HSS feature selection cannot run, yet a priori class labels exist and a supervised linear-discriminant visualization is wanted.

## When NOT to use

- When class labels are unknown or an unbiased view of the data is preferable (supervised LDA is inappropriate)
- When the feature count is low enough (<=~100 dimensions) that HSS feature selection is feasible and interpretable feature drivers are wanted
- When local data structure or non-linear relationships must be preserved (LDA maximizes class separability at the expense of local structure and cannot capture non-linear relationships)

## Inputs

- High-dimensional single-cell expression matrix (cells x genes, e.g., scRNA-seq counts/normalized expression)
- Vector of a priori class labels per cell (train.y, e.g., villus-zone labels)
- Number of PCs to retain (e.g., first 50 PC scores)

## Outputs

- MASS::lda model object with $scaling matrix of LD coefficients
- Merged dataframe of input features, class labels, and LD coordinates from hsslda::makeAxes
- LD1/LD2 scatter plots colored by class label or marker gene expression (ggplot2)

## How to apply

Load the expression matrix together with a priori class labels into a train.x / train.y structure. Compute the first 50 principal components of the data and use the PC scores as LDA predictors, since HSS is not feasible above ~100 dimensions and this preserves an unsupervised feature representation while retaining supervised labels for the LDA step. Fit the model with MASS::lda(x = PCs, grouping = labels), i.e., LDA without HSS feature selection, rather than hsslda::runHSS. Project all cells onto the linear-discriminant axes using hsslda::makeAxes(df, co = lda.res$scaling), which returns a merged dataframe of input features, labels, and LD coordinates for ggplot2 visualization (e.g., LD1 vs LD2 colored by zone label or landmark gene expression).

## Related tools

- **MASS** (Fits LDA on PC scores without HSS feature selection (MASS::lda))
- **hsslda** (Provides makeAxes to generate LD axes from the lda$scaling matrix and project all cells) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Visualizes cells on LD1/LD2 colored by labels or marker gene expression)
- **R** (Runtime environment in which the PC computation, MASS::lda fit, and makeAxes projection are executed)

## Examples

```
lda.res = MASS::lda(x = PCs, grouping = train.y); lda.df = hsslda::makeAxes(df = PCs, co = lda.res$scaling)
```

## Evaluation signals

- LD trajectory is non-stochastic and interpretable: re-running the PCA+LDA pipeline reproduces the same cell ordering along LD axes
- Class labels (e.g., villus zones) order linearly along LD axes in the LD1/LD2 plot
- Landmark genes with known spatial patterns (e.g., ADA peaking at one end, SLC2A2 mid-way) show spatially coherent expression gradients across the LD embedding
- Side-by-side UMAP comparison of the same PC scores shows the LDA axes give a more linear, ordered trajectory
- makeAxes output contains all input rows with LD coordinate columns appended and no dropped cells

## Limitations

- HSS is not feasible for more than 100 dimensions, hence PCs must be used as predictors instead
- LDA maximizes class separability and will come at the expense of local data structure
- LDA does not capture non-linear relationships, which may limit performance on some datasets
- HSS-LDA may not perform well when a large degree of heterogeneity exists within a given class or with an increasing number of populations

## Evidence

- [discussion] HSS not feasible above ~100 dims: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
- [readme] MASS + makeAxes workflow: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [methods] makeAxes projection: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [abstract] LDA class separation rationale: [evidence span withheld — anchor status: not_found]
