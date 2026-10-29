---
name: cross-validated-classification-accuracy-evaluation
description: Use when a trained LDA or HSS-LDA model with a priori labels (e.g., cell-cycle
  phase, day, cell type) must be validated by cross-validated classification accuracy
  and analysis of which classes are confused.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — the article
  states this supervised approach is inappropriate in those situations
- The goal is to preserve local data structure in the embedding rather than maximize
  class separability, since LDA maximizes class separability at the expense of local
  structure
- The dataset has a large number of populations or high heterogeneity within a class
  (e.g., PBMCs labeled by time point), where LDA performance suffers
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: hsslda
  role: R package implementing LDA with hybrid subset selection; its runHSS() output
    includes the final HSS-LDA-model that can be used like any R model for cross-validated
    prediction of class labels
  repo: mamouzgar/hsslda
- name: MASS
  role: Provides MASS::lda() to run LDA without HSS, whose fitted model is then used
    for cross-validated classification of cells into the labeled groups
- name: R
  role: Execution environment for the LDA training, cross-validated prediction, and
    export of accuracy metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:07ff12fe58390b24328e08e4c4d7712000c04096d22bd9f749a4111fd348e1aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - dplyr
  - hsslda
  - stats
  - tidyr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-validated-classification-accuracy-evaluation

## Summary

Quantifies how well an LDA/HSS-LDA supervised embedding separates a priori class labels by performing cross-validated classification of cells and reporting overall accuracy plus the adjacency structure of misclassifications. This turns the linear discriminant axes from a visualization into a verifiable metric of class separation.

## When to use

Use when a trained LDA or HSS-LDA model with a priori labels (e.g., cell-cycle phase, day, cell type) must be validated by cross-validated classification accuracy and analysis of which classes are confused.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — the article states this supervised approach is inappropriate in those situations
- The goal is to preserve local data structure in the embedding rather than maximize class separability, since LDA maximizes class separability at the expense of local structure
- The dataset has a large number of populations or high heterogeneity within a class (e.g., PBMCs labeled by time point), where LDA performance suffers

## Inputs

- Cells x features matrix with numeric predictors (e.g., per-cell module scores for G1.S, S, G2, G2.M, M.G1 gene sets, or TcellHartmann2020_sampleData marker columns)
- Vector of a priori class labels matching the training rows (e.g., cell-cycle phase, or the 'labels' column with day0-day5)

## Outputs

- Overall cross-validated classification accuracy (e.g., ~88% for the five-phase cell-cycle task)
- Confusion structure / fraction of misclassifications occurring between adjacent classes (e.g., >99% between adjacent cell-cycle phases)
- Per-cell predicted class labels and per-cell LD coordinates saved to a results table (e.g., cell_cycle_lda_results.csv)

## How to apply

Train the LDA model on the labeled feature matrix (e.g., a 5-column matrix of G1.S, S, G2, G2.M, M.G1 cell-cycle scores with phase as the grouping factor) using MASS::lda() or the hsslda pipeline, then perform cross-validated prediction of the class labels rather than only projecting cells onto LD1/LD2. Report the overall cross-validated accuracy — for the Hartmann 2020 T cell cell-cycle application this was approximately 88% — and, critically, tabulate the classes involved in each misclassification: in the reference workflow more than 99% of errors fell between adjacent cell-cycle phases. This adjacency check is the key biological validation, because high accuracy alone does not confirm that the LD embedding respects the circular ordering of classes; errors concentrated between neighbors (rather than between distant classes) demonstrate that the discriminant space encodes the expected phase ordering. Save the per-cell predicted labels and the confusion/error-adjacency summary alongside the LD coordinates (e.g., in cell_cycle_lda_results.csv).

## Related tools

- **hsslda** (R package implementing LDA with hybrid subset selection; its runHSS() output includes the final HSS-LDA-model that can be used like any R model for cross-validated prediction of class labels) — mamouzgar/hsslda
- **MASS** (Provides MASS::lda() to run LDA without HSS, whose fitted model is then used for cross-validated classification of cells into the labeled groups)
- **R** (Execution environment for the LDA training, cross-validated prediction, and export of accuracy metrics)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- Cross-validated phase-classification accuracy approximates the reported ~88% for the five cell-cycle phases
- More than 99% of classification errors fall between adjacent cell-cycle phases, confirming the circular ordering of phases on the LD embedding
- The fitted model object reports meaningful discriminant structure, e.g., a Proportion of trace concentrated on LD1/LD2 as in the README example (0.7906 and 0.1890)
- Prior probabilities and group means in the model output are balanced across the labeled groups (e.g., equal priors of 0.1666667 across six day labels)
- Per-cell predicted labels, LD coordinates, and error-adjacency metrics are present in the exported results file (cell_cycle_lda_results.csv)

## Limitations

- HSS-LDA may not perform well when a large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point)
- Performance will suffer with an increasing number of populations, and HSS is not feasible for more than 100 dimensions
- LDA does not capture non-linear relationships, which may limit performance on some datasets
- HSS may disregard features only useful for separating a single class, and relevant colinear features may not all be selected

## Evidence

- [abstract] LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling study of specific aspects of cellular heterogeneity: [evidence span withheld — anchor status: not_found]
- [intro] Supervised framing of LDA as a classifier for labeled single-cell data: "we demonstrate that LDA is an effective supervised tool to visualize and organize cells according to a priori labels such as cell type, cell-cycle phase"
- [readme] README description of LDA's purpose as a classification algorithm: "Linear Discriminant Analysis (LDA) is a classification algorithm that
we’ve repurposed for supervised dimensionality reduction of single-cell
data."
- [readme] README on running LDA directly with MASS when HSS is not needed: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes"
- [discussion] Article caveat that supervised classification requires known labels: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
