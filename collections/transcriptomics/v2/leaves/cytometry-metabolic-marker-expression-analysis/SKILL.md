---
name: cytometry-metabolic-marker-expression-analysis
description: Use when comparing metabolic states of a test cell population against
  a reference population over experimental time points in single-cell cytometry data,
  using the TcellHartmann2020_sampleData-style marker set (GLUT1, HK2, GAPDH, LDHA,
  MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — this supervised
  approach is inappropriate in those situations
- Data has more than 100 dimensions — hybrid subset selection (HSS) is not feasible
  there, though plain LDA/sda can still be used
- Large heterogeneity exists within a class (e.g., PBMCs labeled by time point) or
  a very large number of populations must be separated — HSS-LDA performance suffers
tools:
- name: hsslda
  role: R package providing the sample dataset (TcellHartmann2020_sampleData) and
    makeAxes() to apply a trained LDA model's scaling coefficients to new data
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Runs LDA without HSS feature selection on the 15 metabolic markers grouped
    by day0–day5 labels; its $scaling feeds makeAxes()
- name: R
  role: Analysis environment in which the hsslda and MASS packages are executed
- name: ggplot2
  role: Visualizes mean LD coordinates per time point for CD57low and CD57high populations
    to assess manifold overlap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:f68522bdd83e56a635e49c92227e92dfd7bb2afab834a9f4cccbf233679eb57f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - ggplot2
  - hsslda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cytometry-metabolic-marker-expression-analysis

## Summary

Analyzes metabolic marker expression in T cells by training a linear discriminant analysis (LDA) model on CD57low (non-senescent) CyTOF cells across day0–day5 TCR-stimulation time points using the 15 metabolic marker channels (GLUT1 through HIF1A), then projecting unseen CD57high (senescent) cells onto those LD axes with hsslda::makeAxes() to reveal stunted metabolic progression.

## When to use

Use when comparing metabolic states of a test cell population against a reference population over experimental time points in single-cell cytometry data, using the TcellHartmann2020_sampleData-style marker set (GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A) and day labels.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — this supervised approach is inappropriate in those situations
- Data has more than 100 dimensions — hybrid subset selection (HSS) is not feasible there, though plain LDA/sda can still be used
- Large heterogeneity exists within a class (e.g., PBMCs labeled by time point) or a very large number of populations must be separated — HSS-LDA performance suffers

## Inputs

- TcellHartmann2020_sampleData (CyTOF single-cell matrix with 15 metabolic marker columns GLUT1..HIF1A plus a 'labels' column of day0–day5 classes)
- CD57low training-cell expression matrix with day labels (train.x, train.y)
- Unseen CD57high cell expression matrix over the same 15 markers
- MASS::lda model scaling coefficients (co) from the trained model

## Outputs

- LD-axis coordinates of the projected CD57high cells (saved to cd57high_ld_projection.csv)
- Mean LD coordinates per time point for CD57low and CD57high populations
- ggplot2 visualization of the two populations on LD axes (e.g., HSS_LD1 vs HSS_LD2 colored by labels)

## How to apply

Load the TcellHartmann2020_sampleData from the hsslda R package and subset the 15 metabolic marker channels plus day labels for the CD57low training cells. Train an LDA model on all 15 metabolic markers without feature selection using MASS::lda(), grouping by the day0–day5 class labels, since HSS feature selection is intentionally skipped here to keep all markers. Apply hsslda::makeAxes() with the trained model's scaling coefficients (lda.res$scaling) to project the unseen CD57high cells onto the LD axes. Compute mean LD coordinates per time point for both CD57low and CD57high populations and visualize them with ggplot2 to assess manifold overlap. Rationale: LDA identifies linear combinations of predictors that optimally separate a priori class labels (experimental time points), so axes trained on healthy/non-senescent cells provide a reference trajectory onto which senescent cells can be mapped.

## Related tools

- **hsslda** (R package providing the sample dataset (TcellHartmann2020_sampleData) and makeAxes() to apply a trained LDA model's scaling coefficients to new data) — https://github.com/mamouzgar/hsslda
- **MASS** (Runs LDA without HSS feature selection on the 15 metabolic markers grouped by day0–day5 labels; its $scaling feeds makeAxes())
- **R** (Analysis environment in which the hsslda and MASS packages are executed)
- **ggplot2** (Visualizes mean LD coordinates per time point for CD57low and CD57high populations to assess manifold overlap)

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- CD57high day-5 mean LD coordinates overlap the CD57low mean coordinates at approximately days 2–3 of TCR stimulation (stunted metabolic progression, matching Figures 2G–2K)
- The top three LD1-loading markers are significantly differentially expressed between CD57low and CD57high cells at all time points
- LD1 captures the dominant variance in the trained model (in the README example, Proportion of trace for LD1 ≈ 0.79)
- The projected output file cd57high_ld_projection.csv contains LD coordinates for all CD57high cells over the same LD axes as the CD57low training projection
- The LDA model reports equal prior probabilities across the six day0–day5 groups (0.1666667 each), confirming balanced class grouping during training

## Limitations

- LDA maximizes class separability at the expense of local data structure
- LDA does not capture non-linear relationships, which may limit performance on some datasets
- Features selected (or loading on axes) require follow-up analysis to establish the nature and statistical significance of differences

## Evidence

- [readme] run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [methods] makeAxes projection with trained model scaling: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] T cell metabolic regulome dataset demonstration: "on the T cell metabolic regulome dataset, in which the stunted metabolic states of CD57high T cells were visualized on LD axes trained on CD57low T cells"
- [abstract] LDA purpose for a priori labels: "LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity."
- [readme] ggplot2 visualization of LD axes: "ggplot2::ggplot(lda.df,ggplot2::aes(x = HSS_LD1, y = HSS_LD2, color = labels)) +"
