---
name: cross-version-runtime-benchmarking
description: 'Use when an updated tool version must be validated against its predecessor:
  run both on the same datasets, compare ranked outputs (Spearman of log p-values)
  and measure runtime savings.'
when_to_use_negative:
- The analysis requires comparisons between multiple conditions (e.g., wild-type vs
  knockout) — singleCellHaystack cannot easily conduct such comparisons
- The goal is spatial transcriptomics DEG detection where speed is paramount — SPARK-X
  is the fastest method available and singleCellHaystack has relatively longer runtimes
  despite better sensitivity
- Only one tool version exists and no predecessor output is available for pairwise
  comparison
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: 'Tool under benchmark: binary v0.3.2 (R, ''binary'' branch) and continuous
    v1.0.0+ versions produce the per-gene p-values being compared'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python)
  role: Python implementation (PyPI v0.0.5+) used to run the continuous version on
    h5ad datasets with scanpy
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: Python
  role: Language for loading h5ad datasets with scanpy, computing first-20-PC coordinates,
    and computing Spearman correlations and runtimes
- name: R
  role: Language for installing and running the original binary v0.3.2 implementation
    on the binarized detection matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:e151cab6dd32e10f3c5261cb181586fec9a2a014f8ce2b0dd5027e72303980ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R
  - singleCellHaystack
  - singleCellHaystack (Python)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-version-runtime-benchmarking

## Summary

Benchmarks two versions of singleCellHaystack (binary v0.3.2 vs continuous v1.0.0+) across 119 Tabula Muris and Mouse Cell Atlas scRNA-seq datasets, quantifying agreement of per-gene log p-values (Spearman correlation) and wall-clock runtime reduction.

## When to use

Use when an updated tool version must be validated against its predecessor: run both on the same datasets, compare ranked outputs (Spearman of log p-values) and measure runtime savings.

## When NOT to use

- The analysis requires comparisons between multiple conditions (e.g., wild-type vs knockout) — singleCellHaystack cannot easily conduct such comparisons
- The goal is spatial transcriptomics DEG detection where speed is paramount — SPARK-X is the fastest method available and singleCellHaystack has relatively longer runtimes despite better sensitivity
- Only one tool version exists and no predecessor output is available for pairwise comparison

## Inputs

- 119 scRNA-seq atlas datasets in h5ad format (32 Tabula Muris + 87 Mouse Cell Atlas), read with scanpy sc.read_h5ad
- Binarized gene detection matrix (for the binary v0.3.2 implementation)
- First 20 principal components per dataset (input coordinates for both versions)
- singleCellHaystack binary v0.3.2 (R, 'binary' branch) and continuous v1.0.0+ (R/Python) installations

## Outputs

- Per-gene p-values from each singleCellHaystack version for all 119 datasets
- Per-dataset Spearman rank correlations between log p-values of the two versions (average ~0.92)
- Wall-clock runtimes per version per dataset (mean ~37.8% runtime saving for the continuous version)
- Lists of discrepant genes (e.g., Itm2b, Lyz2 in the Tabula Muris lung dataset) from res.top_features rankings

## How to apply

Install both implementations: the original binary version (R, v0.3.2/0.3.4 from the alexisvdb/singleCellHaystack 'binary' branch via remotes::install_github("alexisvdb/singleCellHaystack@binary")) and the continuous version (R >=1.0.0 from CRAN or the Python package from PyPI). Load each of the 119 datasets (32 Tabula Muris + 87 Mouse Cell Atlas) from h5ad files with scanpy and extract the first 20 principal components as input coordinates, since the continuous method accepts higher-dimensional latent spaces. Run hs.haystack on the binarized detection matrix (binary version) and on the same first-20-PC coordinates with continuous activity levels (continuous version), using basis='pca'. For each dataset compute the Spearman rank correlation between the two versions' log p-values across all genes and record wall-clock runtimes. Aggregate: average Spearman correlation (~0.92 expected), dataset-level agreement (Tabula Muris lung: 0.95, with Itm2b and Lyz2 as discrepant genes via res.top_features rankings), and mean percent runtime saving (~37.8%).

## Related tools

- **singleCellHaystack** (Tool under benchmark: binary v0.3.2 (R, 'binary' branch) and continuous v1.0.0+ versions produce the per-gene p-values being compared) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python)** (Python implementation (PyPI v0.0.5+) used to run the continuous version on h5ad datasets with scanpy) — https://github.com/ddiez/singleCellHaystack-py
- **Python** (Language for loading h5ad datasets with scanpy, computing first-20-PC coordinates, and computing Spearman correlations and runtimes)
- **R** (Language for installing and running the original binary v0.3.2 implementation on the binarized detection matrix)

## Examples

```
import scanpy as sc; import singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Average Spearman correlation of log p-values across the 119 datasets is ~0.92 between binary and continuous versions
- Tabula Muris lung dataset reproduces Spearman 0.95, with Itm2b and Lyz2 identified as discrepant genes via res.top_features
- Continuous version shows an average 37.8% runtime reduction versus the binary version (article reports up to ten-fold reduction in computational time)
- Both runs use identical inputs: same h5ad dataset and the same first-20-PC coordinates (basis='pca')
- Both versions return per-gene p-values for every gene in each dataset, enabling full-ranking Spearman comparison

## Limitations

- The original binary implementation required binary detection data, ignoring subtle expression differences and overestimating significance of modest differences
- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- For spatial transcriptomics, singleCellHaystack has relatively longer runtimes than SPARK-X, despite better sensitivity

## Evidence

- [abstract] Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells: "Performance has been drastically improved, with up to ten times reduction in computational time and scalability to millions of cells"
- [methods] res = hs.haystack(adata, basis="pca"): "res = hs.haystack(adata, basis="pca")"
- [discussion] the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression: "the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression"
- [readme] remotes::install_github("alexisvdb/singleCellHaystack@binary"): [evidence span withheld — anchor status: external_doc]
