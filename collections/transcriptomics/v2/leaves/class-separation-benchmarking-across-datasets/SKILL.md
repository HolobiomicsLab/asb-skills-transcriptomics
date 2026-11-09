---
name: class-separation-benchmarking-across-datasets
description: Use when comparing supervised dimensionality-reduction visualizations
  across multiple single-cell datasets to verify that a reduced feature set preserves
  or improves a priori class-label separation relative to a full-feature baseline.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — HSS-LDA
  is a supervised approach and inappropriate in these situations.
- The dataset has more than 100 dimensions, where HSS is not feasible.
- Large heterogeneity exists within a class (e.g., PBMCs labeled by time point) or
  there are many populations, where HSS-LDA performance suffers.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0092
tools:
- name: hsslda
  role: R package implementing runHSS (hybrid subset selection with forward plus reverse
    stepwise selection around LDA), ElbowPlot, and makeAxes for the HSS feature-selection
    and projection steps
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Provides MASS::lda to fit the plain-LDA baseline on the full marker sets for
    the before/after comparison
- name: ggplot2
  role: Plots HSS_LD1/HSS_LD2 and plain-LDA projections colored by class labels (as
    in Figures S1B-S1G) and renders the configurable ElbowPlot
- name: R
  role: Implementation environment for the hsslda workflow (remotes::install_github,
    library(hsslda))
  repo: https://github.com/mamouzgar/hsslda
- name: CyTOF
  role: Mass cytometry data modality supplying the three benchmark datasets (Morphometry,
    T cell metabolic regulome, Chromotyping)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:8e3a29928e2d22b08fd068b3194812661b5d405a68f9ec232e890a4a9524865a
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# class-separation-benchmarking-across-datasets

## Summary

Benchmark HSS-optimized LDA against plain LDA (and PCA/UMAP/PHATE) across multiple CyTOF datasets by qualitatively comparing class-label separation on LD axes before and after HSS feature selection, verifying that dimensionality reduction (e.g., 13/17, 8/32, 24/32 markers) retains or improves separation.

## When to use

Use when comparing supervised dimensionality-reduction visualizations across multiple single-cell datasets to verify that a reduced feature set preserves or improves a priori class-label separation relative to a full-feature baseline.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — HSS-LDA is a supervised approach and inappropriate in these situations.
- The dataset has more than 100 dimensions, where HSS is not feasible.
- Large heterogeneity exists within a class (e.g., PBMCs labeled by time point) or there are many populations, where HSS-LDA performance suffers.

## Inputs

- Cells-by-markers dataframe for each CyTOF dataset (Morphometry, T cell metabolic regulome via TcellHartmann2020_sampleData, Chromotyping)
- A priori class-label vector per dataset (y argument to runHSS)
- Marker/channel column names, e.g. the 15 metabolic channels of TcellHartmann2020_sampleData (GLUT1 ... HIF1A)

## Outputs

- HSSscores table of scores for each subsetted model
- ElbowPlot ggplot object with calculated elbow point per dataset (saved as PNG)
- HSS-LDA-result table with markers, labels, and HSS_LD1/HSS_LD2 linear discriminants
- Plain-LDA projection from MASS::lda plus makeAxes(df, co) for comparison
- hss_lda_feature_sets.csv with per-dataset selected feature lists, subset scores, and retained-feature counts (13/17, 8/32, 24/32)

## How to apply

For each dataset (Morphometry, T cell metabolic regulome via TcellHartmann2020_sampleData, Chromotyping), load a cells-by-markers dataframe plus its a priori class-label vector and run hsslda::runHSS(x, y, score.method = 'euclidean', downsample = FALSE) so all cells feed the hybrid subset selection (forward plus reverse stepwise selection). Inspect the HSSscores table and ElbowPlot object to identify the calculated elbow point defining the optimal feature set, and confirm retained marker counts match the reported reduced sets (13 of 17, 8 of 32, 24 of 32). Fit a plain LDA baseline with MASS::lda on the full marker set and project with makeAxes(df, co) using the model scaling coefficients. Plot both HSS_LD1/HSS_LD2 and plain-LDA projections with ggplot2 colored by class labels (as in Figures S1B-S1G) and judge success by qualitative retention or improvement of class separation on the reduced matrix. Write per-dataset selected feature lists, subset scores, and retained-feature counts to hss_lda_feature_sets.csv and save elbow plots as PNGs.

## Related tools

- **hsslda** (R package implementing runHSS (hybrid subset selection with forward plus reverse stepwise selection around LDA), ElbowPlot, and makeAxes for the HSS feature-selection and projection steps) — https://github.com/mamouzgar/hsslda
- **MASS** (Provides MASS::lda to fit the plain-LDA baseline on the full marker sets for the before/after comparison)
- **ggplot2** (Plots HSS_LD1/HSS_LD2 and plain-LDA projections colored by class labels (as in Figures S1B-S1G) and renders the configurable ElbowPlot)
- **R** (Implementation environment for the hsslda workflow (remotes::install_github, library(hsslda))) — https://github.com/mamouzgar/hsslda
- **CyTOF** (Mass cytometry data modality supplying the three benchmark datasets (Morphometry, T cell metabolic regulome, Chromotyping))

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE); lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- Retained feature counts match the reported reduced sets: 13 of 17 features (Morphometry), 8 of 32 (T cell metabolic regulome), and 24 of 32 (Chromotyping).
- The ElbowPlot object displays a calculated elbow point consistent with the chosen optimal feature set for each dataset.
- HSS-LDA visualizations qualitatively improve or retain the class-label separation achieved by plain LDA on the full marker sets, despite the reduced feature matrix.
- runHSS is called with downsample = FALSE so all input cells are used, and the output object contains HSSscores, ElbowPlot, HSS-LDA-model, and HSS-LDA-result elements.
- hss_lda_feature_sets.csv contains per-dataset selected feature lists, subset scores, and retained-feature counts, with elbow plots saved as PNG files.

## Limitations

- LDA maximizes class separability and comes at the expense of local data structure, and does not capture non-linear relationships.
- HSS enriches for features useful for separating multiple classes, so relevant colinear features may not all be selected and features useful for only a single class may be disregarded; features require follow-up analysis for the nature and statistical significance of differences.
- Performance suffers with an increasing number of populations and with large heterogeneity within a class; HSS is not feasible for more than 100 dimensions.

## Evidence

- [intro] benchmark against PCA/UMAP/PHATE on three datasets: "We compare and benchmark HSS optimized LDA against PCA, UMAP, and PHATE across three mass cytometry datasets"
- [methods] runHSS signature and inputs: "runHSS(x, y, score.method, custom.score.method, downsample)"
- [readme] runHSS called with downsample = FALSE to use all cells: "runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)"
- [readme] plain LDA baseline with MASS then makeAxes: "lda.res = MASS::lda(x = train.x, grouping = train.y)
lda.df = makeAxes(df = train.x, co = lda.res$scaling)"
- [discussion] HSS-LDA inappropriate without labels: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
