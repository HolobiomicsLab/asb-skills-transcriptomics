---
name: single-cell-dimensionality-reduction
description: Use when single-cell data (e.g., CyTOF markers, scRNA-seq PCs) come with
  a priori class labels such as cell type, cell-cycle phase, or experimental time
  point, and the goal is a supervised visualization whose axes separate those labels.
when_to_use_negative:
- Labels are unknown or an unbiased view of the data is preferable — the approach
  is supervised and inappropriate in those situations.
- Data have more than ~100 dimensions and HSS feature selection is required (HSS is
  not feasible >100 dimensions); use LDA/sda on PCs instead.
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled
  by time point) or the number of populations is very large — performance suffers.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: hsslda
  role: R package implementing runHSS (hybrid subset selection), makeAxes, and ElbowPlot
    for LDA-based supervised dimensionality reduction
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Provides lda() to fit LDA without HSS when feature selection is not feasible;
    its $scaling feeds makeAxes
- name: ggplot2
  role: Visualization of cells on LD axes, e.g. ggplot(lda.df, aes(x = HSS_LD1, y
    = HSS_LD2, color = labels)) + geom_point()
- name: dplyr
  role: Dependency of hsslda; not attached by loading the package, call library(<dep>)
    explicitly
- name: tidyr
  role: Dependency of hsslda; not attached by loading the package, call library(<dep>)
    explicitly
- name: stats
  role: Dependency of hsslda; not attached by loading the package, call library(<dep>)
    explicitly
- name: entropy
  role: Dependency needed for pixel-based scoring metrics (pixel.entropy, pixel.density)
    via create_pixel_grid
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:d8847bb8798bb627d47f3073ea7e13bf008f9562b91adda109cee55e4c79d71b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - dplyr
  - entropy
  - ggplot2
  - hsslda
  - stats
  - tidyr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-dimensionality-reduction

## Summary

Supervised dimensionality reduction of single-cell data using linear discriminant analysis (LDA) with feature selection by hybrid subset selection (HSS), implemented in the R package hsslda, to generate non-stochastic, interpretable axes that optimally separate a priori class labels.

## When to use

Use when single-cell data (e.g., CyTOF markers, scRNA-seq PCs) come with a priori class labels such as cell type, cell-cycle phase, or experimental time point, and the goal is a supervised visualization whose axes separate those labels.

## When NOT to use

- Labels are unknown or an unbiased view of the data is preferable — the approach is supervised and inappropriate in those situations.
- Data have more than ~100 dimensions and HSS feature selection is required (HSS is not feasible >100 dimensions); use LDA/sda on PCs instead.
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) or the number of populations is very large — performance suffers.

## Inputs

- dataframe of cells (rows) x markers/genes (columns), e.g. train.x with 15 metabolic markers (GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A)
- vector of a priori class labels matching training rows, e.g. train.y of day0–day5
- optional first 50 principal components as LDA predictors for high-dimensional transcriptomic data (HSS not feasible above ~100 dimensions)
- optional custom separation metric function for score.method = 'custom'

## Outputs

- hss.result object with HSSscores table of scores for each subsetted model
- ElbowPlot (ggplot-configurable) showing the calculated elbow point for the optimal feature set
- HSS-LDA-model: final MASS lda model on the optimal feature set
- HSS-LDA-result: merged dataframe of input markers, class labels, and LD axes (LD1..LDn / HSS_LD1, HSS_LD2)
- downsampled.cells.included.in.analysis boolean vector (all TRUE when downsampling was not performed)

## How to apply

Split the input into a feature matrix train.x (e.g., the 15 metabolic markers GLUT1–HIF1A from TcellHartmann2020_sampleData) and a label vector train.y (e.g., day0–day5), then call hsslda::runHSS() with a score.method from 'euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom' (with a user function passed via custom.score.method). HSS combines forward and reverse stepwise selection, scoring each candidate marker subset to find the feature set that best separates the a priori labels; set downsample = FALSE to score all cells rather than the runtime-saving subset. Inspect the returned HSSscores table and ElbowPlot to determine the optimal feature set at the elbow point of score versus number of markers. Use the saved HSS-LDA-model like any R model, project cells onto the LD axes (HSS_LD1, HSS_LD2, ...) from the HSS-LDA-result merged dataframe, and visualize with ggplot2. For >~100 dimensions where HSS is infeasible, run LDA without feature selection via MASS::lda and project with hsslda::makeAxes(df, co = lda.res$scaling).

## Related tools

- **hsslda** (R package implementing runHSS (hybrid subset selection), makeAxes, and ElbowPlot for LDA-based supervised dimensionality reduction) — https://github.com/mamouzgar/hsslda
- **MASS** (Provides lda() to fit LDA without HSS when feature selection is not feasible; its $scaling feeds makeAxes)
- **ggplot2** (Visualization of cells on LD axes, e.g. ggplot(lda.df, aes(x = HSS_LD1, y = HSS_LD2, color = labels)) + geom_point())
- **dplyr** (Dependency of hsslda; not attached by loading the package, call library(<dep>) explicitly)
- **tidyr** (Dependency of hsslda; not attached by loading the package, call library(<dep>) explicitly)
- **stats** (Dependency of hsslda; not attached by loading the package, call library(<dep>) explicitly)
- **entropy** (Dependency needed for pixel-based scoring metrics (pixel.entropy, pixel.density) via create_pixel_grid)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- hss_scores.csv exists and contains one scored row for every evaluated candidate subset spanning subset sizes 2–15
- The reported elbow point on the ElbowPlot identifies the optimal feature set consistent with the HSSscores table
- Projected axes are non-stochastic and interpretable: re-running yields identical LD coordinates and landmarks (e.g., ADA at villus tip, SLC2A2 mid-villus) show biologically ordered patterns along LD axes
- downsampled.cells.included.in.analysis is all TRUE when downsample = FALSE, confirming all cells were scored
- Final model output matches the expected lda structure (prior probabilities of groups, group means, coefficients of linear discriminants, proportion of trace)

## Limitations

- HSS is not feasible for more than 100 dimensions; use plain LDA/sda on PCs instead
- LDA maximizes class separability at the expense of local data structure and does not capture non-linear relationships
- Relevant colinear features may not all be selected by HSS since keeping multiple colinear features may not improve separability
- HSS may disregard features only useful for separating a single class, and selected features require follow-up analysis for statistical significance
- Performance suffers with increasing number of populations and with high heterogeneity within a class (e.g., PBMCs labeled by time point)
- HSS is inappropriate when labels are unknown or an unbiased view is preferable

## Evidence

- [intro] HSS combines forward and reverse stepwise selection: "We implement hybrid subset selection (HSS), a heuristic approach using elements of both forward and reverse stepwise selection, to identify a set of features that enable enhanced separation"
- [methods] runHSS signature and downsample parameter: [evidence span withheld — anchor status: not_found]
- [methods] ElbowPlot for optimal feature set: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [methods] makeAxes for new data projection: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] HSS infeasible above 100 dimensions: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
- [discussion] Supervised approach inappropriate without labels: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
- [readme] README runHSS example invocation on TcellHartmann2020 data: "hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)"
- [readme] README elbow output element: "2.  ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
