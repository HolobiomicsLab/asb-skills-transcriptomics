---
name: linear-discriminant-visualization-ggplot2
description: Use when a fitted LDA or HSS-LDA model on cells-by-markers CyTOF data
  must be rendered as a 2D scatter plot of LD axes colored by class labels, or when
  HSS-LDA separation must be visually compared against a plain-LDA baseline.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — the supervised
  LD axes are inappropriate in these situations
- Preservation of local data structure is required — LDA maximizes class separability
  at the expense of local structure
- Relationships are strongly non-linear or the dataset has more than ~100 dimensions
  — LDA does not capture non-linear relationships and HSS is not feasible above 100
  dimensions
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ggplot2
  role: Renders the LD-axis scatter plot colored by class labels (geom_point on HSS_LD1/HSS_LD2
    aesthetics); also used to configure the returned ElbowPlot
- name: hsslda
  role: Provides runHSS (feature selection + LDA fit) whose HSS-LDA-result dataframe
    supplies the HSS_LD1/HSS_LD2 axes to plot, and makeAxes to project new data
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Fits the plain-LDA baseline (MASS::lda) whose scaling coefficients feed makeAxes
    to generate LD axes for the comparison projection
- name: CyTOF
  role: 'Source data modality: mass cytometry cells-by-markers datasets (Morphometry,
    T cell metabolic regulome, Chromotyping) whose class separation is visualized
    on LD axes'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:64ddcbd9098c74ec756be7c8774e2ca7e4cfd0166804a3844109c3e9c30da81d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - CyTOF
  - MASS
  - R
  - dplyr
  - ggplot2
  - hsslda
  - stats
  - tidyr
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# linear-discriminant-visualization-ggplot2

## Summary

Visualizes supervised linear-discriminant (LD) projections of single-cell CyTOF data by plotting HSS_LD1/HSS_LD2 (or makeAxes-derived LD axes) as ggplot2 scatter plots colored by a priori class labels, enabling qualitative comparison of class separation between HSS-LDA and plain MASS::lda models.

## When to use

Use when a fitted LDA or HSS-LDA model on cells-by-markers CyTOF data must be rendered as a 2D scatter plot of LD axes colored by class labels, or when HSS-LDA separation must be visually compared against a plain-LDA baseline.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — the supervised LD axes are inappropriate in these situations
- Preservation of local data structure is required — LDA maximizes class separability at the expense of local structure
- Relationships are strongly non-linear or the dataset has more than ~100 dimensions — LDA does not capture non-linear relationships and HSS is not feasible above 100 dimensions

## Inputs

- HSS-LDA-result dataframe from hsslda::runHSS containing markers, class labels, and linear discriminants (HSS_LD1, HSS_LD2, ...)
- cells-by-markers dataframe projected onto LD axes via makeAxes(df, co = lda.res$scaling) for plain MASS::lda
- a priori class-label vector (e.g., 'labels' column with day0–day5 values)

## Outputs

- ggplot2 scatter plot of HSS_LD1 vs HSS_LD2 (or LD1/LD2) colored by class labels
- PNG files of elbow plots and LD scatter plots
- hss_lda_feature_sets.csv with per-dataset selected feature lists, subset scores, and retained-feature counts

## How to apply

Fit the discriminant model first: either hsslda::runHSS(x, y, score.method = 'euclidean', downsample = FALSE), whose HSS-LDA-result element is a merged dataframe of markers, labels, and the LD axes, or MASS::lda on the full marker set followed by makeAxes(df, co = lda.res$scaling) to project data onto LD axes. Then plot with ggplot2::ggplot(lda.df, ggplot2::aes(x = HSS_LD1, y = HSS_LD2, color = labels)) + ggplot2::geom_point(), where the class-label column is the a priori label (e.g., day0–day5 time points in TcellHartmann2020_sampleData). Judge the result by comparing the colored cluster separation of HSS-LDA axes against the plain-LDA projection (as in Figures S1B–S1G): HSS-LDA should qualitatively retain or improve separation despite the reduced feature set (e.g., 8 of 32 markers for the T cell metabolic regulome). Note that the ElbowPlot object returned by runHSS is itself ggplot-configurable, and outputs such as selected feature lists and elbow plots can be written to CSV/PNG files.

## Related tools

- **ggplot2** (Renders the LD-axis scatter plot colored by class labels (geom_point on HSS_LD1/HSS_LD2 aesthetics); also used to configure the returned ElbowPlot)
- **hsslda** (Provides runHSS (feature selection + LDA fit) whose HSS-LDA-result dataframe supplies the HSS_LD1/HSS_LD2 axes to plot, and makeAxes to project new data) — https://github.com/mamouzgar/hsslda
- **MASS** (Fits the plain-LDA baseline (MASS::lda) whose scaling coefficients feed makeAxes to generate LD axes for the comparison projection)
- **CyTOF** (Source data modality: mass cytometry cells-by-markers datasets (Morphometry, T cell metabolic regulome, Chromotyping) whose class separation is visualized on LD axes)

## Examples

```
ggplot2::ggplot(lda.df, ggplot2::aes(x = HSS_LD1, y = HSS_LD2, color = labels)) + ggplot2::geom_point()
```

## Evaluation signals

- The plotted lda.df contains the expected LD-axis columns: HSS.result$`HSS-LDA-result` includes all input markers, the 'labels' column, and LD1–LD5 discriminants (proportion of trace ~0.79/0.19 for LD1/LD2 in the README example)
- Retained-marker counts from the elbow point match the reported reduced sets: 13 of 17 (Morphometry), 8 of 32 (T cell metabolic regulome), 24 of 32 (Chromotyping)
- Visual check: HSS-LDA colored scatter qualitatively retains or improves class separation relative to the plain MASS::lda projection on the same dataset
- The ggplot call executes without error because the required HSS_LD1/HSS_LD2 columns exist in the dataframe passed to ggplot2::aes()
- For projections onto new data, makeAxes(df = newdata, co = hss.result$`HSS-LDA-model`$scaling) produces LD columns consistent with the training model's scaling matrix

## Limitations

- LDA, by definition, maximizes class separability and comes at the expense of local data structure, and can manifest some degree of crowding
- HSS-LDA may not perform well when a large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) and performance will suffer with an increasing number of populations
- Relevant colinear features may not all be selected by HSS, and HSS may disregard features only useful for separating a single class

## Evidence

- [methods] ggplot2 scatter plot colored by labels: "ggplot2::ggplot(lda.df,ggplot2::aes(x = HSS_LD1, y = HSS_LD2, color = labels)) +"
- [readme] ElbowPlot visualization: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [methods] makeAxes projection: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [readme] HSS-LDA result table contents: "HSS-LDA-result: The final result table containing all initially inputted markers, class labels, and linear discriminants from the opimal model."
- [discussion] LDA expense of local structure: "LDA, by definition, maximizes class separability and minimizes variance within each projected class, and by doing so will come at the expense of local data structure."
