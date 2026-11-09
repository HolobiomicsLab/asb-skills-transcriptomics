---
name: umap-benchmark-comparison
description: Use when single-cell data (mass cytometry, scRNA-seq, or spatially reconstructed
  transcriptomics) have a priori class labels and you need to decide whether supervised
  LD axes separate labeled structure better than unsupervised UMAP/PCA/PHATE embeddings.
when_to_use_negative:
- Class labels are unknown, or an unbiased view of the data is preferable — the article
  explicitly states the supervised approach is inappropriate in these situations.
- No labeled classes exist to supervise LDA, so only UMAP/PCA/PHATE can be computed
  and no benchmark of supervised vs. unsupervised separation is possible.
- The biology of interest is non-linear or requires preserving local neighborhood
  structure — LDA maximizes class separability at the expense of local data structure.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: hsslda
  role: R package providing runHSS for feature selection and makeAxes to project cells
    onto LD axes for the supervised arm of the benchmark
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Fits LDA without HSS feature selection (MASS::lda) when the feature space
    exceeds ~100 dimensions; its $scaling feeds makeAxes
- name: ggplot2
  role: Renders side-by-side scatter plots of LD1/LD2 vs. UMAP colored by labels and
    landmark features
- name: UMAP
  role: Unsupervised nonlinear embedding computed on the same features/PC scores to
    serve as the benchmark comparison against LDA axes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:62309af875657c8068ba9919c89f3cc1116471cce92e972dd4e7ab7c5d875cc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - UMAP
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# umap-benchmark-comparison

## Summary

Benchmark HSS-LDA (supervised linear discriminant axes) against unsupervised embeddings such as UMAP, PCA, and PHATE on the same single-cell data to assess whether a priori class labels (e.g., differentiation time points, spatial zones) are more coherently organized under supervised dimensionality reduction.

## When to use

Use when single-cell data (mass cytometry, scRNA-seq, or spatially reconstructed transcriptomics) have a priori class labels and you need to decide whether supervised LD axes separate labeled structure better than unsupervised UMAP/PCA/PHATE embeddings.

## When NOT to use

- Class labels are unknown, or an unbiased view of the data is preferable — the article explicitly states the supervised approach is inappropriate in these situations.
- No labeled classes exist to supervise LDA, so only UMAP/PCA/PHATE can be computed and no benchmark of supervised vs. unsupervised separation is possible.
- The biology of interest is non-linear or requires preserving local neighborhood structure — LDA maximizes class separability at the expense of local data structure.

## Inputs

- cells x features matrix (e.g., CyTOF marker intensities, scRNA-seq expression counts, or first 50 PC scores of transcriptomic data)
- vector of a priori class labels per cell (e.g., T cell activation day0-day5, intestinal villus zone labels)
- optional landmark gene/marker expression values for overlay (e.g., ADA, SLC2A2)

## Outputs

- LD1/LD2 (or higher LD) coordinate table per cell from hsslda::makeAxes or hss.result$`HSS-LDA-result`
- UMAP (and optionally PCA/PHATE) embedding of the same features
- side-by-side ggplot2 scatter plots colored by class label and landmark feature expression

## How to apply

Compute the embedding alternatives on identical inputs: for LDA, fit on predictor features (or the first 50 principal components when the feature space exceeds ~100 dimensions, since HSS is not feasible above that threshold) with grouping = class labels via MASS::lda, then project all cells onto LD axes with hsslda::makeAxes(df, co = lda.res$scaling). Build a UMAP embedding of the same PC scores or feature matrix for side-by-side comparison. Visualize both embeddings with ggplot2, coloring cells by class label and by landmark features (e.g., ADA, SLC2A2 in the enterocyte villus dataset) to judge whether spatial or temporal order is preserved. Rationale: LDA identifies linear combinations of predictors that optimally separate a priori classes and yields non-stochastic, interpretable axes, whereas UMAP is unbiased and stochastic; benchmarking on the same data reveals which view best organizes the biological process of interest, as in the published villus-trajectory UMAP comparison (Figure 5B/5C).

## Related tools

- **hsslda** (R package providing runHSS for feature selection and makeAxes to project cells onto LD axes for the supervised arm of the benchmark) — https://github.com/mamouzgar/hsslda
- **MASS** (Fits LDA without HSS feature selection (MASS::lda) when the feature space exceeds ~100 dimensions; its $scaling feeds makeAxes)
- **ggplot2** (Renders side-by-side scatter plots of LD1/LD2 vs. UMAP colored by labels and landmark features)
- **UMAP** (Unsupervised nonlinear embedding computed on the same features/PC scores to serve as the benchmark comparison against LDA axes)

## Examples

```
lda.res = MASS::lda(x = train.x, grouping = train.y); lda.df = makeAxes(df = train.x, co = lda.res$scaling)
```

## Evaluation signals

- The LDA trajectory is non-stochastic: repeated runs produce identical LD axis placements (deterministic linear projection), unlike UMAP.
- Landmark features with known spatial/temporal gradients (e.g., ADA peaking at the villus tip, SLC2A2 in mid-villus) show ordered, coherent expression along LD axes, reproducing published comparisons (Figure 5B/5C).
- Class labels form linearly ordered zones/phases along LD1/LD2 with the expected direction (e.g., day0 -> day5 or crypt -> villus tip).
- The makeAxes output merges input features, labels, and LD coordinates into one dataframe with columns matching lda.res$scaling dimensions.
- Both embeddings (LDA and UMAP) are computed on exactly the same input features (same PC scores) so the comparison is apples-to-apples.

## Limitations

- HSS is not feasible above ~100 dimensions, so for high-dimensional data LDA must be run without HSS (e.g., on PC scores) via MASS::lda.
- LDA, by definition, maximizes class separability and minimizes within-class variance, coming at the expense of local data structure.
- LDA does not capture non-linear relationships, which may limit its performance on some datasets relative to UMAP.
- HSS-LDA may not perform well when a large degree of heterogeneity exists within a given class, and performance suffers with an increasing number of populations.
- Relevant colinear features may not all be selected by HSS, and features only useful for separating a single class may be disregarded.

## Evidence

- [results] LDA improved the linear villus trajectory visualization compared to UMAP, and landmark genes with distinct spatial patterns: [evidence span withheld — anchor status: not_found]
- [discussion] HSS is not feasible for more than 100 dimensions: "but in situations in which HSS is not feasible (>100 dimensions) or desirable, LDA and sda still provide useful plots."
- [readme] To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes: "To run LDA without HSS, you can use the MASS package in R, then use makeAxes to generate your LD axes or apply it to a new dataset."
- [intro] We compare and benchmark HSS optimized LDA against PCA, UMAP, and PHATE across three mass cytometry datasets: "We compare and benchmark HSS optimized LDA against PCA, UMAP, and PHATE across three mass cytometry datasets"
- [discussion] This supervised approach is inappropriate in situations in which labels are unknown: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
