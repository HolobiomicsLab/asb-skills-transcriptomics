---
name: hss-feature-selection-optimization
description: Use when you have high-dimensional single-cell data (e.g., CyTOF markers)
  with known a priori class labels and need a non-stochastic, interpretable reduced
  feature set that maximizes class separability for visualization or downstream embedding.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — supervised
  HSS-LDA is inappropriate in these situations
- Dataset has more than 100 dimensions, where HSS is not feasible
- Large degree of heterogeneity exists within a given class or the number of populations
  is very high, where HSS-LDA performance suffers
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: hsslda
  role: R package implementing runHSS, makeAxes, and ElbowPlot for hybrid subset selection
    with LDA
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Provides lda() used to fit LDA without HSS; hsslda's saved model is lda(y
    ~ ., data = x[, markers])
- name: ggplot2
  role: Renders the ElbowPlot and downstream biaxial HSS_LD1 vs HSS_LD2 scatter plots
- name: dplyr
  role: Used to merge HSS-LD columns from separate HSS-LDA runs into a combined feature
    dataframe
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:b312d58838ad8c86a2173d13aacd9ea53a652c62f3742ff803f9fa881283f514
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# hss-feature-selection-optimization

## Summary

Optimize feature selection for supervised LDA dimensionality reduction of single-cell data using hybrid subset selection (HSS), a heuristic combining forward and reverse stepwise selection, implemented in the hsslda R package. HSS identifies a marker set that enhances separation of a priori class labels (e.g., cell-cycle phase, cell line, time point) before fitting the LDA model.

## When to use

Use when you have high-dimensional single-cell data (e.g., CyTOF markers) with known a priori class labels and need a non-stochastic, interpretable reduced feature set that maximizes class separability for visualization or downstream embedding.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — supervised HSS-LDA is inappropriate in these situations
- Dataset has more than 100 dimensions, where HSS is not feasible
- Large degree of heterogeneity exists within a given class or the number of populations is very high, where HSS-LDA performance suffers

## Inputs

- Cells-by-markers dataframe (x), e.g., CyTOF expression matrix with arcsinh-transformed marker columns
- Vector of a priori class labels (y) matching training data rows
- Optional custom separation metric function accepting (x, y, cols)
- Optional downsample boolean (default TRUE) to improve runtime

## Outputs

- hss.result object containing HSSscores table of scores for each subsetted model
- ElbowPlot (ggplot-configurable) with calculated elbow point for the optimal feature set
- Final HSS-LDA-model (MASS-style lda object with $scaling coefficients)
- HSS-LDA-result dataframe merging original markers, class labels, and LD axes (LD1, LD2, ...) for all input cells
- downsampled.cells.included.in.analysis boolean vector

## How to apply

Call hsslda::runHSS(x = cells-by-markers dataframe, y = class label vector, score.method = one of 'euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom') to iteratively add/remove markers via forward and reverse stepwise selection. Inspect the returned ElbowPlot to identify the elbow point marking the optimal feature set size, then use the saved HSS-LDA-model (fit via lda(y ~ ., data = x[, markers]) on the optimal markers) for projection. Apply the model's $scaling coefficients to new data with makeAxes(df, co) to generate HSS_LD axes. Set downsample = FALSE to use all input cells; a custom separation metric can be supplied via custom.score.method with score.method = 'custom'.

## Related tools

- **hsslda** (R package implementing runHSS, makeAxes, and ElbowPlot for hybrid subset selection with LDA) — https://github.com/mamouzgar/hsslda
- **MASS** (Provides lda() used to fit LDA without HSS; hsslda's saved model is lda(y ~ ., data = x[, markers]))
- **ggplot2** (Renders the ElbowPlot and downstream biaxial HSS_LD1 vs HSS_LD2 scatter plots)
- **dplyr** (Used to merge HSS-LD columns from separate HSS-LDA runs into a combined feature dataframe)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE); lda.df.newData = makeAxes(df = newdata, co = hss.result$`HSS-LDA-model`$scaling)
```

## Evaluation signals

- ElbowPlot shows a clear elbow point; the optimal feature set size is read at the elbow of the HSS score curve
- runHSS log reports sequential marker counts (e.g., 'Number of markers: 2' up to the full set) from both forward and reverse steps, confirming hybrid stepwise search
- HSS-LDA-result dataframe contains all input markers, the label vector, and LD axes; prior probabilities of groups in the saved model match label class frequencies
- Projection of cells via makeAxes yields HSS_LD axes on which a priori labels separate (e.g., day0–day5 groups distinct in the biaxial ggplot colored by labels)
- Results are non-stochastic: re-running HSS on the same data with the same score.method yields the same optimal feature set

## Limitations

- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class
- Relevant colinear features may not all be selected by HSS since keeping multiple colinear features may not improve model separability
- LDA maximizes class separability at the expense of local data structure and does not capture non-linear relationships
- HSS is not feasible for more than 100 dimensions
- Selected features are differentially expressed by one or more classes and require follow-up analysis for the nature and statistical significance of differences

## Evidence

- [intro] HSS heuristic rationale: "We implement hybrid subset selection (HSS), a heuristic approach using elements of both forward and reverse stepwise selection, to identify a set of features that enable enhanced separation of these"
- [methods] runHSS signature: "runHSS(x, y, score.method, custom.score.method, downsample)"
- [methods] Elbow plot output: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [methods] makeAxes projection to new data: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [readme] downsample parameter: "(optional) downsample: Boolean, defaults to TRUE. Downsamples data to improve runtime. Set to FALSE to use all input data."
- [discussion] HSS not feasible beyond 100 dimensions: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
