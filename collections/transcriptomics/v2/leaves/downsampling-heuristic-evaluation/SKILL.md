---
name: downsampling-heuristic-evaluation
description: Use when benchmarking runtime scaling of supervised dimensionality reduction
  (HSS-LDA) vs unsupervised methods across increasing single-cell counts (up to ~50,000-200,000
  cells), or when deciding whether to keep the runHSS downsample=TRUE default.
when_to_use_negative:
- When labels are unknown or an unbiased view of the data is preferable — the supervised
  LDA approach is inappropriate in these situations
- When the dataset exceeds 100 dimensions — HSS feature selection is not feasible
  in that regime
- When preserving local data structure or capturing non-linear relationships is the
  primary goal, since LDA maximizes class separability at the expense of local structure
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3316
tools:
- name: hsslda
  role: R package providing runHSS() with the downsample argument (Boolean, defaults
    to TRUE) that downsamples data to improve runtime, and the packaged TcellHartmann2020_sampleData
    used for the benchmark
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Provides baseline lda() to run LDA without HSS for the runtime comparison
- name: R
  role: Implementation environment for the benchmark; hsslda, MASS, cluster, entropy,
    and ggplot2 dependencies are called explicitly
- name: ggplot2
  role: Used to plot runtimes on a log2-transformed scale across cell counts and to
    visualize HSS-LD axes
- name: HSS-LDA
  role: The supervised dimensionality-reduction method being benchmarked against PCA,
    UMAP, and PHATE
  repo: https://github.com/mamouzgar/hsslda
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:12863a5fc35bce5ee347097c031f82c553cc223abccfac85d8766da76737c7d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - HSS-LDA
  - MASS
  - R
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# downsampling-heuristic-evaluation

## Summary

A skill for benchmarking the runtime scaling of the hsslda downsample heuristic (a Boolean defaulting to TRUE that subsets cells to improve runtime) against competing dimensionality-reduction methods (PCA, UMAP, PHATE) as single-cell dataset size grows, using silhouette and pixel class entropy (PCE) scores to confirm embedding quality is preserved.

## When to use

Use when benchmarking runtime scaling of supervised dimensionality reduction (HSS-LDA) vs unsupervised methods across increasing single-cell counts (up to ~50,000-200,000 cells), or when deciding whether to keep the runHSS downsample=TRUE default.

## When NOT to use

- When labels are unknown or an unbiased view of the data is preferable — the supervised LDA approach is inappropriate in these situations
- When the dataset exceeds 100 dimensions — HSS feature selection is not feasible in that regime
- When preserving local data structure or capturing non-linear relationships is the primary goal, since LDA maximizes class separability at the expense of local structure

## Inputs

- TcellHartmann2020_sampleData from the hsslda R package (cells x 15 metabolic markers plus a labels column)
- train.x: dataframe of cells (rows) and marker columns GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A
- train.y: vector of class labels (e.g., day0-day5) matching training data rows
- Cell-count subsets scaling up to 50,000-200,000 cells

## Outputs

- Per-method runtimes at each cell count, reported on a log2-transformed scale
- Silhouette scores (cluster package) for each method's embedding
- PCE (pixel class entropy) scores (entropy package via create_pixel_grid) for each method's embedding
- benchmark_results.csv compiling runtimes, silhouette scores, and PCE scores across methods and cell counts

## How to apply

Load the packaged TcellHartmann2020_sampleData from hsslda, extract the 15 marker columns (GLUT1 through HIF1A) into train.x and the labels column into train.y. Run hsslda::runHSS sweeping score.method across 'euclidean', 'silhouette', 'pixel.entropy', and 'pixel.density' while toggling the downsample argument to exercise the subsetting heuristic on cell-count subsets scaling up to 50,000-200,000 cells. Run competing methods on the identical feature sets: baseline LDA via MASS::lda plus PCA, UMAP, and PHATE, recording runtimes for each method at each cell count and reporting them on a log2 scale because the gap grows so large that linear axes are unreadable. Score each method's embedding with silhouette scores (cluster package) and PCE scores (entropy package, using create_pixel_grid) and compile runtimes, silhouette, and PCE values into benchmark_results.csv. Judge success by reproducing the reported pattern: LDA and PCA are significantly faster than UMAP and PHATE, with the runtime gap growing with cell count, while ranking remains consistent (LDA best silhouette on T cell metabolism and Chromotyping, second to UMAP on Morphometry; LDA superior on continuous datasets).

## Related tools

- **hsslda** (R package providing runHSS() with the downsample argument (Boolean, defaults to TRUE) that downsamples data to improve runtime, and the packaged TcellHartmann2020_sampleData used for the benchmark) — https://github.com/mamouzgar/hsslda
- **MASS** (Provides baseline lda() to run LDA without HSS for the runtime comparison)
- **R** (Implementation environment for the benchmark; hsslda, MASS, cluster, entropy, and ggplot2 dependencies are called explicitly)
- **ggplot2** (Used to plot runtimes on a log2-transformed scale across cell counts and to visualize HSS-LD axes)
- **HSS-LDA** (The supervised dimensionality-reduction method being benchmarked against PCA, UMAP, and PHATE) — https://github.com/mamouzgar/hsslda

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- benchmark_results.csv reproduces the reported ranking: LDA best silhouette on T cell metabolism and Chromotyping, second to UMAP on Morphometry; UMAP/PHATE better PCE on Morphometry; LDA superior on continuous datasets
- Runtime plot requires a log2-transformed scale to visualize the discrepancy between algorithms, confirming LDA and PCA are significantly faster and the gap grows with cell counts
- Toggling downsample=TRUE vs FALSE yields consistent HSS-LDA results, with downsampled.cells.included.in.analysis reflecting which rows were included (all TRUE when no downsampling was performed)
- Silhouette scores computed with the cluster package and PCE scores with the entropy package (via create_pixel_grid) are present for every method embedding in the benchmark
- Same feature set (the 15 GLUT1-HIF1A marker columns) is used across all competing methods (LDA, PCA, UMAP, PHATE)

## Limitations

- LDA and the benchmark are supervised: performance depends on a priori class labels and may not perform well when a large degree of heterogeneity exists within a given class or with an increasing number of populations
- LDA does not capture non-linear relationships, which may limit its performance on some datasets
- HSS is not feasible for more than 100 dimensions, and relevant colinear features may not all be selected; features useful for separating a single class may be disregarded

## Evidence

- [methods] downsample heuristic: "(optional) downsample: Boolean, defaults to TRUE. Downsamples data to improve runtime."
- [results] runtime benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] benchmark scope: "We benchmark HSS-LDA against several popular dimensionality-reduction algorithms and illustrate its utility and versatility for the exploration of single-cell mass cytometry, transcriptomics, and"
- [readme] example data: "channels = c('GLUT1', 'HK2', 'GAPDH', 'LDHA', 'MCT1', 'PFKFB4', 'IDH2', 'CyclinB1', 'GLUD12', 'CS', 'OGDH', 'CytC', 'ATP5A', 'S6_p', 'HIF1A')"
- [discussion] supervised limitation: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
