---
name: jensen-shannon-divergence-similarity-scoring
description: Use when benchmarking whether a cell-similarity measure captures same-type
  cell similarity more strongly than a baseline such as HVG+PCA Euclidean distance,
  with permutation-labeled null models on PBMC data.
when_to_use_negative:
- You only need a single-cell embedding or visualization rather than a quantitative
  similarity benchmark against a permutation null
- The dataset lacks a marker panel or reliable cell-type labels, so same-type groupings
  needed for intra-type similarity scoring cannot be defined
- Cells are already represented by archetype weights or a precomputed metric space
  and no comparative baseline evaluation is requested
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ACTIONet (Python package)
  role: Provides an.pp.filter_adata, an.pp.reduce_kernel, an.run_ACTIONet, an.tl.load_markers,
    and an.tl.annotate_cells_using_markers; its archetypal metric cell space supplies
    the JSD-based cell distances being scored
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: Reads the 10x h5 matrix (sc.read_10x_h5) and performs library-size normalization
    (sc.pp.normalize_total) and log transformation (sc.pp.log1p) before ACTIONet reduction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:ac25b140c9b1e698670c578b93079e4692e3adbca95749f67675c961cc8becbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# jensen-shannon-divergence-similarity-scoring

## Summary

Scores intra-cell-type similarity using ACTIONet's Jensen-Shannon-divergence-based cell distances in the archetypal metric cell space, and validates them against a permutation null, demonstrating a larger deviation from random expectation than Seurat-style HVG+PCA Euclidean distance.

## When to use

Use when benchmarking whether a cell-similarity measure captures same-type cell similarity more strongly than a baseline such as HVG+PCA Euclidean distance, with permutation-labeled null models on PBMC data.

## When NOT to use

- You only need a single-cell embedding or visualization rather than a quantitative similarity benchmark against a permutation null
- The dataset lacks a marker panel or reliable cell-type labels, so same-type groupings needed for intra-type similarity scoring cannot be defined
- Cells are already represented by archetype weights or a precomputed metric space and no comparative baseline evaluation is requested

## Inputs

- 10x Genomics PBMC count matrix (pbmc_10k_v3.h5, read via sc.read_10x_h5)
- PBMC_Monaco2019_12celltypes marker panel (loaded via an.tl.load_markers)
- Per-cell type annotations from an.tl.annotate_cells_using_markers
- Seurat-style HVG + PCA coordinate matrix for the Euclidean-distance baseline

## Outputs

- Intra-cell-type JSD-based similarity score distributions in the ACTIONet archetypal space
- Permutation-null deviation-from-random-expectation distributions per cell type for both methods
- Side-by-side deviation distribution plot (fig4e_deviation_distributions.png, matching Fig. 4e)

## How to apply

Load the 10x PBMC matrix pbmc_10k_v3.h5 with scanpy (sc.read_10x_h5) and make variable names unique with join='.'. Filter cells and genes with an.pp.filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), then apply sc.pp.normalize_total library-size normalization and sc.pp.log1p. Run an.pp.reduce_kernel for randomized SVD-based low-rank reduction and an.run_ACTIONet to build the archetypal-based metric cell space where cell distances can be measured. Assign per-cell type labels with the PBMC_Monaco2019_12celltypes marker panel via an.tl.load_markers and an.tl.annotate_cells_using_markers, then compute intra-cell-type similarity scores using ACTIONet's JSD-based distances while in parallel computing baseline intra-cell-type Euclidean distances on Seurat-style HVG + PCA coordinates. Build null models by repeatedly permuting cell-type labels and compute each method's observed-vs-null deviation per cell type, then plot ACTIONet similarity versus HVG+PCA Euclidean distance deviations side by side as in Fig. 4e.

## Related tools

- **ACTIONet (Python package)** (Provides an.pp.filter_adata, an.pp.reduce_kernel, an.run_ACTIONet, an.tl.load_markers, and an.tl.annotate_cells_using_markers; its archetypal metric cell space supplies the JSD-based cell distances being scored) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (Reads the 10x h5 matrix (sc.read_10x_h5) and performs library-size normalization (sc.pp.normalize_total) and log transformation (sc.pp.log1p) before ACTIONet reduction)

## Examples

```
markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')
```

## Evaluation signals

- ACTIONet JSD-based similarity distribution shows systematically larger positive deviation from the permutation null than the HVG+PCA Euclidean-distance baseline
- Per cell type, observed intra-type JSD similarity is significantly higher than the permuted-label null expectation
- The deviation distributions for ACTIONet similarity and Euclidean distance are plotted side by side and qualitatively match the published Fig. 4e pattern
- Preprocessing invariants hold: variable names made unique with join='.', min_cells_per_feature=0.01 / min_features_per_cell=1000 filtering applied before normalize_total and log1p

## Limitations

- Validation is qualitative (visual agreement with Fig. 4e) rather than a stated statistical significance threshold; the article text provided does not specify p-value criteria for the permutation test
- No software or tool versions are stated for ACTIONet, R, Python, UMAP, or Leiden implementations (CMake 3.19 is the only version given)
- Datasets are cited only by reference numbers without accessions or download links

## Evidence

- [discussion] Archetypal metric space enables JSD distance scoring: "an archetypal-based metric cell space construction that enables measuring cell distances"
- [methods] Marker panel loading call: "markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')"
- [methods] Filtering thresholds before normalization: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [discussion] Randomized reduction step rationale: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [readme] Continuous state justifies similarity over discrete labels: "Because every cell carries a full vector of archetype weights rather than a single label"
