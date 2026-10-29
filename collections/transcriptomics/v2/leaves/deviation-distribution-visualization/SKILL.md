---
name: deviation-distribution-visualization
description: Use when you have two competing cell-distance/similarity measures over
  the same labeled cells and need to quantify and plot which one separates same-type
  cells more strongly than random expectation (e.g., ACTIONet JSD similarity vs. HVG+PCA
  Euclidean distance).
when_to_use_negative:
- The two distance/similarity methods are not computed on the same set of cells with
  common cell-type labels — the permutation null must be comparable across methods.
- You lack a cell-type label source (marker panel or annotations), since per-cell-type
  deviation from the permutation null is the quantity being visualized.
- The goal is a clustering or annotation accuracy benchmark rather than assessing
  similarity-score enrichment over random label expectation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: ACTIONet
  role: Provides the archetypal metric cell space and Jensen-Shannon-divergence-based
    cell similarity whose observed-vs-null deviation is plotted
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: Reads the 10x h5 matrix (sc.read_10x_h5), makes variable names unique, and
    applies normalize_total/log1p preprocessing feeding both metric spaces
- name: Seurat (HVG + PCA pipeline)
  role: Reference implementation of the Euclidean-distance baseline computed on highly-variable-gene
    PCA coordinates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:4b215915019e62ec9f59b43c15c587e257377fa8c049bf2dbe6dcfe3ff1e201f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - Seurat (HVG + PCA pipeline)
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# deviation-distribution-visualization

## Summary

Visualizes, per cell type, how strongly a cell-similarity/distance method (e.g., ACTIONet's Jensen-Shannon-divergence-based similarity in the archetypal space) exceeds a permutation null relative to a baseline method (e.g., Seurat-style HVG+PCA Euclidean distance), reproducing the side-by-side deviation-distribution plot of Fig. 4e of the ACTIONet paper.

## When to use

Use when you have two competing cell-distance/similarity measures over the same labeled cells and need to quantify and plot which one separates same-type cells more strongly than random expectation (e.g., ACTIONet JSD similarity vs. HVG+PCA Euclidean distance).

## When NOT to use

- The two distance/similarity methods are not computed on the same set of cells with common cell-type labels — the permutation null must be comparable across methods.
- You lack a cell-type label source (marker panel or annotations), since per-cell-type deviation from the permutation null is the quantity being visualized.
- The goal is a clustering or annotation accuracy benchmark rather than assessing similarity-score enrichment over random label expectation.

## Inputs

- Raw 10x Genomics PBMC count matrix (pbmc_10k_v3.h5, read via scanpy)
- A marker panel (PBMC_Monaco2019_12celltypes) loaded with an.tl.load_markers for per-cell type annotation
- ACTIONet archetypal space (from an.pp.reduce_kernel + an.run_ACTIONet) and an HVG+PCA coordinate space (Seurat-style baseline)
- Repeatedly permuted cell-type labels forming the null model

## Outputs

- Per-cell-type observed-vs-null deviation distributions for ACTIONet JSD-based similarity and for the Euclidean-distance baseline
- Side-by-side deviation-distribution plot saved as fig4e_deviation_distributions.png (reproducing Fig. 4e)

## How to apply

First build both metric spaces: preprocess the raw 10x matrix (sc.read_10x_h5 with var_names_make_unique(join='.')), filter with an.pp.filter_adata(min_cells_per_feature=0.01, min_features_per_cell=1000), then sc.pp.normalize_total + sc.pp.log1p, run an.pp.reduce_kernel and an.run_ACTIONet to obtain the archetypal metric cell space; in parallel compute HVG+PCA Euclidean distances Seurat-style. Assign cell-type labels with an.tl.load_markers('PBMC_Monaco2019_12celltypes') and an.tl.annotate_cells_using_markers. Compute intra-cell-type scores under each metric, then permute cell-type labels repeatedly to build null distributions and compute the observed-vs-null deviation from random expectation per cell type per method. Finally plot the deviation distributions for ACTIONet similarity and the Euclidean baseline side by side (as in Fig. 4e) and save to fig4e_deviation_distributions.png.

## Related tools

- **ACTIONet** (Provides the archetypal metric cell space and Jensen-Shannon-divergence-based cell similarity whose observed-vs-null deviation is plotted) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (Reads the 10x h5 matrix (sc.read_10x_h5), makes variable names unique, and applies normalize_total/log1p preprocessing feeding both metric spaces)
- **Seurat (HVG + PCA pipeline)** (Reference implementation of the Euclidean-distance baseline computed on highly-variable-gene PCA coordinates)

## Examples

```
import scanpy as sc; adata = sc.read_10x_h5('pbmc_10k_v3.h5'); adata.var_names_make_unique(join='.'); markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')
```

## Evaluation signals

- The ACTIONet JSD-based similarity distribution shows systematically larger positive deviation from the permutation null than the HVG+PCA Euclidean-distance baseline, qualitatively matching Fig. 4e.
- Observed intra-cell-type ACTIONet similarity scores are significantly higher for cells of the same type than random expectation under label permutation.
- Deviation values are computed per cell type (e.g., the 12 Monaco-2019 PBMC cell types), not pooled across all cells.
- The output figure (fig4e_deviation_distributions.png) displays ACTIONet similarity and Euclidean-distance deviation distributions side by side.

## Limitations

- The comparison is qualitative (deviation distribution patterns, as in Fig. 4e) rather than a single scalar significance statistic per method.
- The baseline is a specific Seurat-style HVG + PCA + Euclidean pipeline; deviations reflect that particular parameterization, not all standard-tool distance measures.
- No software/tool versions are stated for ACTIONet, scanpy, or the permutation protocol, so exact reproducibility depends on package versions at run time.

## Evidence

- [methods] article_methods_marker_loading: "markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')"
- [methods] article_methods_filtering: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [discussion] article_discussion_metric_space: "an archetypal-based metric cell space construction that enables measuring cell distances"
- [abstract] article_abstract_superiority: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
- [readme] readme_continuous_state: "Because every cell carries a full vector of archetype weights rather than a single label, downstream analysis works on continuous cell state"
