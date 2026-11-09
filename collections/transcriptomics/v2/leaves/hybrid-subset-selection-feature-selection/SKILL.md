---
name: hybrid-subset-selection-feature-selection
description: Use when high-dimensional single-cell data (e.g., CyTOF cells-by-markers
  tables) has a priori class labels and you need a reduced, interpretable feature
  set for supervised dimensionality reduction with LDA.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — this is
  a supervised approach and is inappropriate in those situations
- More than 100 dimensions, where HSS is not feasible; plain LDA or sda still provide
  useful plots
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled
  by time point) or many populations must be separated
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0092
tools:
- name: hsslda
  role: R package implementing runHSS (HSS feature selection around LDA) and makeAxes
    projection
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Fits plain LDA baseline (MASS::lda) on full marker sets for comparison
- name: ggplot2
  role: Plots HSS_LD1/HSS_LD2 axes colored by class labels and renders the ElbowPlot
- name: R
  role: Runtime environment for the hsslda package and its dependencies (dplyr, tidyr,
    stats)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:4bae3c2093b0db830d995f3745c7a642f819718cdad32d83ec700aa24c5e047e
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# hybrid-subset-selection-feature-selection

## Summary

Feature selection via Hybrid Subset Selection (HSS), a heuristic combining forward and reverse stepwise selection around LDA, implemented in the hsslda R package to reduce input marker dimensionality for supervised single-cell visualization while retaining or improving class separation.

## When to use

Use when high-dimensional single-cell data (e.g., CyTOF cells-by-markers tables) has a priori class labels and you need a reduced, interpretable feature set for supervised dimensionality reduction with LDA.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — this is a supervised approach and is inappropriate in those situations
- More than 100 dimensions, where HSS is not feasible; plain LDA or sda still provide useful plots
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) or many populations must be separated

## Inputs

- Cells-by-markers dataframe (x), e.g., CyTOF channels such as GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A
- Vector of a priori class labels (y) matching training data rows, e.g., 'labels' column with day0–day5
- Optional custom separation-metric function (custom.score.method) taking x, y, and cols

## Outputs

- HSSscores table of scores for each subsetted model
- ElbowPlot (ggplot object) with the calculated elbow point for the optimal feature set
- Final HSS-LDA-model (lda object with coefficients of linear discriminants and scaling)
- HSS-LDA-result dataframe merging markers, class labels, and HSS_LD1/HSS_LD2 linear discriminants
- downsampled.cells.included.in.analysis boolean vector (all TRUE when downsample = FALSE)

## How to apply

Load the dataset as a cells-by-markers dataframe (x) plus an a priori class-label vector (y), e.g., TcellHartmann2020_sampleData channels with 'labels'. Call hsslda::runHSS(x, y, score.method = 'euclidean', downsample = FALSE) — the default scoring metric is euclidean distance; alternatives include 'silhouette', 'pixel.entropy', 'pixel.density', or a user function via score.method = 'custom'. HSS iterates forward and reverse stepwise selection, logging 'Number of markers' at each subset size. Inspect the returned HSSscores table and ElbowPlot object, whose calculated elbow point defines the optimal feature set (e.g., reported retention of 13 of 17 features for Morphometry, 8 of 32 for the T cell metabolic regulome, 24 of 32 for Chromotyping). Extract the final HSS-LDA-model (usable like any R model via MASS::lda output) and HSS-LDA-result table, project new data with makeAxes(df, co) using the model scaling coefficients, and plot HSS_LD1/HSS_LD2 with ggplot2 colored by labels to compare against plain MASS::lda on the full marker set.

## Related tools

- **hsslda** (R package implementing runHSS (HSS feature selection around LDA) and makeAxes projection) — https://github.com/mamouzgar/hsslda
- **MASS** (Fits plain LDA baseline (MASS::lda) on full marker sets for comparison)
- **ggplot2** (Plots HSS_LD1/HSS_LD2 axes colored by class labels and renders the ElbowPlot)
- **R** (Runtime environment for the hsslda package and its dependencies (dplyr, tidyr, stats))

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE); lda.df = makeAxes(df = newdata, co = hss.result$`HSS-LDA-model`$scaling)
```

## Evaluation signals

- Retained-marker counts at the elbow point match the reported reduced sets: 13 of 17 features (Morphometry), 8 of 32 (T cell metabolic regulome), 24 of 32 (Chromotyping)
- HSS-LDA visualization qualitatively retains or improves class-label separation versus plain LDA on the full marker set (Figures S1B–S1G comparison)
- ElbowPlot shows a clearly calculated elbow point defining the optimal feature set
- Proportion of trace in the final lda model is dominated by LD1/LD2 (e.g., 0.7906 and 0.1890 for the T cell metabolic regulome example)
- With downsample = FALSE, downsampled.cells.included.in.analysis values are all TRUE

## Limitations

- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class
- Relevant colinear features may not all be selected, as separability may not be improved by keeping multiple colinear features
- LDA maximizes class separability at the expense of local data structure and does not capture non-linear relationships
- Features selected by HSS are differentially expressed by one or more labeled classes and require follow-up analysis for the nature and statistical significance of differences
- Performance suffers with an increasing number of populations, and the hsslda package is still under development

## Evidence

- [results] HSS-LDA visualizations either qualitatively improved or retained the original class-label separation: [evidence span withheld — anchor status: not_found]
- [methods] runHSS signature: "runHSS(x, y, score.method, custom.score.method, downsample)"
- [methods] ElbowPlot definition: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [readme] Downsampling parameter: "(optional) downsample: Boolean, defaults to TRUE. Downsamples data to improve runtime. Set to FALSE to use all input data."
- [intro] HSS heuristic definition: "We implement hybrid subset selection (HSS), a heuristic approach using elements of both forward and reverse stepwise selection, to identify a set of features that enable enhanced separation of these"
- [discussion] Supervised-only caveat: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
- [discussion] >100 dimensions caveat: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
