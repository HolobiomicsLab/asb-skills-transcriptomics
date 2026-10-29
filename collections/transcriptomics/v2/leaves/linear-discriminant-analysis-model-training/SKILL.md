---
name: linear-discriminant-analysis-model-training
description: Use when you have single-cell measurements (e.g., CyTOF metabolic markers
  such as GLUT1 through HIF1A) plus a priori class labels and want supervised axes
  that optimally separate those labels for visualization or projection of unseen cells.
when_to_use_negative:
- Class labels are unknown, or an unbiased view of the data is preferable — the authors
  note this supervised approach is inappropriate in those situations.
- Dataset has more than 100 dimensions — HSS-based feature selection is not feasible
  there (plain LDA/sda still provide useful plots).
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled
  by time point) or the number of populations is large — performance suffers.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MASS
  role: R package used to fit the LDA model (MASS::lda()) when running LDA without
    HSS feature selection; its $scaling coefficients feed makeAxes
  repo: github.com/mamouzgar/hsslda
- name: hsslda
  role: R package wrapping LDA with Hybrid Subset Selection feature selection; provides
    runHSS(), makeAxes(), ElbowPlot, and TcellHartmann2020_sampleData
  repo: github.com/mamouzgar/hsslda
- name: ggplot2
  role: Visualization of cells on LD axes (e.g., HSS_LD1 vs HSS_LD2 colored by labels)
    and mean LD coordinates per time point
- name: R
  role: Language/environment in which the LDA model is trained and applied
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:6b1817edcadc881f5113a8fb94db9ff543f16e7c2bdfc51d2b2af3b939549480
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MASS
  - R
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# linear-discriminant-analysis-model-training

## Summary

Trains a linear discriminant analysis (LDA) model (e.g., via MASS::lda() in R) on single-cell data with a priori class labels (such as day0-day5 TCR stimulation time points) and repurposes the fitted model's scaling coefficients as supervised LD axes for dimensionality reduction of new data.

## When to use

Use when you have single-cell measurements (e.g., CyTOF metabolic markers such as GLUT1 through HIF1A) plus a priori class labels and want supervised axes that optimally separate those labels for visualization or projection of unseen cells.

## When NOT to use

- Class labels are unknown, or an unbiased view of the data is preferable — the authors note this supervised approach is inappropriate in those situations.
- Dataset has more than 100 dimensions — HSS-based feature selection is not feasible there (plain LDA/sda still provide useful plots).
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) or the number of populations is large — performance suffers.

## Inputs

- cells (rows) x markers/genes (columns) dataframe, e.g., 15 metabolic marker channels from TcellHartmann2020_sampleData (GLUT1 through HIF1A)
- vector of a priori class labels matching training rows, e.g., day0-day5 TCR stimulation time points
- optionally, a dataframe of new/unseen cells to project (e.g., CD57high cells) via makeAxes

## Outputs

- fitted LDA model object with 'Coefficients of linear discriminants' (scaling matrix), 'Prior probabilities of groups', 'Group means', and 'Proportion of trace'
- projected LD-axis coordinate table (e.g., LD1-LD5 or HSS_LD1/HSS_LD2 columns) for training and new data
- mean LD coordinates per class/time point for visualization with ggplot2

## How to apply

Assemble a cells-by-features dataframe (e.g., the 15 metabolic marker channels GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A) and a matching vector of class labels (e.g., day0-day5). Fit the model with MASS::lda(x = train.x, grouping = train.y), either directly (LDA without HSS) or on the optimal feature set identified by hsslda::runHSS() with a scoring metric such as 'euclidean', 'silhouette', 'pixel.entropy', or 'pixel.density'. The fitted model object contains 'Coefficients of linear discriminants' (scaling) and 'Proportion of trace' values; pass the scaling matrix to makeAxes(df, co) to project new/unseen data onto the same LD axes. Rationale: LDA maximizes class separability and minimizes within-class variance, producing non-stochastic, interpretable axes; LD1 typically carries most variance (proportion of trace ~0.79 in the Hartmann T cell metabolic regulome example), so top LD1 loadings identify key feature drivers such as differentially expressed metabolic markers.

## Related tools

- **MASS** (R package used to fit the LDA model (MASS::lda()) when running LDA without HSS feature selection; its $scaling coefficients feed makeAxes) — github.com/mamouzgar/hsslda
- **hsslda** (R package wrapping LDA with Hybrid Subset Selection feature selection; provides runHSS(), makeAxes(), ElbowPlot, and TcellHartmann2020_sampleData) — github.com/mamouzgar/hsslda
- **ggplot2** (Visualization of cells on LD axes (e.g., HSS_LD1 vs HSS_LD2 colored by labels) and mean LD coordinates per time point)
- **R** (Language/environment in which the LDA model is trained and applied)

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- The model summary reports 'Coefficients of linear discriminants' for each selected marker and 'Proportion of trace' values; in the README T cell example LD1 carries ~0.7906 of the trace, so LD1 loadings should dominate downstream interpretation.
- makeAxes() applied to new data with the trained model's $scaling must produce LD coordinate columns on the same axes as the training projections (e.g., LD1-LD5).
- Group means and prior probabilities (equal 0.1666667 for day0-day5 in the example) are printed by the fitted lda model, confirming balanced class labels were used.
- For the CD57high projection use case, validation is that CD57high day-5 mean LD coordinates overlap the CD57low mean coordinates at approximately days 2-3, matching Figures 2G-2K.
- Top LD1-loading markers should be verifiable as differentially expressed between labeled classes with follow-up statistical testing (HSS-selected features require such follow-up for significance).

## Limitations

- LDA maximizes class separability at the expense of local data structure, and it does not capture non-linear relationships.
- HSS feature selection may not retain all relevant colinear features and may disregard features useful for separating only a single class.
- Features selected by HSS are differentially expressed by one or more labeled classes, but follow-up analysis is needed to establish the nature and statistical significance of the differences.
- HSS-LDA may perform poorly with high heterogeneity within a class or an increasing number of populations.

## Evidence

- [abstract] LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity.: "LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity."
- [readme] To run LDA without HSS, use MASS::lda then makeAxes: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes"
- [methods] You can use this final model to apply the same dimensionality reduction to new data using makeAxes.: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [readme] ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set.: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [discussion] LDA, by definition, maximizes class separability and minimizes variance within each projected class, and by doing so will come at the expense of local data structure.: "LDA, by definition, maximizes class separability and minimizes variance within each projected class, and by doing so will come at the expense of local data structure."
