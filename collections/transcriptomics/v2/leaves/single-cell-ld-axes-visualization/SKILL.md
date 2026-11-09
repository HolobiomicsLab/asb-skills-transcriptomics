---
name: single-cell-ld-axes-visualization
description: Use when cells from an unseen condition or subset must be visualized
  on axes learned from labeled training cells (e.g., projecting CD57high T cells onto
  CD57low-trained LD axes over day0–day5 TCR stimulation) to compare progression states.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — the article
  states 'This supervised approach is inappropriate in situations in which labels
  are unknown, or in which an unbiased view of the data is preferable.'
- The biology is expected to be non-linear along the labeled classes — LDA 'does not
  capture non-linear relationships, which may limit its performance on some datasets.'
- There is large within-class heterogeneity or very many populations (e.g., heterogeneous
  PBMCs labeled by time point) — 'HSS-LDA may not perform well in situations in which
  a large degree of heterogeneity exists within a given class' and 'performance will
  suffer with an increasing number of populations.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: hsslda
  role: R package providing runHSS() (LDA with hybrid subset selection feature selection),
    the trained HSS-LDA model object, and makeAxes() to apply the same dimensionality
    reduction to new data
  repo: mamouzgar/hsslda
- name: MASS
  role: Used to run plain LDA without HSS feature selection (MASS::lda(x = train.x,
    grouping = train.y)) when all input features are retained; its $scaling matrix
    feeds makeAxes()
- name: makeAxes
  role: hsslda function that projects new/unseen cells onto the trained LD axes using
    the model's scaling coefficients
  repo: mamouzgar/hsslda
- name: ggplot2
  role: Visualization of the projected cells and per-time-point mean LD coordinates,
    e.g., ggplot(lda.df, aes(x = HSS_LD1, y = HSS_LD2, color = labels)) + geom_point()
- name: R
  role: Implementation environment for the hsslda package, MASS::lda(), and downstream
    plotting/export
  repo: mamouzgar/hsslda
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:a55ac270a8efa6c150357b56f5cd1dc177b64b4fa2546c326543d7cee270f5b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - ggplot2
  - hsslda
  - makeAxes
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-ld-axes-visualization

## Summary

Projects single-cell data (e.g., CyTOF metabolic markers) onto linear discriminant (LD) axes trained on labeled reference cells — using MASS::lda() or hsslda model scaling coefficients plus makeAxes() — to visualize unseen populations (such as CD57high senescent T cells) in the coordinate system of a training population (CD57low T cells) across experimental time points.

## When to use

Use when cells from an unseen condition or subset must be visualized on axes learned from labeled training cells (e.g., projecting CD57high T cells onto CD57low-trained LD axes over day0–day5 TCR stimulation) to compare progression states.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — the article states 'This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable.'
- The biology is expected to be non-linear along the labeled classes — LDA 'does not capture non-linear relationships, which may limit its performance on some datasets.'
- There is large within-class heterogeneity or very many populations (e.g., heterogeneous PBMCs labeled by time point) — 'HSS-LDA may not perform well in situations in which a large degree of heterogeneity exists within a given class' and 'performance will suffer with an increasing number of populations.'

## Inputs

- Training data frame of single cells (rows) by 15 metabolic marker channels (GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A) with class labels (day0–day5), e.g., TcellHartmann2020_sampleData
- Unseen/new-condition single-cell data (e.g., CD57high T cells) with the same marker columns
- Trained LDA model scaling coefficients (model$scaling from MASS::lda() or hss.result$`HSS-LDA-model`)

## Outputs

- Projected LD-axis coordinates for the new data (data frame with LD1..LDn or HSS_LD1/HSS_LD2 columns plus labels)
- Per-time-point mean LD coordinates for training and projected populations
- ggplot2 visualization of the populations on the LD manifold (cf. Figures 2G-2K)
- CSV export of projected coordinates (e.g., cd57high_ld_projection.csv)

## How to apply

