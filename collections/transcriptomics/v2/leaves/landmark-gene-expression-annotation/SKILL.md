---
name: landmark-gene-expression-annotation
description: Use when an LDA (or HSS-LDA) embedding orders cells along a known biological
  axis (e.g., villus zone labels) and you must validate or interpret the axis using
  marker genes with distinct spatial or temporal expression patterns.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — the article
  notes 'This supervised approach is inappropriate in situations in which labels are
  unknown, or in which an unbiased view of the data is preferable.'
- Input data have more than ~100 dimensions and HSS feature selection is required
  — 'in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA
  and sda still provide useful plots', so use PCs or run LDA without HSS as in this
  skill
- The goal is to resolve non-linear structure — 'LDA does not capture non-linear relationships,
  which may limit its performance on some datasets.'
tools:
- name: MASS
  role: Fits the LDA model (without HSS feature selection) whose scaling coefficients
    define the LD axes onto which cells and their landmark gene expression are projected
- name: hsslda
  role: Provides makeAxes() to generate LD coordinates for all cells, producing the
    merged dataframe used for expression-annotated visualization
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Renders LD1/LD2 scatter plots colored by class labels and by landmark gene
    expression (e.g., ADA, SLC2A2)
- name: UMAP
  role: Unsupervised embedding of the same input features used as the side-by-side
    baseline to demonstrate improved linear trajectory coherence with LDA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:ab2ff1275e5171e8f8f3ebcfa0158a2093081c2f93b3647694eee98a3d6a51af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - UMAP
  - ggplot2
  - hsslda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# landmark-gene-expression-annotation

## Summary

Annotates an LDA-ordered single-cell trajectory by coloring cells on linear discriminant axes with landmark gene expression (e.g., ADA at the villus tip, SLC2A2 in the mid-villus), verifying that biologically relevant spatial expression patterns are preserved by supervised dimensionality reduction.

## When to use

Use when an LDA (or HSS-LDA) embedding orders cells along a known biological axis (e.g., villus zone labels) and you must validate or interpret the axis using marker genes with distinct spatial or temporal expression patterns.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — the article notes 'This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable.'
- Input data have more than ~100 dimensions and HSS feature selection is required — 'in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots', so use PCs or run LDA without HSS as in this skill
- The goal is to resolve non-linear structure — 'LDA does not capture non-linear relationships, which may limit its performance on some datasets.'

## Inputs

- Single-cell expression matrix with cells as rows and genes as features (e.g., Moor et al. spatially reconstructed enterocyte scRNA-seq data)
- A priori class/zone labels per cell (train.y, e.g., villus-zone labels)
- Fitted LDA model with a scaling matrix (lda.res$scaling from MASS::lda or an HSS-LDA model)
- LD-coordinate dataframe from makeAxes containing features, labels, and LD axes
- Landmark gene expression values (e.g., ADA, SLC2A2)
- Optional: UMAP embedding of the same input features (e.g., first 50 PCs) for comparison

## Outputs

- ggplot2 scatter plots of cells on LD1/LD2 colored by zone label and by landmark gene expression (ADA, SLC2A2)
- Merged dataframe of input features, class labels, and LD coordinates
- Qualitative validation of a linear villus-tip-to-mid-villus gene-expression gradient preserved on the LD axes, relative to UMAP

## How to apply

Project all cells onto LDA axes by generating LD coordinates from the fitted model's scaling matrix (e.g., hsslda::makeAxes(df, co = lda.res$scaling) after MASS::lda on the PC features), producing a merged dataframe of input features, class labels, and LD values. Plot LD1/LD2 with ggplot2, coloring points by the a priori class labels (e.g., intestinal villus-zone labels) and then re-coloring the same embedding by expression of landmark genes (e.g., ADA, SLC2A2). Interpret the trajectory by checking that each landmark gene peaks at the expected end or region of the LD axis: ADA toward the villus tip and SLC2A2 toward the mid-villus. Compare against an unsupervised UMAP embedding of the same features (e.g., first 50 principal components) side-by-side to judge whether the supervised LD axes yield a more coherent linear order. Correct application yields a non-stochastic, interpretable linear trajectory in which zones are ordered along the LD axes and landmark gene gradients follow the expected spatial order.

## Related tools

- **MASS** (Fits the LDA model (without HSS feature selection) whose scaling coefficients define the LD axes onto which cells and their landmark gene expression are projected)
- **hsslda** (Provides makeAxes() to generate LD coordinates for all cells, producing the merged dataframe used for expression-annotated visualization) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Renders LD1/LD2 scatter plots colored by class labels and by landmark gene expression (e.g., ADA, SLC2A2))
- **UMAP** (Unsupervised embedding of the same input features used as the side-by-side baseline to demonstrate improved linear trajectory coherence with LDA)

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = hsslda::makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- The LD trajectory is non-stochastic: repeating the LDA fit and projection yields the same ordered zone arrangement along LD axes.
- Zones appear linearly ordered along the LD axes, matching the published Figure 5B/5C UMAP comparison layout.
- ADA expression peaks toward the villus-tip end of the LD embedding and SLC2A2 toward the mid-villus region, matching expected spatial biology.
- Landmark gene gradients are more spatially coherent on the LD axes than on the UMAP embedding of the same first 50 PCs.
- The makeAxes output is a merged dataframe containing the input features, the labels column, and the generated LD axes for every input cell.

## Limitations

- LDA, by maximizing class separability, comes at the expense of local data structure, so fine-grained neighborhood relationships may be distorted in the annotated embedding.
- HSS feature selection enriches for features separating multiple classes and may miss features useful for a single class, or relevant colinear features; using unsupervised PC inputs (as in this skill) sidesteps HSS but is required when HSS is not feasible above ~100 dimensions.
- HSS-LDA may perform poorly with large heterogeneity within a class or with many populations; features flagged by the workflow require follow-up statistical analysis.
- The GitHub repository was noted as still under development.

## Evidence

- [readme] LDA without HSS and makeAxes projection: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [methods] ggplot2 visualization of LD axes colored by labels: "ggplot2::ggplot(lda.df,ggplot2::aes(x = HSS_LD1, y = HSS_LD2, color = labels)) +"
- [abstract] Non-stochastic, interpretable axes claim: "We implement feature selection by hybrid subset selection (HSS) and demonstrate that this computationally efficient approach generates non-stochastic, interpretable axes"
- [abstract] LDA optimally separates a priori classes: "LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity."
- [discussion] Limitation: LDA sacrifices local structure: "LDA, by definition, maximizes class separability and minimizes variance within each projected class, and by doing so will come at the expense of local data structure."
- [other] Source card validation of landmark genes: [evidence span withheld — anchor status: not_found]
