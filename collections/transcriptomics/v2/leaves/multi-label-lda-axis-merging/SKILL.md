---
name: multi-label-lda-axis-merging
description: Use when cells have two or more a priori labels capturing distinct biological
  variance (e.g., cell-cycle phase and cell line) and you want a single supervised-then-unsupervised
  embedding that preserves all label relationships.
when_to_use_negative:
- Labels are unknown or an unbiased view of the data is preferable — this supervised
  approach is inappropriate in those situations
- A label has a large degree of heterogeneity within a class or many populations,
  where HSS-LDA performance suffers
- More than 100 dimensions, where HSS feature selection is not feasible
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: hsslda
  role: R package implementing runHSS (hybrid subset selection + LDA) and makeAxes
    for per-label supervised dimensionality reduction
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Alternative for running LDA without HSS, with makeAxes to generate LD axes
- name: dplyr
  role: Merging the HSS-LD columns from the separate label-specific runs into a single
    feature dataframe
- name: ggplot2
  role: Rendering the biaxial plot on HSS_LD1 vs HSS_LD2 colored by label
- name: R
  role: Environment in which runHSS, makeAxes, UMAP embedding, and plotting are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:1984dbc6ecd1024821d507a4d335cc83577073da1d26d04e61c235fdd05bb8c5
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
yes_rate: 0.6428571428571429
n_claims: 14
---

# multi-label-lda-axis-merging

## Summary

Runs separate HSS-LDA analyses on multiple class labels (e.g., cell-cycle phase and cell line in CyTOF data), then merges the resulting HSS-LD axis projections into a single feature set for unsupervised embedding (UMAP), capturing several known sources of variance in one plot.

## When to use

Use when cells have two or more a priori labels capturing distinct biological variance (e.g., cell-cycle phase and cell line) and you want a single supervised-then-unsupervised embedding that preserves all label relationships.

## When NOT to use

- Labels are unknown or an unbiased view of the data is preferable — this supervised approach is inappropriate in those situations
- A label has a large degree of heterogeneity within a class or many populations, where HSS-LDA performance suffers
- More than 100 dimensions, where HSS feature selection is not feasible

## Inputs

- CyTOF or other single-cell expression matrix (cells as rows, markers as columns) in R, e.g. following the TcellHartmann2020_sampleData x/y layout
- Two or more label vectors (e.g., cell-cycle phase such as G0-G2, and cell line) matching the matrix rows

## Outputs

- One HSS-LDA model and ElbowPlot per label from runHSS
- Merged dataframe of HSS-LD axis projections from all label-specific runs (combined with dplyr)
- UMAP embedding of the merged HSS-LD features, e.g. a Figure 3H-style biaxial plot (figure3h_biaxial.png) via ggplot2

## How to apply

For each label vector, call hsslda::runHSS(x = expression matrix, y = label, score.method = 'euclidean') on the same CyTOF marker-by-cell matrix; the hybrid subset selection (forward and reverse stepwise) optimizes the feature set, and the ElbowPlot identifies the optimal feature count for each run. Project cells onto each model's LD axes with makeAxes(df = data, co = model$scaling), then use dplyr to merge the HSS-LD columns from both runs into one combined feature dataframe. Embed the merged HSS-LD feature set with UMAP and render a biaxial plot with ggplot2 (HSS_LD1 vs HSS_LD2, colored by label). Rationale: each HSS-LDA run maximizes separation for one label, so concatenating their LD axes lets an unsupervised method like UMAP capture both sources of known variance simultaneously, as in the Chromotyping Figure 3H biaxial plot.

## Related tools

- **hsslda** (R package implementing runHSS (hybrid subset selection + LDA) and makeAxes for per-label supervised dimensionality reduction) — https://github.com/mamouzgar/hsslda
- **MASS** (Alternative for running LDA without HSS, with makeAxes to generate LD axes)
- **dplyr** (Merging the HSS-LD columns from the separate label-specific runs into a single feature dataframe)
- **ggplot2** (Rendering the biaxial plot on HSS_LD1 vs HSS_LD2 colored by label)
- **R** (Environment in which runHSS, makeAxes, UMAP embedding, and plotting are executed)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE); lda.df.newData = makeAxes(df = newdata, co = hss.result$`HSS-LDA-model`$scaling)
```

## Evaluation signals

- Each runHSS ElbowPlot shows a clear elbow point defining the optimal feature set for its label
- makeAxes output contains HSS-LD columns for every input cell (no rows dropped when downsample = FALSE)
- Merged feature dataframe has the union of LD columns from both runs with row counts matching the original matrix
- On the rendered biaxial plot, cells cluster separately by cell line while a G0-to-G2 cell-cycle trajectory is preserved within each cell line (expert visual review, as in Figure 3H)

## Limitations

- HSS may disregard features only useful for separating a single class, since it enriches for features separating multiple classes
- Relevant colinear features may not all be selected by HSS, as keeping multiple colinear features may not improve model separability
- HSS-LDA may not perform well when a large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) and performance suffers with an increasing number of populations
- HSS is not feasible for more than 100 dimensions; LDA maximizes class separability at the expense of local data structure and does not capture non-linear relationships

## Evidence

- [other] Combining HSS-LDs from separate HSS-LDA analyses of cell line and cell cycle as a feature set input to UMAP yields a biaxial plot that preserves both cell line and cell-cycle relationships: [evidence span withheld — anchor status: not_found]
- [methods] You can use this final model to apply the same dimensionality reduction to new data using makeAxes.: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [readme] You can run hybrid subset selection on a dataset using runHSS().: "You can run hybrid subset selection on a dataset using runHSS()."
- [methods] ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set.: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [discussion] This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable.: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
