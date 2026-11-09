---
name: singlecellhaystack-differential-activity-execution
description: Use when you need clustering-independent detection of differentially
  active features (e.g. spatially variable genes in 2D spatial coordinates, trajectory
  DEGs, pathway activities) from continuous single-cell or spatial expression data.
when_to_use_negative:
- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be
  easily conducted with the current version
- 'Fast spatial transcriptomics ranking is required and runtime is critical: SPARK-X
  is faster, and singleCellHaystack has relatively longer runtimes despite better
  sensitivity'
- Results need to be integrated into cluster-oriented scRNA-seq analyses, which is
  not straightforward with singleCellHaystack's cluster-independent output
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3518
tools:
- name: singleCellHaystack (R package)
  role: Primary R implementation (v1.0.0+, CRAN) for predicting differentially active
    features from continuous activity in any 1D/2D/>2D input space
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack-py
  role: Python implementation (PyPI v0.0.5) enabling efficient identification of DAFs
    for atlas-level datasets with millions of cells; scanpy-compatible via hs.haystack(adata,
    basis='pca')
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: scanpy
  role: Loads and processes AnnData (.h5ad) expression matrices with attached spatial/latent
    coordinates as haystack input
- name: SPARK-X
  role: Fast spatial transcriptomics method used as the comparison baseline for evaluating
    singleCellHaystack SVG rankings
- name: satijalab/seurat-wrappers
  role: Community wrapper providing singleCellHaystack access within Seurat workflows
  repo: https://github.com/satijalab/seurat-wrappers
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:aa1c4271f2fb6f6f8638f89849aa977d8ca6f789ed7ec1ffe77ddd4504ccf110
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R
  - SPARK-X
  - satijalab/seurat-wrappers
  - scanpy
  - singleCellHaystack
  - singleCellHaystack (R package)
  - singleCellHaystack-py
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# singlecellhaystack-differential-activity-execution

## Summary

Run singleCellHaystack (v1.0.0+ R or v0.0.5 Python) to predict differentially active features (genes, accessibility, spatial DEGs) from continuous activity levels over 1D/2D/3D input coordinates using Kullback-Leibler Divergence, without clustering-based comparisons.

## When to use

Use when you need clustering-independent detection of differentially active features (e.g. spatially variable genes in 2D spatial coordinates, trajectory DEGs, pathway activities) from continuous single-cell or spatial expression data.

## When NOT to use

- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted with the current version
- Fast spatial transcriptomics ranking is required and runtime is critical: SPARK-X is faster, and singleCellHaystack has relatively longer runtimes despite better sensitivity
- Results need to be integrated into cluster-oriented scRNA-seq analyses, which is not straightforward with singleCellHaystack's cluster-independent output

## Inputs

- AnnData (.h5ad) expression matrix with cell/sample annotations (scanpy)
- Continuous feature activity matrix: RNA or protein expression, chromatin accessibility, or module scores (sparse matrices supported)
- Input coordinates per cell/sample: 1D trajectory (pseudotime), 2-3D spatial coordinates (e.g. Slide-seqV2 puck bead positions), or higher-dimensional latent spaces (e.g. PCA)
- Optional gene sets for predicting differential activity of biological pathways

## Outputs

- Per-gene (per-feature) differential activity statistics and ranks
- Top-ranked features table, e.g. res.top_features(n=10)
- Gene clusters from cluster_genes (R only; hierarchical clustering method)
- Comparison tables for cross-method evaluation, e.g. svg_rank_comparison.csv with per-gene ranks and agreement flags

## How to apply

Load the expression matrix into an AnnData object (scanpy) or Seurat/SingleCellExperiment equivalent, attaching coordinates such as 2D spatial puck positions, PCA embeddings, or 1D pseudotime as the input space. Run the haystack function against the chosen basis (e.g. hs.haystack(adata, basis='pca') in Python, or the R package from CRAN at version 1.0.0+), which models continuous activity with splines whose flexibility is chosen by cross-validation and ranks features by non-random positioning of activity patterns using Kullback-Leibler Divergence. Because the method uses continuous activity levels instead of a hard binary detection threshold, it avoids overestimating the significance of modest expression differences. Retrieve ranked results with res.top_features(n=10) or the equivalent R result object, and optionally apply cluster_genes (hierarchical clustering; note this is not yet implemented in the Python beta). For SVG benchmarks, extract per-gene ranks and compare against other methods such as SPARK-X, writing comparison tables like svg_rank_comparison.csv with per-gene ranks and agreement flags.

## Related tools

- **singleCellHaystack (R package)** (Primary R implementation (v1.0.0+, CRAN) for predicting differentially active features from continuous activity in any 1D/2D/>2D input space) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack-py** (Python implementation (PyPI v0.0.5) enabling efficient identification of DAFs for atlas-level datasets with millions of cells; scanpy-compatible via hs.haystack(adata, basis='pca')) — https://github.com/ddiez/singleCellHaystack-py
- **scanpy** (Loads and processes AnnData (.h5ad) expression matrices with attached spatial/latent coordinates as haystack input)
- **SPARK-X** (Fast spatial transcriptomics method used as the comparison baseline for evaluating singleCellHaystack SVG rankings)
- **satijalab/seurat-wrappers** (Community wrapper providing singleCellHaystack access within Seurat workflows) — https://github.com/satijalab/seurat-wrappers

## Examples

```
import scanpy as sc; import singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Reproducing the documented Slide-seqV2 mouse hippocampus (SCP815) SVG ranks: singleCellHaystack ranks Plp1=20, Mal=55, Zfhx3=378, Cox4i1=1123; flag mismatched rows if ranks diverge
- res.top_features(n=10) returns a ranked feature list rather than cluster-comparison p-values, confirming cluster-independent execution
- Cross-validation runs without error during the spline-flexibility modeling step, indicating the continuous-activity modeling pipeline completed
- Results show higher sensitivity (top rankings) for spatially patterned genes like Plp1 and Mal relative to SPARK-X (Plp1: 20th vs 1139th; Mal: 55th vs 1543rd), consistent with the paper's reported sensitivity advantage

## Limitations

- The Python implementation is in beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available
- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted
- Relatively longer runtimes compared to SPARK-X for spatial transcriptomics, despite better sensitivity
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- Input data with a hard binary detection threshold (pre-v1.0 'binary' branch behavior) can overestimate significance of modest expression differences

## Evidence

- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space: "`singleCellHaystack` uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of
samples that are non-randomly positioned inside any input space."
- [abstract] it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [methods] res = hs.haystack(adata, basis="pca"): "res = hs.haystack(adata, basis="pca")"
- [methods] This repository contains a python implementation of singleCellHaystack (version >= 1.0.0): "This repository contains a python implementation of [singleCellHaystack](https://github.com/alexisvdb/singleCellHaystack) (version >= 1.0.0)."
- [discussion] the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression: "the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression"
- [readme] Python implementation is beta and missing features: "This package is currently in beta. The most important functionality in the R package works, but some features are not yet available."
