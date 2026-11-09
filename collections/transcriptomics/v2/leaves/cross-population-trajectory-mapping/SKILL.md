---
name: cross-population-trajectory-mapping
description: Use when comparing the state/progression of an unseen cell population
  against a labeled reference population along shared axes — e.g., asking whether
  CD57high senescent T cells overlap CD57low cells at days 2–3 of TCR stimulation
  on LD axes.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — the supervised
  approach is inappropriate in these situations per the article's discussion.
- More than ~100 features must be considered, since HSS feature selection is not feasible
  at that dimensionality (plain LDA without HSS may still be used).
- The target population requires preserving local data structure or capturing non-linear
  relationships — LDA maximizes class separability at the expense of local structure
  and does not capture non-linear relationships.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: MASS
  role: Fits the LDA model on the reference population's metabolic markers without
    feature selection; its $scaling coefficients define the LD axes
- name: hsslda
  role: Provides makeAxes() to project the unseen population onto the trained LD axes
    and the TcellHartmann2020_sampleData example dataset
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Visualizes the projected LD coordinates of both populations, colored by class
    labels, to assess trajectory overlap
- name: R
  role: Language environment in which the training, projection (makeAxes), and visualization
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:c03f5e7ddcc067d062961e55db136db3b8cdac73061b10ed60ff42356c87011e
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-population-trajectory-mapping

## Summary

Projects an unseen cell population onto linear discriminant (LD) axes trained on a reference population, so that two biological trajectories (e.g., senescent CD57high vs non-senescent CD57low T cells over TCR stimulation days 0–5) can be compared on a shared manifold. Grounded in the HSS-LDA makeAxes workflow of Amouzgar et al. 2022 (Patterns).

## When to use

Use when comparing the state/progression of an unseen cell population against a labeled reference population along shared axes — e.g., asking whether CD57high senescent T cells overlap CD57low cells at days 2–3 of TCR stimulation on LD axes.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — the supervised approach is inappropriate in these situations per the article's discussion.
- More than ~100 features must be considered, since HSS feature selection is not feasible at that dimensionality (plain LDA without HSS may still be used).
- The target population requires preserving local data structure or capturing non-linear relationships — LDA maximizes class separability at the expense of local structure and does not capture non-linear relationships.

## Inputs

- Reference-population feature table: single-cell mass cytometry (CyTOF) intensities for 15 metabolic markers (GLUT1 through HIF1A) with day0–day5 labels, e.g., TcellHartmann2020_sampleData from the hsslda package
- Unseen target-population feature table with the same marker columns but no requirement for training labels
- LDA model scaling coefficients (model$scaling from MASS::lda or hss.result$`HSS-LDA-model`$scaling)

## Outputs

- Projected LD-axis coordinates (e.g., LD1/LD2 or HSS_LD1/HSS_LD2) for the target population, e.g., cd57high_ld_projection.csv
- Per-time-point mean LD coordinates for both populations (trajectory summary table)
- ggplot2 trajectory-overlap plot showing where the target population's mean coordinates fall on the reference manifold (cf. Figures 2G–2K)

## How to apply

Train an LDA model on the reference population using the 15 metabolic marker channels (GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A) with day0–day5 class labels; run without feature selection via MASS::lda() when all markers should be retained, or via hsslda::runHSS() with score.method='euclidean' and downsample=FALSE when an optimal feature subset is desired. Extract the model's scaling coefficients (lda.res$scaling) and apply them to the unseen population with hsslda::makeAxes(df = newdata, co = model$scaling), which projects new cells onto the same LD axes without retraining. Compute mean LD coordinates per time point for both populations and visualize with ggplot2 (e.g., HSS_LD1 vs HSS_LD2 colored by labels) to assess trajectory overlap. Rationale: LDA identifies linear combinations of predictors that optimally separate a priori classes, so axes trained on the reference encode its differentiation manifold, onto which the second population can be mapped for direct comparison of stunted versus full metabolic progression.

## Related tools

- **MASS** (Fits the LDA model on the reference population's metabolic markers without feature selection; its $scaling coefficients define the LD axes)
- **hsslda** (Provides makeAxes() to project the unseen population onto the trained LD axes and the TcellHartmann2020_sampleData example dataset) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Visualizes the projected LD coordinates of both populations, colored by class labels, to assess trajectory overlap)
- **R** (Language environment in which the training, projection (makeAxes), and visualization are executed)

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df.newData = makeAxes(df = newdata, co = lda.res$scaling)
```

## Evaluation signals

- The CD57high day-5 mean LD coordinates overlap the CD57low mean coordinates at approximately days 2–3 of TCR stimulation, matching Figures 2G–2K.
- The top three features contributing to HSS-LD1 are significantly differentially expressed between CD57low and CD57high cells at all time points.
- The projection uses the trained model's scaling matrix directly (co = model$scaling); no re-fitting on the target population occurs — verify makeAxes was called with the training model's coefficients.
- The LDA model summary shows equal prior probabilities (~0.1667 for each of the six day classes) and an LD1 proportion of trace around 0.79, consistent with the README's trained model.
- The output table contains the same marker columns as the input plus new LD columns (LD1–LD5 or HSS_LD1/HSS_LD2) and labels.

## Limitations

- Relevant colinear features may not all be selected by HSS, since keeping multiple colinear features may not improve separability.
- HSS may disregard features only useful for separating a single class, and features selected by HSS require follow-up analysis for the nature and statistical significance of differences.
- LDA does not capture non-linear relationships and comes at the expense of local data structure; performance also suffers with an increasing number of populations or high heterogeneity within a class.
- HSS-LDA is inappropriate when labels are unknown or an unbiased view of the data is preferable.

## Evidence

- [methods] You can use this final model to apply the same dimensionality reduction to new data using makeAxes.: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] on the T cell metabolic regulome dataset, in which the stunted metabolic states of CD57high T cells were visualized on LD axes trained on CD57low T cells: "on the T cell metabolic regulome dataset, in which the stunted metabolic states of CD57high T cells were visualized on LD axes trained on CD57low T cells"
- [readme] To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset.: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [abstract] LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity.: "LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity."
- [discussion] This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable.: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
