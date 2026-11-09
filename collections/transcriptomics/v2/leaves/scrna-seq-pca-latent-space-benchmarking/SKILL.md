---
name: scrna-seq-pca-latent-space-benchmarking
description: Use when comparing two versions or parameterizations of a DEG/differentially-active-feature
  method across many scRNA-seq atlas datasets (e.g. 32 Tabula Muris + 87 Mouse Cell
  Atlas) to quantify rank agreement, runtime savings, and discrepant genes.
when_to_use_negative:
- When the goal is comparing multiple experimental conditions (e.g. wild-type vs knockout)
  — singleCellHaystack currently cannot easily conduct such comparisons.
- When results must be integrated into a cluster-oriented DEG analysis — the paper
  notes results are not straightforward to incorporate into cluster-based workflows.
- When benchmarking spatial transcriptomics methods where raw speed is the priority
  — SPARK-X is faster, though singleCellHaystack offers better sensitivity.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: singleCellHaystack (R, binary branch)
  role: Original v0.3.2/0.3.4 binary-detection implementation run on the binarized
    detection matrix with basis='pca' to obtain baseline per-gene p-values
  repo: https://github.com/alexisvdb/singleCellHaystack/tree/binary
- name: singleCellHaystack (R/Python, v1.0.0+)
  role: Updated continuous-activity implementation run on the same first-20-PC coordinates
    to obtain comparison per-gene p-values and top_features
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python)
  role: Python implementation (pip install singleCellHaystack) used with scanpy to
    read h5ad inputs and run hs.haystack on atlas-scale datasets
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: Python (scanpy)
  role: Loads h5ad datasets via sc.read_h5ad and supplies the PCA basis (basis='pca')
    as input coordinates
- name: R
  role: Runtime environment for the binary-branch and CRAN implementations of singleCellHaystack
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:90ed0651329e806474c6296e0d5439eb09499244cbcf25b212d5d457053ca908
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python (scanpy)
  - R
  - singleCellHaystack (Python)
  - singleCellHaystack (R, binary branch)
  - singleCellHaystack (R/Python, v1.0.0+)
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# scrna-seq-pca-latent-space-benchmarking

## Summary

Benchmark a clustering-independent DEG/DAF method (e.g. singleCellHaystack) across many scRNA-seq atlas datasets by running old vs. new implementations on the same first-20-PC latent space and comparing per-gene log p-values via Spearman correlation, runtimes, and top-feature discrepancies.

## When to use

Use when comparing two versions or parameterizations of a DEG/differentially-active-feature method across many scRNA-seq atlas datasets (e.g. 32 Tabula Muris + 87 Mouse Cell Atlas) to quantify rank agreement, runtime savings, and discrepant genes.

## When NOT to use

- When the goal is comparing multiple experimental conditions (e.g. wild-type vs knockout) — singleCellHaystack currently cannot easily conduct such comparisons.
- When results must be integrated into a cluster-oriented DEG analysis — the paper notes results are not straightforward to incorporate into cluster-based workflows.
- When benchmarking spatial transcriptomics methods where raw speed is the priority — SPARK-X is faster, though singleCellHaystack offers better sensitivity.

## Inputs

- scRNA-seq atlas datasets in h5ad format (e.g. 32 Tabula Muris + 87 Mouse Cell Atlas datasets)
- First 20 principal components per dataset as input coordinates for hs.haystack
- Binarized gene detection matrix (for the binary v0.3.2 implementation)

## Outputs

- Per-gene p-values (and log p-values) from both binary and continuous singleCellHaystack runs per dataset
- Per-dataset Spearman rank correlation between the two versions' log p-values across all genes
- Wall-clock runtimes and mean percent runtime saving across datasets
- Top-feature rankings (res.top_features) identifying discrepant genes such as Itm2b and Lyz2

## How to apply

Install both implementations (binary singleCellHaystack v0.3.2/0.3.4 from the 'binary' branch in R, and continuous v1.0.0+/Python 0.0.5 from PyPI/CRAN). Load each dataset from h5ad with scanpy and compute/extract the first 20 principal components as the shared input coordinates, then run hs.haystack with basis='pca' under both the binary detection-matrix model and the continuous-activity model. For each dataset, compute the Spearman rank correlation between the two versions' log p-values across all genes and record wall-clock runtimes. Aggregate metrics across datasets (mean Spearman correlation, per-dataset correlations, mean percent runtime saving) and inspect res.top_features rankings to identify discrepant genes. Treat agreement around Spearman ~0.92 average and a ~37.8% mean runtime reduction as the expected benchmark outcome for this comparison.

## Related tools

- **singleCellHaystack (R, binary branch)** (Original v0.3.2/0.3.4 binary-detection implementation run on the binarized detection matrix with basis='pca' to obtain baseline per-gene p-values) — https://github.com/alexisvdb/singleCellHaystack/tree/binary
- **singleCellHaystack (R/Python, v1.0.0+)** (Updated continuous-activity implementation run on the same first-20-PC coordinates to obtain comparison per-gene p-values and top_features) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python)** (Python implementation (pip install singleCellHaystack) used with scanpy to read h5ad inputs and run hs.haystack on atlas-scale datasets) — https://github.com/ddiez/singleCellHaystack-py
- **Python (scanpy)** (Loads h5ad datasets via sc.read_h5ad and supplies the PCA basis (basis='pca') as input coordinates)
- **R** (Runtime environment for the binary-branch and CRAN implementations of singleCellHaystack)

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Average Spearman correlation of log p-values across the 119 datasets is approximately 0.92
- Tabula Muris lung dataset reproduces Spearman correlation of 0.95 between the two versions' log p-values
- Discrepant genes Itm2b and Lyz2 appear in the res.top_features rankings for the Tabula Muris lung dataset
- Continuous version shows an average ~37.8% runtime reduction versus the binary version
- Same first-20-PC coordinates are used for both implementations so only the activity model differs

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted with singleCellHaystack
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses compared with cluster-based DEG prediction approaches
- The Python implementation is in beta and lacks weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes
- Relatively longer runtimes than SPARK-X for spatial transcriptomics, despite better sensitivity

## Evidence

- [methods] Spearman agreement across 119 datasets: "res = hs.haystack(adata, basis="pca")"
- [intro] Continuous vs binary design rationale: "First, our new method uses continuous activity levels, enabling the identification of changes in any kind of features (e.g., RNA or protein concentrations, chromatin accessibility of genomic loci,"
- [methods] Binary version origin: "the original implementation of `singleCellHaystack` for R ([version 0.3.4](https://github.com/alexisvdb/singleCellHaystack/tree/binary))"
- [methods] Reading h5ad input: "adata = sc.read_h5ad("data.h5ad")"
- [discussion] Binary threshold limitation motivating benchmark: "the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression"
- [methods] Top features output: "res.top_features(n=10)"
- [readme] Binary branch installation (README): [evidence span withheld — anchor status: external_doc]
