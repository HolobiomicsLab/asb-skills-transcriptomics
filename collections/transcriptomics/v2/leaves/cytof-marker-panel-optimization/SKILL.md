---
name: cytof-marker-panel-optimization
description: Use when you have single-cell mass-cytometry-style data with an a priori
  class label (e.
when_to_use_negative:
- Class labels are unknown, or an unbiased (unsupervised) view of the data is preferable
  — HSS-LDA is a supervised method and inappropriate in that setting.
- Data has more than ~100 dimensions — HSS is not feasible there; plain LDA or sda
  should be used instead.
- Large heterogeneity within a given class or a large number of populations (e.g.,
  PBMCs labeled only by time point) — separation performance will suffer.
tools:
- name: hsslda
  role: R package implementing LDA with hybrid subset selection; runHSS() performs
    the forward-reverse stepwise feature-selection loop, ElbowPlot() renders the elbow
    point, makeAxes() projects new data onto the learned LD axes
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Provides lda() for fitting the LDA models scored during subset evaluation,
    and for running LDA without HSS
- name: ggplot2
  role: Renders the elbow plot and the final HSS_LD1 vs HSS_LD2 visualization of cells
    colored by class labels
- name: entropy
  role: Dependency for pixel-based separation metrics (pixel.entropy) used to score
    candidate subsets; must be installed before hsslda
- name: cluster
  role: Dependency for silhouette-based scoring of candidate feature subsets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:a0416777b191d1b33d2b947ff75a9a0401a570d2b4d645e842e4cccd4a23fc1a
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cytof-marker-panel-optimization

## Summary

Automated tuning of the marker/feature set input to LDA for supervised dimensionality reduction of single-cell (CyTOF) data, using hybrid subset selection (HSS) — a heuristic combining forward and reverse stepwise selection scored by class-separation metrics — so that a priori labels (e.g., day0–day5 time points over 15 metabolic markers) are optimally separated for visualization.

## When to use

Use when you have single-cell mass-cytometry-style data with an a priori class label (e.g., time points or cell types) over a moderate number of markers (~2–100), and want to automatically select the marker subset that best separates those labels for LDA visualization instead of manually tuning the feature set.

## When NOT to use

- Class labels are unknown, or an unbiased (unsupervised) view of the data is preferable — HSS-LDA is a supervised method and inappropriate in that setting.
- Data has more than ~100 dimensions — HSS is not feasible there; plain LDA or sda should be used instead.
- Large heterogeneity within a given class or a large number of populations (e.g., PBMCs labeled only by time point) — separation performance will suffer.

## Inputs

- Marker/cell dataframe train.x (rows = cells, columns = markers/genes, e.g., 15 metabolic channels from TcellHartmann2020_sampleData)
- Class label vector train.y (factor per cell, e.g., day0–day5)
- score.method choice: 'euclidean' | 'silhouette' | 'pixel.entropy' | 'pixel.density' | 'custom'
- custom.score.method: user-supplied separation-metric function (required when score.method = 'custom')
- downsample: Boolean (default TRUE; set FALSE to use all input cells)

## Outputs

- HSSscores table of separation scores for every evaluated candidate subset (subset sizes 2–15)
- ElbowPlot (ggplot2) visualizing score vs. number of markers with the calculated elbow point
- HSS-LDA-model: final MASS::lda model on the optimal feature set (usable like any R model)
- HSS-LDA-result: merged dataframe of input markers, class labels, and LD axes (HSS_LD1, HSS_LD2, ...) for plotting
- hss_scores.csv: written score table

## How to apply

Load the TcellHartmann2020_sampleData-style dataset and split it into a marker matrix train.x (15 metabolic markers: GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A) and a class-label vector train.y (day0–day5). Call hsslda::runHSS(), which begins by fitting pairwise LDA models (MASS::lda) over all 2-marker combinations to select the best initial subset, then at each iteration performs a forward step (add one marker) and a reverse step (remove one marker), scoring each candidate subset with score.method — one of 'euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom' with a user function passed via custom.score.method; pixel-based metrics require building pixel grids via create_pixel_grid and the entropy/cluster packages. Set downsample = FALSE to score on all cells rather than the downsampled default. Aggregate all subset scores in the HSSscores table and pick the optimal feature set at the elbow point of score versus number of markers via ElbowPlot; the final HSS-LDA-model can be applied to new data with makeAxes using the model's $scaling.

## Related tools

- **hsslda** (R package implementing LDA with hybrid subset selection; runHSS() performs the forward-reverse stepwise feature-selection loop, ElbowPlot() renders the elbow point, makeAxes() projects new data onto the learned LD axes) — https://github.com/mamouzgar/hsslda
- **MASS** (Provides lda() for fitting the LDA models scored during subset evaluation, and for running LDA without HSS)
- **ggplot2** (Renders the elbow plot and the final HSS_LD1 vs HSS_LD2 visualization of cells colored by class labels)
- **entropy** (Dependency for pixel-based separation metrics (pixel.entropy) used to score candidate subsets; must be installed before hsslda)
- **cluster** (Dependency for silhouette-based scoring of candidate feature subsets)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- hss_scores.csv exists and contains one scored row for every evaluated candidate subset, spanning subset sizes 2–15 (matching the runHSS console log 'Number of markers: 2' through 15).
- The reported elbow point on ElbowPlot identifies the optimal feature set consistently with the score values in the HSSscores table.
- The final HSS-LDA-model has the expected LDA structure: prior probabilities across all 6 day-labels, group means, coefficients of linear discriminants, and proportion-of-trace values (LD1 should carry the largest share of variance, e.g., 0.7906 in the example run).
- makeAxes applied to new data with the model's $scaling yields the same LD axis projection; plotting HSS_LD1 vs HSS_LD2 colored by labels should show visibly separated day classes.
- With downsample = FALSE, downsampled.cells.included.in.analysis is TRUE for all rows, confirming all input cells were used in scoring.

## Limitations

- HSS enriches for features useful for separating multiple classes, so features only useful for separating a single class may be disregarded.
- Relevant colinear features may not all be selected, as separability may not improve by keeping multiple colinear features; selected features require follow-up analysis for the nature and statistical significance of differences.
- HSS is not feasible for more than 100 dimensions, and LDA does not capture non-linear relationships; performance also suffers with an increasing number of populations or large heterogeneity within a class.

## Evidence

- [intro] HSS heuristic approach: "We implement hybrid subset selection (HSS), a heuristic approach using elements of both forward and reverse stepwise selection"
- [readme] runHSS usage: "You can run hybrid subset selection on a dataset using runHSS()."
- [methods] Elbow plot output: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [readme] Custom scoring metric: "You can also add your own custom separation metric to perform feature selection with by changing score.method = ‘custom’"
- [abstract] Reproducibility of axes: "this computationally efficient approach generates non-stochastic, interpretable axes"
