---
name: villus-zone-trajectory-visualization
description: Use when spatial scRNA-seq data with prior zone/position labels (e.g.,
  villus zones) must be organized into a coherent linear differentiation trajectory,
  and PCA/UMAP alone give spatially incoherent orderings of landmark genes.
when_to_use_negative:
- Labels are unknown or an unbiased view of the data is preferable — the supervised
  approach is inappropriate in these situations
- More than ~100 features must be selected individually, since HSS is not feasible
  above ~100 dimensions (use PC scores as LDA predictors instead, or plain LDA)
- Preserving local data structure is critical — LDA maximizes class separability and
  comes at the expense of local structure
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASS
  role: Fits the LDA model (MASS::lda) on PC scores with zone labels, without HSS
    feature selection
- name: hsslda
  role: Provides makeAxes to project cells onto the LD axes from the MASS lda scaling
    matrix
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Visualizes cells on LD1/LD2 colored by zone label and landmark gene expression
- name: UMAP
  role: Generates a comparison embedding of the same PCs to benchmark the LDA trajectory
    (Figure 5B/5C comparison)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:2b442a9fc24ebedd47cb867d48b8430534a5509852e4975d4d7c47877ae58fd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - UMAP
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# villus-zone-trajectory-visualization

## Summary

Visualizes a linear differentiation trajectory of enterocytes along intestinal villi by applying LDA (without HSS feature selection) to the first 50 principal components of spatially reconstructed scRNA-seq data, using a priori villus-zone labels, and comparing the LD embedding against UMAP.

## When to use

Use when spatial scRNA-seq data with prior zone/position labels (e.g., villus zones) must be organized into a coherent linear differentiation trajectory, and PCA/UMAP alone give spatially incoherent orderings of landmark genes.

## When NOT to use

- Labels are unknown or an unbiased view of the data is preferable — the supervised approach is inappropriate in these situations
- More than ~100 features must be selected individually, since HSS is not feasible above ~100 dimensions (use PC scores as LDA predictors instead, or plain LDA)
- Preserving local data structure is critical — LDA maximizes class separability and comes at the expense of local structure

## Inputs

- Spatially reconstructed enterocyte scRNA-seq expression matrix (Moor et al.)
- A priori villus-zone class labels (train.y) matched to cells (train.x)
- Landmark gene expression values (e.g., ADA, SLC2A2) for coloring cells

## Outputs

- LD1/LD2 embedding of all cells colored by villus-zone label
- Merged dataframe of input features, labels, and LD coordinates from makeAxes
- Side-by-side UMAP embedding of the same PCs for comparison
- Landmark-gene (ADA, SLC2A2) expression plots along the linear trajectory

## How to apply

Load the spatial scRNA-seq expression matrix (e.g., Moor et al. enterocyte data) together with the a priori villus-zone class labels in a train.x / train.y structure. Compute the first 50 principal components and use the PC scores as LDA predictors, because HSS is not feasible above ~100 dimensions and this task calls for unsupervised-PC inputs with supervised labels. Fit the model with MASS::lda(x = PCs, grouping = zone labels), i.e., LDA without HSS feature selection, then project all cells onto the linear-discriminant axes with hsslda::makeAxes(df, co = lda.res$scaling), yielding a merged dataframe of features, labels, and LD coordinates. Visualize cells on LD1/LD2 with ggplot2, coloring by zone label and by landmark gene expression (ADA at the villus tip, SLC2A2 in the mid-villus), and build a UMAP embedding of the same PCs for side-by-side comparison (as in Figure 5B/5C).

## Related tools

- **MASS** (Fits the LDA model (MASS::lda) on PC scores with zone labels, without HSS feature selection)
- **hsslda** (Provides makeAxes to project cells onto the LD axes from the MASS lda scaling matrix) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Visualizes cells on LD1/LD2 colored by zone label and landmark gene expression)
- **UMAP** (Generates a comparison embedding of the same PCs to benchmark the LDA trajectory (Figure 5B/5C comparison))

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- LD trajectory is non-stochastic and reproducible across runs (a key property of LDA axes)
- Zones appear ordered linearly along the LD axes, matching the published Figure 5B/5C UMAP comparison
- ADA expression peaks toward the villus-tip end of the trajectory and SLC2A2 toward the mid-villus, preserving biologically relevant spatial patterns
- Landmark gene spatial patterns are more coherent on LD axes than on the UMAP embedding of the same PCs
- makeAxes output contains a merged dataframe with all input features, class labels, and LD coordinates (schema check)

## Limitations

- LDA, by definition, maximizes class separability and minimizes within-class variance, coming at the expense of local data structure
- LDA does not capture non-linear relationships, which may limit performance on some datasets
- HSS-LDA may not perform well when a large degree of heterogeneity exists within a given class
- Performance will suffer with an increasing number of populations
- The supervised approach is inappropriate when labels are unknown or an unbiased view is preferable

## Evidence

- [methods] LDA without HSS on PCs: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes"
- [methods] makeAxes projection: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] HSS dimension limit: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
- [abstract] Non-stochastic axes: "We implement feature selection by hybrid subset selection (HSS) and demonstrate that this computationally efficient approach generates non-stochastic, interpretable axes"
- [discussion] Local structure trade-off: "LDA, by definition, maximizes class separability and minimizes variance within each projected class, and by doing so will come at the expense of local data structure."
- [readme] Package purpose: "LDA identifies linear combinations of predictors that optimally separate a priori labels, enabling users to tailor visualizations to separate specific aspects of cellular heterogeneity."
