---
name: runtime-comparison-visualization
description: Use when comparing computational scalability of several feature-prediction
  or SVG methods across multiple spatial transcriptomics datasets, and you need per-method
  elapsed wall-clock times rendered as a comparison figure.
when_to_use_negative:
- When the goal is accuracy/sensitivity of SVG detection rather than runtime — use
  cross-method ranking correlations or detection benchmarks instead
- When inputs differ across methods (e.g., different feature subsets or full gene
  sets per method) — runtime comparisons on non-identical inputs are not interpretable
- When multi-condition differential comparison is the goal — the paper notes comparisons
  between multiple conditions (e.g., wild-type and knockout) cannot easily be conducted
  with singleCellHaystack
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: singleCellHaystack
  role: Benchmarked SVG/DAF prediction method run on spatial coordinates; one of the
    slower methods in the runtime comparison
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python package)
  role: Python implementation of singleCellHaystack usable via scanpy/anndata inputs;
    pip-installable for the benchmark harness
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: SPARK-X
  role: Competing SVG method expected to be the fastest in the runtime comparison
- name: Seurat FindSpatiallyVariableFeatures
  role: Competing method run twice (Moran's I and mark variogram) via Seurat; SeuratWrappers
    provides extension installation
  repo: https://github.com/satijalab/seurat-wrappers
- name: scanpy
  role: Loads h5ad count matrices and coordinates and restricts to top 1000 HVGs before
    benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:e572d52c3e3231453c9669ff4f4f5dcba0fd7cbb5db658bfb255c2d5095dbd9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - SPARK-X
  - Seurat FindSpatiallyVariableFeatures
  - scanpy
  - singleCellHaystack
  - singleCellHaystack (Python package)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# runtime-comparison-visualization

## Summary

A benchmarking skill that measures wall-clock runtimes of competing spatially-variable-gene (SVG) prediction methods (singleCellHaystack, SPARK-X, SPARK, Seurat FindSpatiallyVariableFeatures with Moran's I and mark variogram, MERINGUE, Giotto binSpect kmeans/rank) on identically restricted inputs (e.g., top 1000 HVGs from MERFISH, Visium, Slide-seqV2, HDST), then aggregates runtimes into a Fig 2A-style comparison chart.

## When to use

Use when comparing computational scalability of several feature-prediction or SVG methods across multiple spatial transcriptomics datasets, and you need per-method elapsed wall-clock times rendered as a comparison figure.

## When NOT to use

- When the goal is accuracy/sensitivity of SVG detection rather than runtime — use cross-method ranking correlations or detection benchmarks instead
- When inputs differ across methods (e.g., different feature subsets or full gene sets per method) — runtime comparisons on non-identical inputs are not interpretable
- When multi-condition differential comparison is the goal — the paper notes comparisons between multiple conditions (e.g., wild-type and knockout) cannot easily be conducted with singleCellHaystack

## Inputs

- Spatial transcriptomics count matrices and spatial coordinates for MERFISH, Visium, Slide-seqV2 and HDST datasets (h5ad files readable via scanpy read_h5ad)
- Per-dataset feature restriction to the top 1000 highly variable genes
- Per-method run configuration: single CPU core per method

## Outputs

- Per-method elapsed wall-clock runtimes for each dataset
- Fig 2A-style runtime comparison bar chart aggregating runtimes across the four platforms
- Optional: pairwise Spearman rank correlations between SVG method rankings per dataset

## How to apply

Load each dataset's count matrix and spatial coordinates with scanpy/anndata (sc.read_h5ad) and restrict the feature set to the top 1000 highly variable genes so all methods see identical inputs. Run each method pinned to a single CPU core and record elapsed wall-clock time per dataset: singleCellHaystack on the spatial coordinates, SPARK-X, Seurat FindSpatiallyVariableFeatures twice (method='moran'I' and mark variogram), MERINGUE, and Giotto binSpect twice (kmeans and rank). Aggregate per-method runtimes across the four datasets and render a Fig 2A-style bar chart of runtime vs method, optionally faceted by platform. As a sanity check that speed comparisons are meaningful, also compute pairwise Spearman rank correlations between the methods' SVG rankings per dataset; expect roughly 0.70-0.95. Judge success by whether the reproduced chart shows SPARK-X as generally the fastest method, followed by singleCellHaystack, consistent with the paper's statement that 'For spatial transcriptomics the fastest method available is SPARK-X.'

## Related tools

- **singleCellHaystack** (Benchmarked SVG/DAF prediction method run on spatial coordinates; one of the slower methods in the runtime comparison) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python package)** (Python implementation of singleCellHaystack usable via scanpy/anndata inputs; pip-installable for the benchmark harness) — https://github.com/ddiez/singleCellHaystack-py
- **SPARK-X** (Competing SVG method expected to be the fastest in the runtime comparison)
- **Seurat FindSpatiallyVariableFeatures** (Competing method run twice (Moran's I and mark variogram) via Seurat; SeuratWrappers provides extension installation) — https://github.com/satijalab/seurat-wrappers
- **scanpy** (Loads h5ad count matrices and coordinates and restricts to top 1000 HVGs before benchmarking)

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Reproduced runtime bar chart shows SPARK-X as generally the fastest method across the four spatial platforms, matching the paper's claim
- singleCellHaystack ranks second-fastest after SPARK-X in the aggregated runtime comparison
- Each method was run on the identical top-1000-HVG feature set of each dataset with one CPU core, so timing differences are attributable to method, not inputs
- Pairwise Spearman rank correlations between methods' SVG rankings fall in the 0.70-0.95 range reported in the article, confirming the compared outputs are comparable
- All methods either complete or fail reproducibly; most methods scaling poorly or failing on larger datasets (HDST, Slide-seqV2) should be recorded rather than silently dropped

## Limitations

- Most competing methods scaled poorly or failed on the larger spatial datasets, so some runtime cells may be missing or censored
- singleCellHaystack has relatively longer runtimes compared to SPARK-X for spatial transcriptomics ('The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes.')
- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted with singleCellHaystack, limiting what the benchmarked methods can be asked to do
- The Python singleCellHaystack package is in beta; features such as weights.advanced.Q, seeding grid points, and hierarchical clustering for cluster_genes are not yet available

## Evidence

- [discussion] For spatial transcriptomics the fastest method available is SPARK-X.: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes.: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [abstract] Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells.: "Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells"
- [readme] Readme example invocation with scanpy and haystack: "adata = sc.read_h5ad("data.h5ad")

[... process adata object ...]

res = hs.haystack(adata, basis="pca")"
