---
name: elbow-point-detection
description: Use when a runHSS() result (hss.result) contains an HSSscores table over
  subsetted feature-set sizes and you must pick the elbow-optimal number of markers
  to define the final HSS-LDA model.
when_to_use_negative:
- When class labels are unknown or an unbiased view of the data is preferable — HSS-LDA's
  supervised feature selection is inappropriate in these situations
- When the input has more than ~100 dimensions, where HSS is not feasible
- When the goal requires local data structure or non-linear relationships, which LDA
  maximization of class separability sacrifices
edam_operation: http://edamontology.org/operation_3094
tools:
- name: hsslda
  role: R package whose runHSS() returns the HSSscores table and ElbowPlot used to
    compute the elbow point and define the final marker set
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Renders the elbow plot (hss.result$ElbowPlot) for visual inspection and PNG
    export
- name: MASS
  role: Fits the final LDA (lda(y ~ ., data = x[, markers])) on the elbow-selected
    feature set
- name: R
  role: Environment in which the elbow-point computation and downstream subsetting
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:7020e09b47c6a2d010784770beb682ed0d5def93fb337e7b06e4e1ffa2dd65a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3094
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# elbow-point-detection

## Summary

Determines the optimal marker-set size for HSS-LDA feature selection by compiling the best separation score achieved at each feature-set size (from the HSSscores table) and computing the elbow point of that score curve, which fixes the final feature set used to fit the LDA model.

## When to use

Use when a runHSS() result (hss.result) contains an HSSscores table over subsetted feature-set sizes and you must pick the elbow-optimal number of markers to define the final HSS-LDA model.

## When NOT to use

- When class labels are unknown or an unbiased view of the data is preferable — HSS-LDA's supervised feature selection is inappropriate in these situations
- When the input has more than ~100 dimensions, where HSS is not feasible
- When the goal requires local data structure or non-linear relationships, which LDA maximization of class separability sacrifices

## Inputs

- HSSscores table of separation scores for each subsetted feature-set size (chained output of the HSS scoring filter step)
- hss.result object from hsslda::runHSS() containing the ElbowPlot element
- Input single-cell matrix (cells x markers, e.g., the 15-channel TcellHartmann2020_sampleData) for subsetting to elbow-optimal markers

## Outputs

- Computed elbow point (optimal feature-set size, e.g., 10 of 15 markers)
- ElbowPlot visualization (ggplot2 object, exportable as PNG)
- Elbow-optimal marker vector used to fit the final LDA model

## How to apply

Load the HSSscores subset-score table, which is the chained output of the upstream hybrid subset selection (HSS) step that iteratively adds and removes markers (e.g., from 2 up to 15 markers using the euclidean, silhouette, pixel.entropy, or pixel.density scoring metric). Compile the best score for each feature-set size and compute the elbow point on the score-versus-size curve, which is visualized as hss.result$ElbowPlot (a ggplot2-configurable object). The elbow marks where additional markers stop improving the separation metric, so the feature set up to the elbow becomes the final marker set. Subset the input single-cell matrix to these elbow-optimal markers and fit the final LDA via MASS::lda(y ~ ., data = x[, markers]) in the hsslda runHSS downstream model-fitting routine. In the vignette (TcellHartmann2020_sampleData with 15 metabolic markers and 6 day0–day5 labels), the elbow point selected 10 of 15 markers, yielding LD1–LD5 with LD1 explaining 0.7906 of the trace.

## Related tools

- **hsslda** (R package whose runHSS() returns the HSSscores table and ElbowPlot used to compute the elbow point and define the final marker set) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Renders the elbow plot (hss.result$ElbowPlot) for visual inspection and PNG export)
- **MASS** (Fits the final LDA (lda(y ~ ., data = x[, markers])) on the elbow-selected feature set)
- **R** (Environment in which the elbow-point computation and downstream subsetting are executed)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- The fitted model object (hss.result$`HSS-LDA-model`) exists and its output table contains all input markers, class labels, and the LD axes
- The selected feature count in the final model matches the computed elbow point (10 of 15 markers in the vignette)
- LD1 explains 0.7906 of the trace in the vignette model, with LD1–LD5 trace proportions 0.7906/0.1890/0.0115/0.0066/0.0023
- The elbow plot object is ggplot2-configurable and the marker-size trajectory in HSSscores covers forward and reverse stepwise moves (e.g., sizes oscillating 9–15 in the README log)
- The final HSS-LDA-result dataframe merges markers, labels, and LD1–LD5 axes for all input cells

## Limitations

- Relevant colinear features may not all be selected by HSS, since keeping multiple colinear features may not improve model separability
- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class
- Performance suffers with an increasing number of populations and when a large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point)

## Evidence

- [readme] ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set.: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [readme] HSSscores: a table of all scores for each subsetted model.: "HSSscores: a table of all scores for each subsetted model."
- [abstract] We implement feature selection by hybrid subset selection (HSS) and demonstrate that this computationally efficient approach generates non-stochastic, interpretable axes: "We implement feature selection by hybrid subset selection (HSS) and demonstrate that this computationally efficient approach generates non-stochastic, interpretable axes"
- [readme] Note that scoring metrics like silhouette or pixel class entropy (PCE) score will require additional package dependencies.: "Note that scoring metrics like silhouette or pixel class entropy (PCE) score will require additional package dependencies."
