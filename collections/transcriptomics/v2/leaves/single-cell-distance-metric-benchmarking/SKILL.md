---
name: single-cell-distance-metric-benchmarking
description: Use when you need to validate that a proposed cell-to-cell similarity/distance
  measure captures same-cell-type affinity more strongly than a conventional pipeline
  baseline (e.g., HVG+PCA Euclidean distance), by quantifying deviation from a permutation
  null.
when_to_use_negative:
- When you have no independent reference cell-type labels or marker panel to define
  'same cell type' groups — the permutation test cannot be constructed without a grouping
  to shuffle
- When the goal is clustering or annotation itself rather than validating that a distance
  metric outperforms a baseline on same-type affinity
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: ACTIONet (Python package)
  role: computes JSD-based cell distances in the archetypal metric cell space (an.pp.reduce_kernel,
    an.run_ACTIONet) and loads/annotates the PBMC_Monaco2019_12celltypes marker panel
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: reads the 10x HDF5 matrix (sc.read_10x_h5) and performs normalize_total/log1p
    preprocessing and the HVG+PCA baseline for the Euclidean-distance comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:69b009d555de3e98ca8148a3444fca739364ceb296be827b6597df7d11d11465
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - scanpy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-distance-metric-benchmarking

## Summary

Benchmark a single-cell similarity metric (ACTIONet's Jensen-Shannon-divergence-based distances in archetypal space) against a standard baseline (Seurat-style HVG + PCA + Euclidean distance) using permutation tests that compare each metric's observed intra-cell-type scores to a label-shuffled null, as in Fig. 4e of the ACTIONet paper.

## When to use

Use when you need to validate that a proposed cell-to-cell similarity/distance measure captures same-cell-type affinity more strongly than a conventional pipeline baseline (e.g., HVG+PCA Euclidean distance), by quantifying deviation from a permutation null.

## When NOT to use

- When you have no independent reference cell-type labels or marker panel to define 'same cell type' groups — the permutation test cannot be constructed without a grouping to shuffle
- When the goal is clustering or annotation itself rather than validating that a distance metric outperforms a baseline on same-type affinity

## Inputs

- 10x Genomics scRNA-seq count matrix in HDF5 format (e.g., pbmc_10x_v3.h5 / pbmc_10k_v3.h5 from 14,311-cell PBMC data)
- Reference marker panel (PBMC_Monaco2019_12celltypes) for cell-type label assignment
- Preprocessed/log-normalized AnnData object for computing the Seurat-style HVG + PCA + Euclidean baseline

## Outputs

- Per-cell-type intra-type similarity score distributions for ACTIONet JSD-based distances and for the Euclidean baseline
- Per-cell-type deviation-from-null statistics after label permutation
- Side-by-side deviation distribution plot (fig4e_deviation_distributions.png)

## How to apply

Preprocess the 10x PBMC matrix (pbmc_10k_v3.h5, read with scanpy's sc.read_10x_h5 and var_names_make_unique) by filtering with an.pp.filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), then sc.pp.normalize_total library-size normalization and sc.pp.log1p. Build the archetypal cell space with an.pp.reduce_kernel (randomized SVD low-rank reduction) and an.run_ACTIONet, and assign reference cell-type labels using the PBMC_Monaco2019_12celltypes marker panel via an.tl.load_markers and an.tl.annotate_cells_using_markers. For each method, compute intra-cell-type similarity: JSD-based cell distances in ACTIONet's archetypal space versus Euclidean distances on HVG+PCA coordinates (the Seurat-style baseline). Construct the null by repeatedly permuting cell-type labels, then compute the per-cell-type deviation of observed scores from the null expectation for each metric. Judge success by whether ACTIONet's JSD similarity shows systematically larger positive deviation from random expectation than the Euclidean baseline, matching the published Fig. 4e pattern; plot the deviation distributions side by side (fig4e_deviation_distributions.png).

## Related tools

- **ACTIONet (Python package)** (computes JSD-based cell distances in the archetypal metric cell space (an.pp.reduce_kernel, an.run_ACTIONet) and loads/annotates the PBMC_Monaco2019_12celltypes marker panel) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (reads the 10x HDF5 matrix (sc.read_10x_h5) and performs normalize_total/log1p preprocessing and the HVG+PCA baseline for the Euclidean-distance comparison)

## Examples

```
markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes'); labels = an.tl.annotate_cells_using_markers(adata, markers, directions, names); an.pp.reduce_kernel(adata); an.run_ACTIONet(adata)
```

## Evaluation signals

- ACTIONet JSD-based similarity distribution shows systematically larger positive deviation from the permutation null than the HVG+PCA Euclidean baseline, qualitatively matching Fig. 4e
- Permutation nulls are constructed from repeated cell-type label shuffles; observed-vs-null deviation must be computed per cell type, not pooled
- Preprocessing invariants: variables made unique with join='.', filter thresholds min_cells_per_feature=0.01 and min_features_per_cell=1000 applied before normalize_total and log1p
- Marker-based annotation (an.tl.annotate_cells_using_markers with PBMC_Monaco2019_12celltypes) must assign labels to every cell before intra-type distances are computed
- Both methods must be evaluated on the same set of cells/labels so the deviation comparison is apples-to-apples

## Limitations

- No software/tool versions are stated for ACTIONet, R, Python, UMAP, or Leiden implementations, so exact version reproducibility is not guaranteed
- The paper cites datasets only by reference numbers without accessions or download links; pbmc_10k_v3.h5 must be obtained separately
- Validation is qualitative (visual side-by-side deviation distributions as in Fig. 4e), not a formal statistical significance threshold
- Comparison is specific to PBMC data with a 12-cell-type Monaco 2019 marker panel; transfer to other tissues requires a new marker panel and revalidation

## Evidence

- [methods] perm test workflow preprocessing: "adata = sc.read_10x_h5('pbmc_10k_v3.h5')
adata.var_names_make_unique(join='.')"
- [methods] filter thresholds: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [methods] marker panel annotation: "markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')"
- [discussion] archetypal metric space rationale: "an archetypal-based metric cell space construction that enables measuring cell distances"
- [discussion] low-rank reduction: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [discussion] reproducibility caveat: [evidence span withheld — anchor status: not_found]
- [abstract] ACTIONet role: "We implement this concept in ACTIONet, a comprehensive framework that combines archetypal analysis and manifold learning"