Train an LDA model on the labeled reference cells — for the T cell metabolic regulome dataset this means the 15 metabolic marker channels (GLUT1 through HIF1A) from TcellHartmann2020_sampleData grouped by day0–day5 labels — either via hsslda::runHSS() with feature selection by hybrid subset selection (HSS) or via MASS::lda() without feature selection. Extract the model's scaling coefficients (the linear discriminant loadings) and apply makeAxes(df = newdata, co = model$scaling) to project the unseen cells (e.g., CD57high) into the same LD coordinate system without retraining. Compute per-time-point mean LD coordinates for both training and projected populations and plot with ggplot2 (e.g., HSS_LD1 vs HSS_LD2 colored by labels) to assess where the unseen population falls on the manifold. The rationale is that LDA maximizes class separability along a small number of axes, so a stunted or shifted population should map onto an earlier or distinct position relative to the reference time-course — here, CD57high day-5 cells overlapping CD57low days 2–3.

## Related tools

- **hsslda** (R package providing runHSS() (LDA with hybrid subset selection feature selection), the trained HSS-LDA model object, and makeAxes() to apply the same dimensionality reduction to new data) — mamouzgar/hsslda
- **MASS** (Used to run plain LDA without HSS feature selection (MASS::lda(x = train.x, grouping = train.y)) when all input features are retained; its $scaling matrix feeds makeAxes())
- **makeAxes** (hsslda function that projects new/unseen cells onto the trained LD axes using the model's scaling coefficients) — mamouzgar/hsslda
- **ggplot2** (Visualization of the projected cells and per-time-point mean LD coordinates, e.g., ggplot(lda.df, aes(x = HSS_LD1, y = HSS_LD2, color = labels)) + geom_point())
- **R** (Implementation environment for the hsslda package, MASS::lda(), and downstream plotting/export) — mamouzgar/hsslda

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- The projection uses the training model's $scaling coefficients via makeAxes() — verify the output LD columns (LD1..LDn or HSS_LD1/HSS_LD2) exist for every input cell and no retraining occurred on the new data.
- Biological validation (Hartmann 2020 T cell dataset): the CD57high day-5 mean LD coordinates should overlap the CD57low mean coordinates at approximately days 2–3 of TCR stimulation (cf. Figures 2G-2K).
- The top features by LD1 loading magnitude (top-3 LD1 markers) should be significantly differentially expressed between CD57low and CD57high cells at all time points — this follow-up statistical test is required because 'Features selected by HSS are differentially expressed by one or more labeled classes, and follow-up analysis should be performed.'
- When run with HSS, check the elbow plot (hss.result$ElbowPlot) confirms the optimal feature set, and the model summary's 'Proportion of trace' shows LD1 dominates (e.g., ~0.79 for LD1 vs ~0.19 for LD2 in the README example), justifying LD1-focused interpretation.
- Downsampling status check: if downsample = FALSE was set, all rows of 'downsampled.cells.included.in.analysis' should be TRUE, ensuring the projection is computed on the full dataset.

## Limitations

- HSS feature selection may omit relevant colinear features ('relevant colinear features may not all be selected by HSS as the separability of the model may not be improved by keeping multiple colinear features') and may disregard features useful for separating only a single class.
- LDA maximizes class separability and minimizes within-class variance at the expense of local data structure, and cannot capture non-linear relationships.
- HSS is not feasible above 100 dimensions; performance degrades with increasing numbers of populations or large within-class heterogeneity.
- Selected/loaded features require follow-up statistical analysis to establish the nature and significance of differences between labeled classes.

## Evidence

- [abstract] We repurpose the classification algorithm, linear discriminant analysis (LDA), for supervised dimensionality reduction of single-cell data.: "We repurpose the classification algorithm, linear discriminant analysis (LDA), for supervised dimensionality reduction of single-cell data."
- [methods] makeAxes for new data: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] CD57high on CD57low-trained axes: "on the T cell metabolic regulome dataset, in which the stunted metabolic states of CD57high T cells were visualized on LD axes trained on CD57low T cells"
- [readme] MASS + makeAxes without HSS: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [readme] ggplot2 projection plotting: "ggplot2::ggplot(lda.df,ggplot2::aes(x = HSS_LD1, y = HSS_LD2, color = labels)) +
  ggplot2::geom_point()"
