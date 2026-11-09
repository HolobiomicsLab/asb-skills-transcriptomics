---
name: single-cell-cytometry-data-handling
description: Use when you have single-cell data (cells x markers, e.g. CyTOF) with
  a priori class labels (cell type, cell-cycle phase, time point) and need an interpretable,
  non-stochastic low-dimensional visualization that maximizes class separation.
when_to_use_negative:
- Labels are unknown or an unbiased view of the data is preferable — the supervised
  approach is inappropriate in these situations
- Input has more than 100 dimensions — HSS is not feasible there; use plain LDA (MASS)
  or sda instead
- A large degree of heterogeneity exists within a given class or there are many populations
  — HSS-LDA performance will suffer
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: hsslda
  role: R package providing runHSS (hybrid subset selection) and makeAxes for the
    full HSS-LDA pipeline
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: fits the final LDA model (lda(y ~ ., data = x[, markers])) on the elbow-optimal
    markers; can also run LDA without HSS
- name: ggplot2
  role: renders and customizes the elbow plot and the LD-axis scatter plots of labeled
    cells
- name: R
  role: runtime environment for the hsslda package and pipeline scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:e655daea427deb98afdd13db1a7f62bf2bb27d5d3b87a7fb6dc92a01a3166d62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-cytometry-data-handling

## Summary

Implements the HSS-LDA pipeline of Amouzgar et al. (Patterns, 2022): hybrid subset selection (HSS) feature selection followed by linear discriminant analysis (LDA) for supervised dimensionality reduction of single-cell mass cytometry (CyTOF) data, delivered via the hsslda R package.

## When to use

Use when you have single-cell data (cells x markers, e.g. CyTOF) with a priori class labels (cell type, cell-cycle phase, time point) and need an interpretable, non-stochastic low-dimensional visualization that maximizes class separation.

## When NOT to use

- Labels are unknown or an unbiased view of the data is preferable — the supervised approach is inappropriate in these situations
- Input has more than 100 dimensions — HSS is not feasible there; use plain LDA (MASS) or sda instead
- A large degree of heterogeneity exists within a given class or there are many populations — HSS-LDA performance will suffer

## Inputs

- cells-by-markers dataframe (rows = cells, columns = marker intensities, e.g. 15 metabolic CyTOF channels: GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A)
- vector of a priori class labels y matching training rows (e.g. day0–day5 time points)
- optional custom separation metric function for score.method = 'custom'

## Outputs

- hss.result object with HSSscores table of all subset-model scores
- ElbowPlot (ggplot2-configurable) showing the computed elbow point for the optimal feature set
- final HSS-LDA-model (MASS lda object with LD coefficients and proportion-of-trace)
- HSS-LDA-result dataframe merging all input markers, class labels, and LD1-LD5 discriminant axes
- downsampled.cells.included.in.analysis boolean vector (all TRUE if downsampling was off)

## How to apply

Load a cells-by-markers dataframe (e.g. TcellHartmann2020_sampleData with 15 metabolic markers and a 'labels' column) and call hsslda::runHSS(x, y, score.method, downsample), choosing a scoring metric from 'euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom'. runHSS performs hybrid subset selection — a heuristic combining forward and reverse stepwise selection — logging scores at each marker-set size in the HSSscores table. Read the elbow point from hss.result$ElbowPlot to pick the optimal marker set (in the vignette, 10 of 15 markers), then fit the final LDA via MASS::lda(y ~ ., data = x[, markers]) on the elbow-optimal markers. Project all input cells onto the LD1-LD5 axes and merge markers, class labels, and LD axes into hss.result$`HSS-LDA-result`; reuse the model on new data with makeAxes(df, co = model$scaling).

## Related tools

- **hsslda** (R package providing runHSS (hybrid subset selection) and makeAxes for the full HSS-LDA pipeline) — https://github.com/mamouzgar/hsslda
- **MASS** (fits the final LDA model (lda(y ~ ., data = x[, markers])) on the elbow-optimal markers; can also run LDA without HSS)
- **ggplot2** (renders and customizes the elbow plot and the LD-axis scatter plots of labeled cells)
- **R** (runtime environment for the hsslda package and pipeline scripts)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- The fitted model object exists and the HSS-LDA-result table contains all initially inputted markers, class labels, and LD axes (LD1–LD5)
- The number of selected features matches the elbow point computed from the HSSscores table (vignette: 10 of 15 markers)
- Proportion of trace is dominated by LD1 (vignette value: 0.7906, then LD2 = 0.1890)
- If downsampling = FALSE, every entry in downsampled.cells.included.in.analysis is TRUE and the result includes all input data projected onto the HSS-LD axes
- Applying the saved model's scaling matrix to new data via makeAxes reproduces the same LD axes as the training projection

## Limitations

- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class
- Relevant colinear features may not all be selected by HSS, as keeping multiple colinear features may not improve model separability
- LDA maximizes class separability and minimizes within-class variance at the expense of local data structure, and does not capture non-linear relationships
- Features selected by HSS are differentially expressed by one or more labeled classes and require follow-up analysis for the nature and statistical significance of differences

## Evidence

- [readme] runHSS signature: [evidence span withheld — anchor status: not_found]
- [intro] feature-selection rationale: "We implement hybrid subset selection (HSS), a heuristic approach using elements of both forward and reverse stepwise selection, to identify a set of features"
- [readme] model fitting: [evidence span withheld — anchor status: not_found]
- [methods] projection to new data: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] supervision caveat: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
