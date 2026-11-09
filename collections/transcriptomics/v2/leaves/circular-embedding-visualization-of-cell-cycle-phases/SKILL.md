---
name: circular-embedding-visualization-of-cell-cycle-phases
description: Use when single-cell data carry a priori labels along a cyclical process
  (e.g., G1.S, S, G2, G2.M, M.G1 cell-cycle phase scores) and the goal is a supervised,
  interpretable circular embedding plus phase-classification accuracy rather than
  an unbiased clustering.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable (the supervised
  approach is explicitly inappropriate there)
- More than 100 feature dimensions, where HSS feature selection is not feasible
- Large heterogeneity within a class or a large number of populations, where HSS-LDA
  performance suffers
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASS
  role: Provides lda() to train the LDA model on the cell-cycle score matrix with
    phase labels and generate LD axes; used without HSS feature selection for this
    cyclical embedding
- name: hsslda
  role: R package offering runHSS() for hybrid subset selection feature selection
    and makeAxes() to project cells onto the LD axes
  repo: https://github.com/mamouzgar/hsslda
- name: ggplot2
  role: Plots the LD embedding and circular (polar-coordinate) visualization of cell-cycle
    phases
- name: R
  role: Implementation environment for the LDA projection, polar transformation, cross-validation,
    and trajectory smoothing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:9b1d638fe57aa26f7c84a3c2cd31d20b7e0881d9655aafd6e37e1a8f22cd9196
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# circular-embedding-visualization-of-cell-cycle-phases

## Summary

Repurposes linear discriminant analysis (LDA) as supervised dimensionality reduction to project single cells onto LD axes built from a priori cell-cycle phase labels, then converts LD1/LD2 coordinates to polar (angle, radius) coordinates to obtain a circular embedding in which cell-cycle phases are ordered cyclically.

## When to use

Use when single-cell data carry a priori labels along a cyclical process (e.g., G1.S, S, G2, G2.M, M.G1 cell-cycle phase scores) and the goal is a supervised, interpretable circular embedding plus phase-classification accuracy rather than an unbiased clustering.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable (the supervised approach is explicitly inappropriate there)
- More than 100 feature dimensions, where HSS feature selection is not feasible
- Large heterogeneity within a class or a large number of populations, where HSS-LDA performance suffers

## Inputs

- Per-cell cell-cycle module score matrix with columns G1.S, S, G2, G2.M, M.G1 (cells x 5 scores)
- A priori cell-cycle phase class labels per cell
- Gene expression matrix from the Hartmann 2020 T cell proliferation scRNA-seq dataset (e.g., TcellHartmann2020_sampleData) for marker trajectory plots

## Outputs

- Circular LDA embedding (polar angle/radius per cell) reproducing Figure 6E
- Cross-validated phase-classification accuracy (~88%) and fraction of misclassifications between adjacent phases (>99%)
- Angular pseudotime-ordered smoothed expression trajectories of CCNB1, CDK1, and TPX2 (Figure 6H)
- cell_cycle_lda_results.csv with per-cell phase scores, LD coordinates, angular positions, and CV metrics

## How to apply

Compute per-cell module scores for the curated G1.S, S, G2, G2.M, and M.G1 gene sets to build a 5-column score matrix, then train an LDA model with cell-cycle phase as the a priori class labels using MASS::lda() in R (optionally with feature selection via the hsslda package's runHSS) and project cells onto the LD1/LD2 axes with makeAxes. Convert the LD1/LD2 coordinates to polar (angle, radius) coordinates and plot the circular embedding with ggplot2 so phases land at their expected angular positions (Figure 6E). Run cross-validated classification of cells into the five phases and record overall accuracy plus the fraction of errors between adjacent phases; order cells by angular pseudotime and plot smoothed expression of G2/M markers (CCNB1, CDK1, TPX2) to confirm the cyclic ordering (Figure 6H). Save per-cell phase scores, LD coordinates, angular positions, and cross-validation metrics to a results CSV.

## Related tools

- **MASS** (Provides lda() to train the LDA model on the cell-cycle score matrix with phase labels and generate LD axes; used without HSS feature selection for this cyclical embedding)
- **hsslda** (R package offering runHSS() for hybrid subset selection feature selection and makeAxes() to project cells onto the LD axes) — https://github.com/mamouzgar/hsslda
- **ggplot2** (Plots the LD embedding and circular (polar-coordinate) visualization of cell-cycle phases)
- **R** (Implementation environment for the LDA projection, polar transformation, cross-validation, and trajectory smoothing)

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- Cross-validated phase-classification accuracy approximates the reported ~88%
- More than 99% of classification errors occur between adjacent cell-cycle phases, confirming circular phase ordering
- Cell-cycle phases appear at their expected angular positions on the polar (LD1/LD2) embedding
- Smoothed CCNB1, CDK1, and TPX2 expression along angular pseudotime peaks near the G2/M region of the cycle
- cell_cycle_lda_results.csv contains per-cell phase scores, LD coordinates, angular positions, and cross-validation metrics with consistent row counts

## Limitations

- LDA maximizes class separability and comes at the expense of local data structure
- LDA does not capture non-linear relationships, which may limit performance on some datasets
- HSS feature selection may not keep relevant colinear features and may disregard features only useful for separating a single class
- Performance will suffer with an increasing number of populations, and HSS-LDA may underperform when a large degree of heterogeneity exists within a class

## Evidence

- [intro] LDA as supervised tool for a priori labels: "Here, we demonstrate that LDA is an effective supervised tool to visualize and organize cells according to a priori labels such as cell type, cell-cycle phase, or experimental time point."
- [readme] MASS for LDA without HSS: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [readme] ggplot2 plotting of LD axes: "ggplot2::ggplot(lda.df,ggplot2::aes(x = HSS_LD1, y = HSS_LD2, color = labels)) +"
- [discussion] Supervised approach inappropriate without labels: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
- [discussion] HSS-LDA dimension limit: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
