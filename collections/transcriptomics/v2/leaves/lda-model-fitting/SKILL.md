---
name: lda-model-fitting
description: Use when you have a cells-by-markers dataframe (e.g., CyTOF Morphometry,
  T cell metabolic regulome, or Chromotyping data) with an a priori class-label vector
  (cell type, cell-cycle phase, time point) and want supervised dimensionality reduction
  onto LD axes.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable (supervised
  LDA is inappropriate)
- More than 100 dimensions, where HSS feature selection is not feasible
- Data with strong non-linear relationships or large heterogeneity within a class
  (e.g., PBMCs labeled by time point), or many populations, where LDA/HSS-LDA performance
  suffers
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: hsslda
  role: R package implementing LDA with Hybrid Subset Selection feature selection;
    runHSS() fits the final LDA on the elbow-optimal marker set and makeAxes() projects
    new data
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Provides MASS::lda used to fit the LDA model (lda(y ~ ., data = x[, markers]))
    and the plain-LDA baseline without HSS
- name: ggplot2
  role: Plots the projected HSS_LD1/HSS_LD2 axes colored by class labels and renders
    the ElbowPlot
- name: dplyr
  role: Package dependency to attach explicitly before using its functions in the
    hsslda workflow
- name: tidyr
  role: Package dependency to attach explicitly before using its functions in the
    hsslda workflow
- name: stats
  role: Package dependency to attach explicitly before using its functions in the
    hsslda workflow
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:eedb19100258dec28ef78084a00ffb46f251334f4c6cbeb404a8d580aa2c2f83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - dplyr
  - ggplot2
  - hsslda
  - stats
  - tidyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# lda-model-fitting

## Summary

Fit a linear discriminant analysis (LDA) model on single-cell feature tables (e.g., CyTOF cells-by-markers dataframes) using MASS::lda in R, with the hsslda package, to obtain linear discriminant (LD) axes that optimally separate a priori class labels for supervised dimensionality reduction and visualization.

## When to use

Use when you have a cells-by-markers dataframe (e.g., CyTOF Morphometry, T cell metabolic regulome, or Chromotyping data) with an a priori class-label vector (cell type, cell-cycle phase, time point) and want supervised dimensionality reduction onto LD axes.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable (supervised LDA is inappropriate)
- More than 100 dimensions, where HSS feature selection is not feasible
- Data with strong non-linear relationships or large heterogeneity within a class (e.g., PBMCs labeled by time point), or many populations, where LDA/HSS-LDA performance suffers

## Inputs

- cells-by-markers dataframe (single-cell CyTOF marker intensities, e.g., TcellHartmann2020_sampleData with 15 channels such as GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A)
- a priori class-label vector y matching training data rows (e.g., day0–day5 labels)
- optional elbow-optimal marker list from hss.result HSSscores/ElbowPlot
- model scaling coefficient matrix (hss.result$`HSS-LDA-model`$scaling) for makeAxes projection

## Outputs

- fitted LDA model object (MASS lda with coefficients of linear discriminants, group means, prior probabilities, proportion of trace)
- HSS-LDA-result dataframe merging all input markers, class labels, and LD axes (LD1–LD5 / HSS_LD1, HSS_LD2)
- ElbowPlot (ggplot2 object) with the calculated elbow point defining the optimal feature set
- HSSscores table of scores for each subsetted model
- projected axes for new data via makeAxes(df, co)

## How to apply

Load the single-cell dataset as a cells-by-markers dataframe x and a matching class-label vector y, then run hybrid subset selection via hsslda::runHSS(x, y, score.method, custom.score.method, downsample) — with score.method = 'euclidean' (default) and downsample = FALSE to use all cells — which performs forward plus reverse stepwise selection around LDA and compiles an elbow point from the HSSscores table (ElbowPlot) to pick the optimal marker set. The final model is fitted internally as MASS::lda(y ~ ., data = x[, markers]) on the elbow-optimal markers (e.g., 10 of 15 markers in the T cell regulome vignette), and all input cells are projected onto the resulting LD1–LD5 axes; LD1 explained 0.7906 of the trace in the vignette. For a plain-LDA baseline without feature selection, call MASS::lda(x = train.x, grouping = train.y) directly on the full marker set. Apply the fitted scaling coefficients to new data with makeAxes(df, co) using the model's scaling matrix, then visualize HSS_LD1/HSS_LD2 (or LD1/LD2) with ggplot2 colored by class labels.

## Related tools

- **hsslda** (R package implementing LDA with Hybrid Subset Selection feature selection; runHSS() fits the final LDA on the elbow-optimal marker set and makeAxes() projects new data) — https://github.com/mamouzgar/hsslda
- **MASS** (Provides MASS::lda used to fit the LDA model (lda(y ~ ., data = x[, markers])) and the plain-LDA baseline without HSS)
- **ggplot2** (Plots the projected HSS_LD1/HSS_LD2 axes colored by class labels and renders the ElbowPlot)
- **dplyr** (Package dependency to attach explicitly before using its functions in the hsslda workflow)
- **tidyr** (Package dependency to attach explicitly before using its functions in the hsslda workflow)
- **stats** (Package dependency to attach explicitly before using its functions in the hsslda workflow)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE); lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- The fitted model object exists and its Call is lda(y ~ ., data = x[, markers]) with the marker set matching the computed elbow point (e.g., 10 of 15 markers in the vignette)
- The HSS-LDA-result table contains all initially inputted markers, class labels, and the LD axes, with all cells projected onto HSS-LD axes
- Proportion of trace values are reported for each LD axis (vignette: LD1 = 0.7906, LD2 = 0.1890, LD3 = 0.0115, LD4 = 0.0066, LD5 = 0.0023)
- Retained marker counts match reported reduced sets: 13 of 17 features (Morphometry), 8 of 32 (T cell metabolic regulome), 24 of 32 (Chromotyping)
- Qualitative class-label separation on HSS_LD1/HSS_LD2 plots retains or improves over the plain-LDA projection built with makeAxes(df, lda.res$scaling)

## Limitations

- LDA maximizes class separability and minimizes within-class variance at the expense of local data structure, and does not capture non-linear relationships
- Relevant colinear features may not all be selected by HSS because keeping multiple colinear features may not improve separability; features useful for separating only a single class may be disregarded
- Performance suffers with an increasing number of populations and with large heterogeneity within a given class; features selected by HSS require follow-up analysis for the nature and statistical significance of differences

## Evidence

- [abstract] We repurpose the classification algorithm, linear discriminant analysis (LDA), for supervised dimensionality reduction of single-cell data.: "We repurpose the classification algorithm, linear discriminant analysis (LDA), for supervised dimensionality reduction of single-cell data."
- [methods] To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes"
- [methods] ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set.: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [readme] You can use this final model to apply the same dimensionality reduction to new data using makeAxes.: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [readme] lda(y ~ ., data = x[, markers]) with Proportion of trace: LD1 0.7906: "Call:
lda(y ~ ., data = x[, markers])"
