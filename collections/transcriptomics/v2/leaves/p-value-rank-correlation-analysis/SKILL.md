---
name: p-value-rank-correlation-analysis
description: Use when you need to compare per-gene p-value rankings produced by two
  versions or variants of a feature-scanning method across multiple datasets, to quantify
  whether a methodological change (e.g., binary detection vs continuous activity)
  preserves DEG predictions.
when_to_use_negative:
- You need comparisons between multiple conditions (e.g., wild-type vs knockout) —
  the article states such comparisons cannot be easily conducted and are planned future
  work.
- Your goal is spatial DEG testing where SPARK-X-class speed is required — the article
  notes singleCellHaystack has 'relatively longer runtimes' than SPARK-X despite better
  sensitivity.
- You rely on hierarchical clustering of genes — the Python implementation lacks the
  cluster_genes hierarchical clustering method (beta-stage missing feature).
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack (R, binary branch v0.3.4)
  role: 'Reference variant: run on binarized detection matrix with basis=''pca'' to
    produce per-gene p-values to correlate against'
  repo: https://github.com/alexisvdb/singleCellHaystack/tree/binary
- name: singleCellHaystack (continuous, R >= 1.0.0 / Python 0.0.5)
  role: 'Test variant: run on the same first-20-PC coordinates to produce continuous-model
    per-gene p-values'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Python / scanpy
  role: Load h5ad atlas datasets, compute PCA coordinates, compute Spearman correlations
    of log p-values, and time runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:94a00f00e5238c9cd7b3f55cede79dffda4c4316a102bec3d1c6720c6dd45816
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python / scanpy
  - R
  - singleCellHaystack (R, binary branch v0.3.4)
  - singleCellHaystack (continuous, R >= 1.0.0 / Python 0.0.5)
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# p-value-rank-correlation-analysis

## Summary

Quantifies agreement between two per-feature significance-testing variants (here, binary v0.3.4 vs continuous v1.0.0+ singleCellHaystack) by computing Spearman rank correlation between their log p-values across all genes in each dataset, and benchmarks wall-clock runtimes.

## When to use

Use when you need to compare per-gene p-value rankings produced by two versions or variants of a feature-scanning method across multiple datasets, to quantify whether a methodological change (e.g., binary detection vs continuous activity) preserves DEG predictions.

## When NOT to use

- You need comparisons between multiple conditions (e.g., wild-type vs knockout) — the article states such comparisons cannot be easily conducted and are planned future work.
- Your goal is spatial DEG testing where SPARK-X-class speed is required — the article notes singleCellHaystack has 'relatively longer runtimes' than SPARK-X despite better sensitivity.
- You rely on hierarchical clustering of genes — the Python implementation lacks the cluster_genes hierarchical clustering method (beta-stage missing feature).

## Inputs

- scRNA-seq atlas datasets in h5ad format (e.g., Tabula Muris, Mouse Cell Atlas), read via sc.read_h5ad
- First 20 principal components per dataset as shared input coordinates (basis='pca')
- Binarized gene detection matrix (for the binary v0.3.4 implementation)
- Continuous expression/activity matrix (for the v1.0.0+ continuous implementation)

## Outputs

- Per-gene p-values from each singleCellHaystack version for every dataset
- Per-dataset Spearman rank correlation of log p-values across all genes
- Wall-clock runtimes per variant and mean percent runtime reduction
- Top-ranked features per dataset via res.top_features(n=10) for identifying discrepant genes

## How to apply

Install both method variants: the original binary singleCellHaystack (R, branch 'binary' of alexisvdb/singleCellHaystack, v0.3.4) via remotes::install_github("alexisvdb/singleCellHaystack@binary"), and the continuous version from CRAN (>= 1.0.0) or PyPI (pip install singleCellHaystack). Load each atlas dataset (32 Tabula Muris + 87 Mouse Cell Atlas, h5ad format) with scanpy and compute/extract the first 20 principal components as input coordinates. Run the binary implementation on the binarized detection matrix with basis='pca' and the continuous implementation on the same first-20-PC coordinates, so both operate on identical input spaces. For each dataset, rank-correlate the two versions' log p-values across all genes with Spearman correlation, and record wall-clock runtimes for both variants. Aggregate across datasets (mean Spearman correlation, per-dataset correlations, mean percent runtime saving) and inspect discrepant genes via res.top_features rankings (e.g., Itm2b and Lyz2 in the Tabula Muris lung dataset).

## Related tools

- **singleCellHaystack (R, binary branch v0.3.4)** (Reference variant: run on binarized detection matrix with basis='pca' to produce per-gene p-values to correlate against) — https://github.com/alexisvdb/singleCellHaystack/tree/binary
- **singleCellHaystack (continuous, R >= 1.0.0 / Python 0.0.5)** (Test variant: run on the same first-20-PC coordinates to produce continuous-model per-gene p-values) — https://github.com/alexisvdb/singleCellHaystack
- **Python / scanpy** (Load h5ad atlas datasets, compute PCA coordinates, compute Spearman correlations of log p-values, and time runs)

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Average Spearman correlation of log p-values across the 119 datasets should be ~0.92.
- The Tabula Muris lung dataset should reproduce Spearman 0.95, with Itm2b and Lyz2 flagged as discrepant genes in top_features rankings.
- Correlations must be computed across ALL genes per dataset (not a filtered subset) and only on identical input coordinates (first 20 PCs) for both variants.
- Continuous version should show an average ~37.8% wall-clock runtime reduction versus the binary version (article reports up to tenfold improvement).
- Both variants must consume the same per-dataset h5ad input; differences in outputs should be attributable to the binary vs continuous activity model only.

## Limitations

- Multi-condition (e.g., wild-type vs knockout) comparisons are not supported in the current singleCellHaystack versions.
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses.
- The Python package is in beta: weights.advanced.Q, the seeding grid method, and hierarchical cluster_genes clustering are not yet available.
- Runtime advantage over the binary version is average-based; per-dataset gains vary (up to tenfold on large datasets like the Mouse Organogenesis Cell Atlas with >100k cells).

## Evidence

- [methods] res = hs.haystack(adata, basis="pca"): "res = hs.haystack(adata, basis="pca")"
- [readme] the original implementation of singleCellHaystack for R (version 0.3.4) is on the binary branch: "the original implementation of `singleCellHaystack` for R ([version 0.3.4](https://github.com/alexisvdb/singleCellHaystack/tree/binary))"
- [abstract] Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells: "Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells"
- [discussion] the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression: "the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression"
- [readme] install the old binary version from the binary branch on GitHub: [evidence span withheld — anchor status: external_doc]
