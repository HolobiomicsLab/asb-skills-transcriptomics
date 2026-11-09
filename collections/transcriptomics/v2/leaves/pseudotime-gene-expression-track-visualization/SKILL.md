---
name: pseudotime-gene-expression-track-visualization
description: Use when a supervised LDA embedding yields a circular or ordered cell
  arrangement (e.g., cell-cycle phases on LD axes) and you need to confirm phase ordering
  by tracking marker expression (e.g., CCNB1, CDK1, TPX2) along the inferred pseudotime.
when_to_use_negative:
- Labels are unknown or an unbiased view of the data is preferable — LDA/HSS-LDA is
  a supervised approach and inappropriate in these situations
- The embedding does not represent a cyclic or ordinal process — angular pseudotime
  ordering is only meaningful when phases form a circular trajectory (e.g., cell cycle),
  not a branching or unstructured layout
- Data has more than 100 dimensions, where HSS feature selection is not feasible and
  LDA performance may suffer
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: MASS
  role: Runs the base LDA (lda()) whose LD1/LD2 coordinates are converted to polar/angular
    pseudotime before trajectory plotting
- name: hsslda
  role: R package implementing LDA with hybrid subset selection; its HSS-LDA-result
    table of LD axes can serve as the coordinate source for pseudotime ordering
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Plots the smoothed marker expression trajectories along angular pseudotime
    and the polar LDA embedding
- name: dplyr
  role: Data manipulation for sorting cells by angular pseudotime and joining expression
    values
- name: tidyr
  role: Reshaping per-cell scores and expression values into long format for trajectory
    plotting
- name: stats
  role: Base R functions for converting LD coordinates to polar (angle, radius) values
    and smoothing expression along pseudotime
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:bfbddf20762d4234351ea4b68b8a8a2679793308ac7b68888a8ef9974c5e0b8a
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pseudotime-gene-expression-track-visualization

## Summary

Orders cells along a pseudotime axis derived from an LDA embedding (e.g., angular position on the cyclical cell-cycle LD1/LD2 embedding) and plots smoothed expression trajectories of marker genes along that axis to validate that the supervised embedding recapitulates the underlying biology.

## When to use

Use when a supervised LDA embedding yields a circular or ordered cell arrangement (e.g., cell-cycle phases on LD axes) and you need to confirm phase ordering by tracking marker expression (e.g., CCNB1, CDK1, TPX2) along the inferred pseudotime.

## When NOT to use

- Labels are unknown or an unbiased view of the data is preferable — LDA/HSS-LDA is a supervised approach and inappropriate in these situations
- The embedding does not represent a cyclic or ordinal process — angular pseudotime ordering is only meaningful when phases form a circular trajectory (e.g., cell cycle), not a branching or unstructured layout
- Data has more than 100 dimensions, where HSS feature selection is not feasible and LDA performance may suffer

## Inputs

- Per-cell LD coordinates (LD1/LD2) from an LDA model trained on the 5-column cell-cycle score matrix (G1.S, S, G2, G2.M, M.G1) from the Hartmann 2020 T cell scRNA-seq dataset
- Per-cell gene expression values (or normalized counts) for marker genes such as CCNB1, CDK1, TPX2
- Per-cell cell-cycle phase labels for coloring/validating trajectory alignment

## Outputs

- Smoothed gene expression trajectory plots along angular pseudotime (reproducing Figure 6H)
- Per-cell table of phase scores, LD coordinates, and angular positions (cell_cycle_lda_results.csv)

## How to apply

After projecting cells onto linear discriminant axes with MASS::lda() (or the hsslda model), convert the LD coordinates to polar coordinates (angle, radius) so that angular position serves as a circular pseudotime for the cell-cycle phases. Sort cells by this angular pseudotime and compute smoothed expression trajectories for cell-cycle marker genes — G2/M markers CCNB1, CDK1, and TPX2 in the Hartmann 2020 T cell scRNA-seq data — plotting the trajectories with ggplot2 to reproduce Figure 6H. The rationale is that a correctly ordered cyclical embedding should show marker expression peaking at the phases where the biology predicts it (G2/M markers near G2.M), providing a trajectory-level validation that complements the ~88% cross-validated phase-classification accuracy and the >99% adjacent-phase-only error pattern.

## Related tools

- **MASS** (Runs the base LDA (lda()) whose LD1/LD2 coordinates are converted to polar/angular pseudotime before trajectory plotting)
- **hsslda** (R package implementing LDA with hybrid subset selection; its HSS-LDA-result table of LD axes can serve as the coordinate source for pseudotime ordering) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Plots the smoothed marker expression trajectories along angular pseudotime and the polar LDA embedding)
- **dplyr** (Data manipulation for sorting cells by angular pseudotime and joining expression values)
- **tidyr** (Reshaping per-cell scores and expression values into long format for trajectory plotting)
- **stats** (Base R functions for converting LD coordinates to polar (angle, radius) values and smoothing expression along pseudotime)

## Examples

```
lda.res <- MASS::lda(x = score_matrix, grouping = phase_labels); ld <- makeAxes(df = score_matrix, co = lda.res$scaling); angle <- atan2(ld$LD2, ld$LD1)
```

## Evaluation signals

- The smoothed CCNB1, CDK1, and TPX2 trajectories peak near the G2.M phase on the angular pseudotime axis, matching expected cell-cycle biology (Figure 6H pattern)
- Cross-validated phase-classification accuracy from the underlying LDA model approximates the reported ~88%
- More than 99% of classification errors fall between adjacent cell-cycle phases, confirming circular ordering of phases on the LD embedding
- Angular positions computed from LD1/LD2 polar conversion arrange the five phases (G1.S, S, G2, G2.M, M.G1) in their expected cyclic order
- Saved results (cell_cycle_lda_results.csv) contain per-cell phase scores, LD coordinates, and angular positions with no missing assignments

## Limitations

- LDA maximizes class separability and minimizes within-class variance at the expense of local data structure, so trajectory smoothing on LD-derived pseudotime may distort fine-grained expression dynamics
- LDA does not capture non-linear relationships, which may limit trajectory fidelity on some datasets
- HSS-LDA may not perform well when a large degree of heterogeneity exists within a given class, and performance suffers with an increasing number of populations
- Relevant colinear genes may not all be selected by HSS, and features useful for separating only a single phase may be disregarded

## Evidence

- [other] Convert the LD1/LD2 coordinates to polar (angle, radius) coordinates to build the circular embedding of cell-cycle phases reproducing Figure 6E, plotting with ggplot2.: [evidence span withheld — anchor status: not_found]
- [other] Order cells by angular pseudotime and plot smoothed G2/M marker trajectories.: [evidence span withheld — anchor status: not_found]
- [abstract] LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling study of cellular heterogeneity.: "LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity."
- [readme] README: makeAxes projects new data onto the LD axes.: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] LDA comes at the expense of local data structure.: "LDA, by definition, maximizes class separability and minimizes variance within each projected class, and by doing so will come at the expense of local data structure."
