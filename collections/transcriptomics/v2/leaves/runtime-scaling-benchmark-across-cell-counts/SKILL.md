---
name: runtime-scaling-benchmark-across-cell-counts
description: Use when comparing computational runtime scaling of dimensionality-reduction
  methods (LDA/HSS-LDA, PCA, UMAP, PHATE) on single-cell data as cell counts increase,
  or when judging whether a supervised LDA embedding trades acceptable runtime for
  class-separation quality (silhouette, PCE).
when_to_use_negative:
- Labels are unknown, or an unbiased view of the data is preferable — the supervised
  LDA approach being benchmarked is inappropriate in these situations
- Input has more than 100 dimensions — HSS feature selection is not feasible there,
  so LDA/HSS-LDA timing comparisons lose their main setup
- The goal is capturing non-linear relationships or local data structure — LDA maximizes
  class separability at the expense of local structure, so benchmark results will
  not favor it on such datasets
tools:
- name: hsslda
  role: R package providing runHSS (LDA with hybrid subset selection feature selection),
    the primary method whose runtime is benchmarked; also packages the TcellHartmann2020_sampleData
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Provides baseline LDA (MASS::lda) without HSS feature selection, used as a
    comparison point; axes generated via makeAxes
- name: ggplot2
  role: R visualization package used to plot runtimes on a log2-transformed scale
    across cell counts and to visualize LD axes (e.g. HSS_LD1 vs HSS_LD2)
- name: R
  role: Implementation environment in which all benchmarked methods (hsslda, MASS::lda,
    PCA, UMAP, PHATE) and scoring (cluster, entropy) are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:7345c8e03996281b7416854c74edfd63813ce9657280a243cf300944642310c4
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
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# runtime-scaling-benchmark-across-cell-counts

## Summary

Benchmarks how the computational runtime of LDA (via hsslda::runHSS and MASS::lda) scales against PCA, UMAP, and PHATE on single-cell datasets of increasing cell counts, alongside silhouette and PCE embedding-quality scores. Demonstrates that the runtime gap between linear methods (LDA, PCA) and nonlinear manifold methods (UMAP, PHATE) grows large enough to require a log2 scale to visualize.

## When to use

Use when comparing computational runtime scaling of dimensionality-reduction methods (LDA/HSS-LDA, PCA, UMAP, PHATE) on single-cell data as cell counts increase, or when judging whether a supervised LDA embedding trades acceptable runtime for class-separation quality (silhouette, PCE).

## When NOT to use

- Labels are unknown, or an unbiased view of the data is preferable — the supervised LDA approach being benchmarked is inappropriate in these situations
- Input has more than 100 dimensions — HSS feature selection is not feasible there, so LDA/HSS-LDA timing comparisons lose their main setup
- The goal is capturing non-linear relationships or local data structure — LDA maximizes class separability at the expense of local structure, so benchmark results will not favor it on such datasets

## Inputs

- Single-cell feature table (cells x markers), e.g. TcellHartmann2020_sampleData with 15 metabolic marker columns (GLUT1, HK2, GAPDH, LDHA, MCT1, PFKFB4, IDH2, CyclinB1, GLUD12, CS, OGDH, CytC, ATP5A, S6_p, HIF1A)
- Vector of a priori class labels (train.y, e.g. day0-day5 labels)
- Cell-count subsets of the same data scaling up to 50,000-200,000 cells

## Outputs

- benchmark_results.csv with runtime, silhouette score (cluster package), and PCE score (entropy package) per method per cell count
- log2-scale runtime comparison plot (ggplot2) showing LDA/PCA vs UMAP/PHATE scaling across cell counts
- Embedding result tables (LD axes from makeAxes, UMAP/PHATE coordinates) for scoring

## How to apply

Load the packaged TcellHartmann2020_sampleData from hsslda and extract the 15 marker columns (GLUT1 through HIF1A) plus the labels column into train.x and train.y. Run hsslda::runHSS sweeping score.method across 'euclidean', 'silhouette', 'pixel.entropy', and 'pixel.density', toggling downsample (default TRUE) to exercise the subsetting heuristic on cell-count subsets scaling up to 50,000-200,000 cells. Run the competing methods on the identical feature sets: baseline LDA via MASS::lda followed by makeAxes, plus PCA, UMAP, and PHATE. Record runtimes at each cell count and report them on a log2-transformed scale (the gap grows so large that linear axes are unreadable); compute silhouette scores (cluster package) and PCE scores (entropy package, using create_pixel_grid) for each method's embedding. Compile runtimes, silhouette scores, and PCE scores into benchmark_results.csv and check the ranking reproduces the reported pattern.

## Related tools

- **hsslda** (R package providing runHSS (LDA with hybrid subset selection feature selection), the primary method whose runtime is benchmarked; also packages the TcellHartmann2020_sampleData) — https://github.com/mamouzgar/hsslda
- **MASS** (Provides baseline LDA (MASS::lda) without HSS feature selection, used as a comparison point; axes generated via makeAxes)
- **ggplot2** (R visualization package used to plot runtimes on a log2-transformed scale across cell counts and to visualize LD axes (e.g. HSS_LD1 vs HSS_LD2))
- **R** (Implementation environment in which all benchmarked methods (hsslda, MASS::lda, PCA, UMAP, PHATE) and scoring (cluster, entropy) are run)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)
```

## Evaluation signals

- Ranking pattern in benchmark_results.csv: LDA best silhouette on T cell metabolism and Chromotyping datasets, second to UMAP on Morphometry; UMAP/PHATE better PCE on Morphometry; LDA superior on continuous datasets
- Runtime pattern: LDA and PCA significantly faster than UMAP and PHATE, with the runtime gap growing as cell counts increase
- Runtime comparison plot requires a log2-transformed y-axis to be readable across the 50,000-200,000 cell-count range
- runHSS result object contains HSSscores, ElbowPlot with a calculated elbow point, HSS-LDA-model, HSS-LDA-result, and downsampled.cells.included.in.analysis (all TRUE when downsample = FALSE)
- downsample toggling must change runtime measurably while the final output still projects all input cells onto the HSS-LD axes

## Limitations

- HSS is not feasible for more than 100 dimensions; in such cases plain LDA or sda still provide useful plots
- LDA does not capture non-linear relationships, which may limit its performance on some datasets, and comes at the expense of local data structure
- HSS-LDA performance suffers with an increasing number of populations and when a large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point)
- Relevant colinear features may not all be selected by HSS, and HSS may disregard features only useful for separating a single class

## Evidence

- [abstract] We benchmark HSS-LDA against several popular dimensionality-reduction algorithms: "We benchmark HSS-LDA against several popular dimensionality-reduction algorithms and illustrate its utility and versatility for the exploration of single-cell mass cytometry, transcriptomics, and"
- [intro] Benchmarked against PCA, UMAP, and PHATE: "We compare and benchmark HSS optimized LDA against PCA, UMAP, and PHATE across three mass cytometry datasets"
- [readme] downsample parameter and runtime: "(optional) downsample: Boolean, defaults to TRUE. Downsamples data to improve runtime. Set to FALSE to use all input data."
- [discussion] HSS dimension limit: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
- [results] LDA runtime advantage (finding card): [evidence span withheld — anchor status: not_found]
