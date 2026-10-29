---
name: stepwise-feature-selection
description: Use when single-cell data (e.g., CyTOF markers) has a priori class labels
  (e.g., time points day0–day5) and you need to automatically tune the feature set
  input to LDA for visualization instead of manually picking markers.
when_to_use_negative:
- Labels are unknown, or an unbiased view of the data is preferable — HSS-LDA is a
  supervised approach
- Data has more than 100 dimensions, where HSS is not feasible
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled
  by time point) or the number of populations is large — performance suffers
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0092
tools:
- name: hsslda
  role: R package implementing runHSS() hybrid subset selection with LDA feature selection;
    provides runHSS, makeAxes, ElbowPlot
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Fits the underlying LDA models (MASS::lda) used to score candidate feature
    subsets and produce the final HSS-LDA-model
- name: ggplot2
  role: Renders the elbow plot and the final HSS_LD1 vs HSS_LD2 visualization of labeled
    cells
- name: cluster
  role: Dependency for pixel-based separation metrics such as silhouette scoring of
    candidate subsets
- name: entropy
  role: Dependency for the pixel class entropy (PCE) separation metric; must be installed
    before hsslda
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:f70712f07855e55523215d4013ba66fa39041838917ae563e0637c07716e5efd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - cluster
  - dplyr
  - entropy
  - ggplot2
  - hsslda
  - stats
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# stepwise-feature-selection

## Summary

Hybrid subset selection (HSS) combines forward and reverse stepwise selection to automatically identify a marker subset whose separation scores maximize class separation for LDA-based supervised dimensionality reduction of single-cell data.

## When to use

Use when single-cell data (e.g., CyTOF markers) has a priori class labels (e.g., time points day0–day5) and you need to automatically tune the feature set input to LDA for visualization instead of manually picking markers.

## When NOT to use

- Labels are unknown, or an unbiased view of the data is preferable — HSS-LDA is a supervised approach
- Data has more than 100 dimensions, where HSS is not feasible
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) or the number of populations is large — performance suffers

## Inputs

- x: dataframe of cells (rows) and markers/genes (columns), e.g., 15 metabolic markers from TcellHartmann2020_sampleData
- y: vector of a priori class labels matching training rows (e.g., day0–day5)
- score.method: one of 'euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom'
- custom.score.method: user-supplied separation metric function (required when score.method = 'custom')
- downsample: Boolean (default TRUE); set FALSE to use all input data

## Outputs

- HSSscores: table of separation scores for every evaluated candidate subset spanning subset sizes 2–15
- ElbowPlot: ggplot2 elbow plot with calculated elbow point for the optimal feature set
- HSS-LDA-model: final MASS::lda model restricted to the optimal marker set
- HSS-LDA-result: merged dataframe of markers, class labels, and LD axes (LD1–LD5)
- downsampled.cells.included.in.analysis: boolean vector of rows included in analysis

## How to apply

Load the marker matrix (e.g., train.x from TcellHartmann2020_sampleData with 15 metabolic markers such as GLUT1, HK2, GAPDH, LDHA, MCT1) and class label vector (train.y, day0–day5). Run hsslda::runHSS(), which fits pairwise LDA models via MASS::lda over all 2-marker combinations as the initial subset, then iterates: each round performs a forward step (add one marker) and a reverse step (remove one marker), scoring each candidate subset with a separation metric chosen via score.method ('euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom' with a user function passed through custom.score.method). Set downsample = FALSE to use all cells. Aggregate subset scores in HSSscores, then determine the optimal feature set at the elbow point of score versus number of markers using the ggplot-configurable ElbowPlot. The final HSS-LDA-model and HSS-LDA-result are returned and can project new data via makeAxes.

## Related tools

- **hsslda** (R package implementing runHSS() hybrid subset selection with LDA feature selection; provides runHSS, makeAxes, ElbowPlot) — https://github.com/mamouzgar/hsslda
- **MASS** (Fits the underlying LDA models (MASS::lda) used to score candidate feature subsets and produce the final HSS-LDA-model)
- **ggplot2** (Renders the elbow plot and the final HSS_LD1 vs HSS_LD2 visualization of labeled cells)
- **cluster** (Dependency for pixel-based separation metrics such as silhouette scoring of candidate subsets)
- **entropy** (Dependency for the pixel class entropy (PCE) separation metric; must be installed before hsslda)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- hss_scores.csv exists and contains one scored row for every evaluated candidate subset, spanning subset sizes 2 through 15
- The elbow point reported on the ElbowPlot identifies the optimal feature set consistent with the HSSscores table
- The final HSS-LDA-model output shows coefficients of linear discriminants only for markers in the optimal subset (README example: 10 markers, LD1–LD5 with Proportion of trace LD1 = 0.7906)
- With downsample = FALSE, downsampled.cells.included.in.analysis contains all TRUE values
- Projecting new data with makeAxes using the model's scaling matrix produces LD coordinates consistent with the training axes

## Limitations

- Relevant colinear features may not all be selected by HSS, as separability may not improve by keeping multiple colinear features
- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class
- Features selected by HSS are differentially expressed by one or more labeled classes and require follow-up analysis to understand the nature and statistical significance of these differences
- LDA does not capture non-linear relationships, which may limit performance on some datasets
- HSS-LDA performance suffers with an increasing number of populations and with high heterogeneity within a class

## Evidence

- [intro] HSS loop: "We implement hybrid subset selection (HSS), a heuristic approach using elements of both forward and reverse stepwise selection, to identify a set of features that enable enhanced separation"
- [methods] custom metric: "You can also add your own custom separation metric to perform feature selection with by changing score.method = 'custom'"
- [methods] elbow plot: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [readme] score methods: "score.method: A scoring metric to use for feature selection, which can include: ‘euclidean’, ‘silhouette’, ‘pixel.entropy’, ‘pixel.density’, or ‘custom’"
- [readme] downsample flag: "(optional) downsample: Boolean, defaults to TRUE. Downsamples data to improve runtime. Set to FALSE to use all input data."
