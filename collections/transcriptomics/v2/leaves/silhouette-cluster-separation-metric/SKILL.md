---
name: silhouette-cluster-separation-metric
description: Use when running hsslda::runHSS feature selection on single-cell marker
  data (e.g., CyTOF metabolic markers) and you want subset scoring based on cluster
  cohesion vs. separation of a priori class labels rather than euclidean distance.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — HSS-LDA
  is a supervised method and inappropriate in these situations
- Feature space exceeds ~100 dimensions, where HSS (and thus silhouette-scored subset
  search) is not feasible
- Silhouette/pixel-based metrics cannot be used because their dependencies (e.g.,
  cluster, entropy) are unavailable and the default euclidean metric is preferred
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: hsslda
  role: R package providing runHSS() with score.method = 'silhouette' option for silhouette-scored
    hybrid subset selection
  repo: https://github.com/mamouzgar/hsslda
- name: cluster
  role: Dependency used to compute silhouette scores of candidate feature subsets
    during HSS
- name: MASS
  role: Provides lda() used to fit pairwise and final LDA models over candidate subsets
    being scored
- name: ggplot2
  role: Renders the configurable ElbowPlot of silhouette score versus number of markers
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:0796fe893fad5071045c5650a14e5e8442116819e262edf0e7b9cf792397cd43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - cluster
  - dplyr
  - ggplot2
  - hsslda
  - stats
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# silhouette-cluster-separation-metric

## Summary

Uses the silhouette width (cluster package) as a separation metric to score candidate feature subsets during hybrid subset selection (HSS) for HSS-LDA, guiding which markers are retained for the LDA visualization of single-cell data.

## When to use

Use when running hsslda::runHSS feature selection on single-cell marker data (e.g., CyTOF metabolic markers) and you want subset scoring based on cluster cohesion vs. separation of a priori class labels rather than euclidean distance.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — HSS-LDA is a supervised method and inappropriate in these situations
- Feature space exceeds ~100 dimensions, where HSS (and thus silhouette-scored subset search) is not feasible
- Silhouette/pixel-based metrics cannot be used because their dependencies (e.g., cluster, entropy) are unavailable and the default euclidean metric is preferred

## Inputs

- cells-by-markers dataframe (x), e.g., TcellHartmann2020_sampleData with 15 metabolic marker columns (GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A)
- class label vector (y) of a priori labels matching training rows (e.g., day0–day5 time points)
- score.method = 'silhouette' option string; optional downsample Boolean (default TRUE)
- cluster R package installed as dependency for silhouette computation

## Outputs

- hss.result object containing HSSscores (table of silhouette scores per candidate feature subset)
- ElbowPlot (ggplot2) visualizing score versus number of markers with the calculated elbow point for the optimal feature set
- HSS-LDA-model: final MASS lda model trained on the silhouette-optimal feature set
- HSS-LDA-result: merged dataframe of markers, class labels, and LD axes (e.g., HSS_LD1, HSS_LD2)
- Optionally hss_scores.csv export of the score table

## How to apply

Invoke runHSS with score.method = 'silhouette' on a cells-by-markers dataframe (x) and a class label vector (y), e.g., the 15 metabolic markers (GLUT1...HIF1A) and day0–day5 labels of TcellHartmann2020_sampleData. At each HSS iteration, the forward and reverse stepwise steps generate candidate marker subsets, and each candidate subset is scored by computing silhouette scores on the LDA-projected cells using the cluster package (a dependency that must be installed explicitly, since dependencies are not attached by loading hsslda). Subset scores are aggregated in the HSSscores table and the optimal feature set is read from the elbow point of the ElbowPlot (score versus number of markers); set downsample = FALSE to score on all input cells at the cost of runtime. Because silhouette scores require additional dependencies beyond the default euclidean metric, verify the cluster (and for PCE metrics, entropy) packages are available before running.

## Related tools

- **hsslda** (R package providing runHSS() with score.method = 'silhouette' option for silhouette-scored hybrid subset selection) — https://github.com/mamouzgar/hsslda
- **cluster** (Dependency used to compute silhouette scores of candidate feature subsets during HSS)
- **MASS** (Provides lda() used to fit pairwise and final LDA models over candidate subsets being scored)
- **ggplot2** (Renders the configurable ElbowPlot of silhouette score versus number of markers)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'silhouette', downsample = FALSE)
```

## Evaluation signals

- hss.result$HSSscores contains one scored row for every evaluated candidate subset, spanning subset sizes from 2 up to the full marker count (15 for TcellHartmann2020_sampleData)
- hss.result$ElbowPlot renders and the reported elbow point identifies the optimal feature set consistent with the score table
- Final HSS-LDA-model is an MASS lda object whose selected markers correspond to the subset with best silhouette-based separation
- HSS-LDA-result dataframe contains all initially inputted markers, class labels, and LD axes; all downsampled.cells.included.in.analysis values are TRUE when downsample = FALSE
- Projected axes (e.g., HSS_LD1, HSS_LD2) show day0–day5 labels separated, matching benchmark expectations that LDA achieves best silhouette on T cell metabolism data

## Limitations

- Relevant colinear features may not all be selected by HSS, as separability may not improve by keeping multiple colinear features
- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class
- HSS-LDA performance suffers with an increasing number of populations and when a large degree of heterogeneity exists within a given class
- Scoring metrics like silhouette require additional package dependencies beyond those attached by loading hsslda; call library(<dep>) explicitly
- Features selected by HSS are differentially expressed by one or more labeled classes; follow-up analysis is needed to establish the nature and statistical significance of differences
