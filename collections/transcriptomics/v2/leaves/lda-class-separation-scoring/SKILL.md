---
name: lda-class-separation-scoring
description: Use when fitting LDA on single-cell data with known a priori labels (e.g.,
  day0–day5 CyTOF markers) and you need to score candidate feature subsets to pick
  the subset with the class separation.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — supervised
  separation scoring is inappropriate in these situations.
- The dataset has more than ~100 dimensions, where HSS-based subset scoring is not
  computationally feasible.
- A large degree of heterogeneity exists within a class or there are many populations
  — separation-score-based subset selection will perform poorly.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: hsslda
  role: R package implementing runHSS(), which executes the forward-reverse stepwise
    loop and scores each candidate subset via score.method / custom.score.method
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Fits pairwise lda models over initial 2-marker combinations to seed the subset
    search and provides the final LDA model
- name: ggplot2
  role: Renders the ElbowPlot of separation score versus number of markers, ggplot-configurable
- name: cluster
  role: Dependency required for silhouette-based subset scoring
- name: entropy
  role: Dependency required for pixel class entropy (pixel.entropy) subset scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:fe6fba04936bbe354a3f4bc02a8488bf02d79de97315ecf79c64f9e784f10c1c
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# lda-class-separation-scoring

## Summary

Scores candidate marker subsets produced by hybrid subset selection (HSS) with a selectable class-separation metric (euclidean, silhouette, pixel.entropy, pixel.density, or a custom function) so the feature set fed to LDA best separates a priori labels in single-cell data. It replaces manual, user-intensive tuning of the LDA input feature set.

## When to use

Use when fitting LDA on single-cell data with known a priori labels (e.g., day0–day5 CyTOF markers) and you need to score candidate feature subsets to pick the subset with the best class separation.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — supervised separation scoring is inappropriate in these situations.
- The dataset has more than ~100 dimensions, where HSS-based subset scoring is not computationally feasible.
- A large degree of heterogeneity exists within a class or there are many populations — separation-score-based subset selection will perform poorly.

## Inputs

- Feature matrix train.x: cells (rows) x 15 metabolic markers (GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A) from TcellHartmann2020_sampleData
- Class label vector train.y (a priori labels, e.g., day0–day5) aligned to rows of train.x
- score.method: one of 'euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom'
- custom.score.method: user-supplied scoring function taking (x, y, custom.score.method, ...) when score.method = 'custom'
- downsample: Boolean (default TRUE; set FALSE to use all input cells)

## Outputs

- HSSscores table: one separation-score row per evaluated candidate subset, spanning subset sizes 2–15
- ElbowPlot (ggplot2): score versus number of markers with calculated elbow point marking the optimal feature set
- Optimal feature (marker) subset used to train the final HSS-LDA model

## How to apply

Inside the HSS loop, each iteration performs a forward step (add one marker) and a reverse step (remove one marker), yielding candidate marker subsets of increasing size from an initial best 2-marker pair (fit with MASS::lda over all pairs). Score every candidate subset with the metric chosen via score.method: 'euclidean', 'silhouette', 'pixel.entropy', or 'pixel.density' (the pixel-based metrics build a grid with create_pixel_grid and rely on the entropy and cluster packages), or pass a user-supplied separation function through custom.score.method with score.method = 'custom'. Collect all subset scores into the HSSscores table, set downsample = FALSE to use all cells rather than the runtime-saving default of TRUE, then identify the optimal feature set at the elbow point of score versus number of markers. The rationale is that LDA separates a priori classes, so scoring subsets directly on class separability yields LD axes tailored to the biological aspect of interest (e.g., differentiation over time).

## Related tools

- **hsslda** (R package implementing runHSS(), which executes the forward-reverse stepwise loop and scores each candidate subset via score.method / custom.score.method) — https://github.com/mamouzgar/hsslda
- **MASS** (Fits pairwise lda models over initial 2-marker combinations to seed the subset search and provides the final LDA model)
- **ggplot2** (Renders the ElbowPlot of separation score versus number of markers, ggplot-configurable)
- **cluster** (Dependency required for silhouette-based subset scoring)
- **entropy** (Dependency required for pixel class entropy (pixel.entropy) subset scoring)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- hss_scores.csv exists and contains one scored row for every evaluated candidate subset spanning subset sizes 2–15 (matching the README run log that iterates 'Number of markers: 2' through 15 with forward and reverse steps).
- The reported elbow point on the ElbowPlot identifies the feature set consistent with the maximum/improving separation scores in the HSSscores table.
- With downsample = FALSE, all cells are used: the output reports 'Using all input data for HSS-LDA' and the downsampled.cells.included.in.analysis vector is TRUE for every row.
- A final MASS::lda model trained only on the optimal marker subset is returned, with group prior probabilities (~0.1667 per day0–day5 class) and LD coefficients.
- Custom scoring path returns the same structure when score.method = 'custom' is paired with a function accepting x, y, and custom.score.method arguments.

## Limitations

- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class.
- Relevant colinear features may not all be selected, as model separability may not improve by keeping multiple colinear features.
- Pixel-based metrics (pixel.entropy, pixel.density) require additional package dependencies (entropy, cluster).
- Downsampling defaults to TRUE to improve runtime; scores from a downsampled run may differ from the full-data run.

## Evidence

- [abstract] HSS scores candidate subsets with selectable separation metric: "We implement feature selection by hybrid subset selection (HSS)"
- [methods] Elbow point defines the optimal feature set: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [methods] Custom separation metric can be plugged into runHSS: "You can then input your custom function into the hsslda::runHSS() function as an argument into custom.score.method"
- [readme] Concrete scoring run over the 15-marker T cell dataset: "hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)"
