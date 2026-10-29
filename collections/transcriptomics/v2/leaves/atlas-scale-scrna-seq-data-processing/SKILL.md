---
name: atlas-scale-scrna-seq-data-processing
description: Use when you need clustering-independent DEG/DAF predictions on atlas-level
  scRNA-seq datasets (h5ad files, up to millions of cells), or to benchmark the continuous
  v1.0.0+ version against the binary v0.3.4 branch across many datasets.
when_to_use_negative:
- The analysis requires comparisons between multiple conditions (e.g., wild-type vs
  knockout) — this is a stated limitation with multi-condition methods only planned
  as future work
- Results must be directly integrated into a cluster-oriented scRNA-seq analysis —
  incorporation of singleCellHaystack results with cluster information is not yet
  supported
- Spatial transcriptomics where minimum runtime is critical — SPARK-X is faster, singleCellHaystack
  has relatively longer runtimes despite better sensitivity
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack-py
  role: Python implementation (PyPI v0.0.5) of singleCellHaystack >= 1.0.0 used to
    run hs.haystack on h5ad atlas datasets with basis='pca'
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: singleCellHaystack (R)
  role: R package from CRAN (v1.0.0+) for the continuous model; the @binary branch
    provides the legacy v0.3.4 binary-detection implementation for comparison runs
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: scanpy
  role: Used to read h5ad atlas datasets (sc.read_h5ad) and provide the PCA basis
    consumed by hs.haystack
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:322e3f69cf153f115c92b7d69997b1ef08b383f5748d9a25052d763a4678b017
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ddiez/singleCellHaystack-py
  tools:
  - Python
  - R
  - scanpy
  - singleCellHaystack (R)
  - singleCellHaystack-py
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# atlas-scale-scrna-seq-data-processing

## Summary

A clustering-independent workflow for predicting differentially active features (genes) in atlas-scale scRNA-seq data using singleCellHaystack v1.0.0+ (R/Python), which models continuous activity levels over PCA coordinates via Kullback-Leibler Divergence instead of a hard binary detection threshold.

## When to use

Use when you need clustering-independent DEG/DAF predictions on atlas-level scRNA-seq datasets (h5ad files, up to millions of cells), or to benchmark the continuous v1.0.0+ version against the binary v0.3.4 branch across many datasets.

## When NOT to use

- The analysis requires comparisons between multiple conditions (e.g., wild-type vs knockout) — this is a stated limitation with multi-condition methods only planned as future work
- Results must be directly integrated into a cluster-oriented scRNA-seq analysis — incorporation of singleCellHaystack results with cluster information is not yet supported
- Spatial transcriptomics where minimum runtime is critical — SPARK-X is faster, singleCellHaystack has relatively longer runtimes despite better sensitivity

## Inputs

- h5ad-format single-cell RNA-seq atlas files (e.g., 32 Tabula Muris + 87 Mouse Cell Atlas datasets, pbmc3k, MOCA 100k cells)
- First 20 principal components computed from the expression matrix as input coordinates
- For the legacy binary version: a binarized gene detection matrix

## Outputs

- Per-gene p-values for differential activity for every gene in each dataset
- Top-ranked features via res.top_features(n=10)
- Benchmark metrics: per-dataset Spearman correlation between binary and continuous log p-values, and wall-clock runtimes

## How to apply

Install the continuous implementation from PyPI (pip install singleCellHaystack, version 0.0.5) or CRAN (singleCellHaystack >= 1.0.0), and the legacy binary R version via remotes::install_github("alexisvdb/singleCellHaystack@binary"). Load each h5ad atlas dataset with scanpy (sc.read_h5ad) and compute/extract the first 20 principal components as the input coordinates for the method. Run hs.haystack(adata, basis="pca") to obtain per-gene p-values, then inspect res.top_features(n=10) for the top-ranked features. The continuous version uses cross-validation for spline flexibility, sparse matrices, and makes no strong assumptions about the input distribution, which enables scalability to millions of cells; benchmark against the binary version by computing Spearman correlation of log p-values per dataset (expected average ~0.92 across 119 Tabula Muris + Mouse Cell Atlas datasets, with ~37.8% mean runtime reduction).

## Related tools

- **singleCellHaystack-py** (Python implementation (PyPI v0.0.5) of singleCellHaystack >= 1.0.0 used to run hs.haystack on h5ad atlas datasets with basis='pca') — https://github.com/ddiez/singleCellHaystack-py
- **singleCellHaystack (R)** (R package from CRAN (v1.0.0+) for the continuous model; the @binary branch provides the legacy v0.3.4 binary-detection implementation for comparison runs) — https://github.com/alexisvdb/singleCellHaystack
- **scanpy** (Used to read h5ad atlas datasets (sc.read_h5ad) and provide the PCA basis consumed by hs.haystack)

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Average Spearman correlation of log p-values between binary and continuous versions across the 119 datasets is ~0.92
- Tabula Muris lung dataset reproduces Spearman 0.95, with Itm2b and Lyz2 surfacing as discrepant genes in res.top_features rankings
- Continuous version shows a mean ~37.8% wall-clock runtime reduction versus the binary version (up to tenfold reduction reported in the paper)
- res.top_features(n=10) returns a ranked feature list for every dataset without errors on atlas-scale inputs (e.g., MOFA/MOCA 100k cells)
- Pipeline runs end-to-end: read_h5ad -> hs.haystack(basis='pca') -> top_features for all 119 datasets

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- Relatively longer runtimes than SPARK-X for spatial transcriptomics, despite better sensitivity
- The Python package is in beta: weights.advanced.Q (formerly use.advanced.sampling), the 'seeding' grid-point method, and hierarchical clustering for cluster_genes are not yet available

## Evidence

- [abstract] Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells: "Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells"
- [intro] a Python implementation has been developed which enables the efficient application of singleCellHaystack to atlas level datasets with millions of cells: "a Python implementation has been developed which enables the efficient application of singleCellHaystack to atlas level datasets with millions of cells"
- [methods] res = hs.haystack(adata, basis="pca"): "res = hs.haystack(adata, basis="pca")"
- [readme] It does so without relying on clustering of samples into arbitrary clusters.: [evidence span withheld — anchor status: external_doc]
