---
name: single-cell-differential-activity-prediction
description: Use when you have scRNA-seq, scATAC-seq, spatial transcriptomics, or
  trajectory data in h5ad/Seurat form with an embedding or coordinate space (PCA,
  pseudotime, 1D-3D spatial) and need clustering-independent detection of features
  with non-random activity patterns.
when_to_use_negative:
- Comparing multiple conditions (e.g., wild-type vs knockout) — the article states
  such comparisons 'cannot be easily conducted' and multi-condition methods are future
  work
- Analyses that need DEG results integrated into a cluster-oriented scRNA-seq workflow
  — results are 'not as straightforward to incorporate' compared to cluster-based
  approaches
- Spatial transcriptomics where runtime is the overriding constraint — SPARK-X is
  faster, though singleCellHaystack offers better sensitivity
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack (R, v1.0.0+)
  role: 'Core method: predicts differentially active features from continuous activity
    over PCA/spatial/trajectory coordinates; legacy binary v0.3.x available on the
    ''binary'' branch'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack-py (v0.0.5)
  role: Python implementation for atlas-level datasets with millions of cells; exposes
    hs.haystack and res.top_features
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: Python / scanpy
  role: Reads h5ad input data and provides PCA embeddings used as the basis for haystack
- name: R
  role: Environment for the full-featured CRAN package including cluster_genes and
    advanced sampling options
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: SPARK-X
  role: Alternative spatial transcriptomics tool referenced for runtime comparison
    (faster, less sensitive)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:3122525a954241350a2ce5303a40c728521f64818bd9237c2834047dd20e8130
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - Python / scanpy
  - R
  - SPARK-X
  - singleCellHaystack (R, v1.0.0+)
  - singleCellHaystack-py (v0.0.5)
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-differential-activity-prediction

## Summary

Predict differentially active features (DEGs, spatial DEGs, accessible chromatin, pathway activities) in single-cell or spatial genomics data using singleCellHaystack v1.0+, which models continuous activity levels with cross-validated splines and Kullback-Leibler divergence instead of cluster comparisons or hard binary detection thresholds.

## When to use

Use when you have scRNA-seq, scATAC-seq, spatial transcriptomics, or trajectory data in h5ad/Seurat form with an embedding or coordinate space (PCA, pseudotime, 1D-3D spatial) and need clustering-independent detection of features with non-random activity patterns.

## When NOT to use

- Comparing multiple conditions (e.g., wild-type vs knockout) — the article states such comparisons 'cannot be easily conducted' and multi-condition methods are future work
- Analyses that need DEG results integrated into a cluster-oriented scRNA-seq workflow — results are 'not as straightforward to incorporate' compared to cluster-based approaches
- Spatial transcriptomics where runtime is the overriding constraint — SPARK-X is faster, though singleCellHaystack offers better sensitivity

## Inputs

- h5ad-format single-cell dataset readable by scanpy (e.g., pbmc3k, atlas-scale data with millions of cells)
- First 20 principal components or any 1D/2D/3D or higher-dimensional coordinate space (pseudotime, Visium/Slide-seq V2 spatial coordinates)
- Continuous feature activity matrices: RNA or protein expression, chromatin accessibility, or gene-set/module scores
- For legacy comparisons: a binarized detection matrix

## Outputs

- Per-feature significance p-values ranked by non-random activity in the input space
- Top-ranked feature lists via res.top_features(n=10)
- Gene clusters via cluster_genes (hierarchical clustering, R implementation)
- Runtime metrics for benchmarking against the binary v0.3.x version

## How to apply

Load the data (e.g., adata = sc.read_h5ad('data.h5ad') with scanpy) and compute an embedding such as the first 20 principal components, which serves as the input coordinate space; singleCellHaystack accepts 1D trajectories, 2-3D spatial coordinates, and higher-dimensional latent spaces. Run hs.haystack(adata, basis='pca') to model each feature's continuous activity (not binarized detection) over the coordinate space using splines whose flexibility is chosen by cross-validation, and rank features by Kullback-Leibler divergence-based significance. Inspect results with res.top_features(n=10) and optionally group genes with cluster_genes (R only). For comparisons against the legacy version, install the R binary branch via remotes::install_github('alexisvdb/singleCellHaystack@binary') and correlate log p-values across genes with Spearman rank correlation; the continuous model typically yields concordant rankings (average Spearman ~0.92 across 119 Tabula Muris / Mouse Cell Atlas datasets) with substantial runtime savings (~37.8% mean, up to 10x).

## Related tools

- **singleCellHaystack (R, v1.0.0+)** (Core method: predicts differentially active features from continuous activity over PCA/spatial/trajectory coordinates; legacy binary v0.3.x available on the 'binary' branch) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack-py (v0.0.5)** (Python implementation for atlas-level datasets with millions of cells; exposes hs.haystack and res.top_features) — https://github.com/ddiez/singleCellHaystack-py
- **Python / scanpy** (Reads h5ad input data and provides PCA embeddings used as the basis for haystack)
- **R** (Environment for the full-featured CRAN package including cluster_genes and advanced sampling options) — https://github.com/alexisvdb/singleCellHaystack
- **SPARK-X** (Alternative spatial transcriptomics tool referenced for runtime comparison (faster, less sensitive))

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Spearman correlation between log p-values of the binary (v0.3.x) and continuous versions across all genes should average ~0.92 over 119 Tabula Muris / Mouse Cell Atlas datasets (lung dataset: 0.95)
- Discrepant genes in the Tabula Muris lung comparison should include Itm2b and Lyz2 via res.top_features rankings
- Continuous version should show a mean ~37.8% runtime reduction (up to 10x) versus the binary version
- Top features returned by res.top_features(n=10) should be genes with non-random activity patterns in the chosen basis; empty or uniform rankings indicate a mis-specified basis or embedding
- Python results should match R results for the supported subset (weights.advanced.Q, seeding grid, and cluster_genes are missing in the beta Python package)

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted; multi-condition support is planned future work
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- Longer runtimes than SPARK-X for spatial transcriptomics, though sensitivity is better
- The Python package is beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet implemented
- The original binary version's hard threshold ignores subtle expression differences and can overestimate significance of modest expression differences

## Evidence

- [abstract] singleCellHaystack now accepts continuous features (RNA or protein expression, chromatin accessibility, module scores) from single-cell, spatial, and bulk genomics data: "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
- [abstract] Performance improved with up to ten times reduction in computational time: "Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells"
- [intro] Cross-validation for spline flexibility: "Second, it uses cross-validation for choosing a suitable flexibility of splines during its modeling steps"
- [methods] Core workflow: read h5ad and run haystack on PCA basis: "adata = sc.read_h5ad("data.h5ad")"
- [discussion] Continuous over binary rationale: "the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression"
- [readme] Install legacy binary version from GitHub branch: [evidence span withheld — anchor status: external_doc]
- [readme] Method uses Kullback-Leibler divergence without clustering: "`singleCellHaystack` uses Kullback-Leibler
Divergence to find features that have patterns of activity in subsets of
samples that are non-randomly positioned inside any input space"
