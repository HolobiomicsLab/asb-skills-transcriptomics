---
name: runtime-benchmarking-across-methods
description: Use when comparing computational runtime or scalability of competing
  spatial-omics feature-detection methods across multiple platforms, or when verifying
  that a new method's runtime matches published benchmark claims.
when_to_use_negative:
- The goal is detecting differentially active features for its own accuracy/sensitivity
  rather than benchmarking method runtime — use singleCellHaystack directly instead
- Comparison involves multiple biological conditions (e.g., wild-type vs knockout)
  — singleCellHaystack-based pipelines cannot easily conduct such comparisons
- Only one platform and one method are involved and no cross-method or cross-dataset
  comparison is required
tools:
- name: singleCellHaystack (Python)
  role: Benchmarked method run via hs.haystack on spatial coordinates; also the reference
    fast method after SPARK-X
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: singleCellHaystack (R)
  role: Benchmarked R implementation (>= 1.0.0) from CRAN/GitHub
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: SPARK-X
  role: Benchmarked competitor; generally the fastest method in the runtime comparison
- name: Seurat FindSpatiallyVariableFeatures
  role: Benchmarked competitor run twice (Moran's I and mark variogram); available
    via SeuratWrappers
  repo: satijalab/seurat-wrappers
- name: scanpy
  role: Used to load .h5ad datasets and select the top 1000 HVGs
- name: MERINGUE
  role: Benchmarked competitor for spatial transcriptomics
- name: Giotto binSpect
  role: Benchmarked competitor run twice (kmeans and rank variants)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:e11d314dd22cf73b12fa329e4957c623d00ef0ea33c8f9adecc74dd928abf566
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ddiez/singleCellHaystack-py
  tools:
  - Giotto binSpect
  - MERINGUE
  - Python
  - SPARK-X
  - Seurat FindSpatiallyVariableFeatures
  - scanpy
  - singleCellHaystack (Python)
  - singleCellHaystack (R)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# runtime-benchmarking-across-methods

## Summary

A benchmarking skill for comparing wall-clock runtimes of spatially-variable-gene (SVG) prediction methods — singleCellHaystack, SPARK, SPARK-X, Seurat FindSpatiallyVariableFeatures (Moran's I, mark variogram), MERINGUE, and Giotto binSpect (kmeans, rank) — on the top 1000 HVGs of MERFISH, Visium, Slide-seqV2 and HDST datasets, and for ranking method agreement via pairwise Spearman correlations.

## When to use

Use when comparing computational runtime or scalability of competing spatial-omics feature-detection methods across multiple platforms, or when verifying that a new method's runtime matches published benchmark claims.

## When NOT to use

- The goal is detecting differentially active features for its own accuracy/sensitivity rather than benchmarking method runtime — use singleCellHaystack directly instead
- Comparison involves multiple biological conditions (e.g., wild-type vs knockout) — singleCellHaystack-based pipelines cannot easily conduct such comparisons
- Only one platform and one method are involved and no cross-method or cross-dataset comparison is required

## Inputs

- anndata/.h5ad count matrices with spatial coordinates for MERFISH, Visium, Slide-seqV2 and HDST datasets
- Top 1000 highly variable genes per dataset
- Spatial coordinate matrices (2D/3D) used as input space for each method

## Outputs

- Per-method wall-clock runtime table (seconds) per dataset and method
- Fig 2A-style runtime comparison bar chart aggregated across datasets
- Pairwise Spearman rank correlation matrix of SVG rankings across method pairs per dataset

## How to apply

Load each spatial dataset's count matrix and coordinates with scanpy's read_h5ad into an anndata object and restrict to the top 1000 highly variable genes. Run each method on identical inputs with one CPU core, recording elapsed wall-clock time: hs.haystack from the Python singleCellHaystack package, SPARK-X, Seurat's FindSpatiallyVariableFeatures with both method='moran'I' and method='markvariot', MERINGUE, and Giotto binSpect with kmeans and rank variants. Aggregate per-method runtimes across the four platforms (MERFISH, Visium, Slide-seqV2, HDST) and render a Fig 2A-style runtime bar chart. As a complementary agreement check, compute pairwise Spearman rank correlations between the SVG rankings of all method pairs per dataset. Decision rationale: the paper states that for spatial transcriptomics the fastest method available is SPARK-X, followed by singleCellHaystack, and reports cross-method Spearman correlations typically in the 0.70–0.95 range.

## Related tools

- **singleCellHaystack (Python)** (Benchmarked method run via hs.haystack on spatial coordinates; also the reference fast method after SPARK-X) — https://github.com/ddiez/singleCellHaystack-py
- **singleCellHaystack (R)** (Benchmarked R implementation (>= 1.0.0) from CRAN/GitHub) — https://github.com/alexisvdb/singleCellHaystack
- **SPARK-X** (Benchmarked competitor; generally the fastest method in the runtime comparison)
- **Seurat FindSpatiallyVariableFeatures** (Benchmarked competitor run twice (Moran's I and mark variogram); available via SeuratWrappers) — satijalab/seurat-wrappers
- **scanpy** (Used to load .h5ad datasets and select the top 1000 HVGs)
- **MERINGUE** (Benchmarked competitor for spatial transcriptomics)
- **Giotto binSpect** (Benchmarked competitor run twice (kmeans and rank variants))

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Reproduced runtime bar chart must show SPARK-X as generally the fastest method, consistent with the paper's statement that 'for spatial transcriptomics the fastest method available is SPARK-X'
- singleCellHaystack should rank second-fastest in runtime among the compared methods
- Cross-method pairwise Spearman rank correlations should fall in the typical 0.70–0.95 range reported in the article
- All methods must run on identical inputs (top 1000 HVGs per platform) restricted to a single CPU core for a fair runtime comparison
- Larger datasets (e.g., HDST, Slide-seqV2) should reproduce the finding that most methods scale poorly or fail while SPARK-X remains fast

## Limitations

- singleCellHaystack has relatively longer runtimes than SPARK-X for spatial transcriptomics, making it 'the best alternative, despite relatively longer runtimes'
- Most compared methods scale poorly or fail on larger spatial datasets, so runtime conclusions are platform- and size-dependent
- The Python singleCellHaystack package is in beta; some R features are missing (weights.advanced.Q, seeding grid method, hierarchical clustering for cluster_genes)
- Cross-method rank agreement (Spearman ~0.70–0.95) is only typical, not guaranteed, and varies by dataset

## Evidence

- [discussion] SPARK-X fastest finding: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] Runtime vs sensitivity trade-off: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [discussion] Python implementation scalability: "Furthermore, our new Python implementation enables efficient identification of DAFs for atlas level datasets with millions of cells."
- [readme] README usage example: "adata = sc.read_h5ad("data.h5ad")"
- [readme] README install/invocation: "This repository contains a Python implementation of [singleCellHaystack](https://github.com/alexisvdb/singleCellHaystack) (version >= 1.0.0)."
