---
name: adjusted-rand-index-benchmarking
description: Use when comparing clustering accuracy of multiple single-cell methods
  on scRNA-seq datasets with known cell-type labels, and per-dataset ARI averages
  are needed to rank methods.
when_to_use_negative:
- Datasets lack known cell-type labels, so no ground truth exists for computing an
  adjusted Rand index
- The goal is trajectory/pseudotime or classification evaluation rather than unsupervised
  clustering accuracy — scDHA.pt and scDHA.class would be the appropriate modules
  instead
- Input is already a low-dimensional embedding; this skill benchmarks full clustering
  pipelines from expression matrices
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: Method under test; produces unsupervised cell clusters via non-negative kernel
    autoencoder plus stacked Bayesian autoencoder
  repo: github.com/duct317/scDHA
- name: CIDR
  role: Comparator clustering method using implicit imputation and dimensionality
    reduction, run with default settings
  repo: github.com/VCCRI/CIDR
- name: SC3
  role: Comparator clustering method run with default settings
- name: SEURAT
  role: Comparator clustering method run with default settings
- name: SINCERA
  role: Comparator clustering method run with default settings
- name: SCANPY
  role: Comparator clustering method run with default settings
- name: mclust
  role: Provides adjustedRandIndex used to score clusters against known cell labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:d7db8396e99918afde5f49420a5be175037db70927bc038bc82229148c76456a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
  - SC3
  - SCANPY
  - SEURAT
  - SINCERA
  - mclust
  - scDHA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# adjusted-rand-index-benchmarking

## Summary

Benchmark single-cell clustering methods (scDHA vs SC3, SEURAT, SINCERA, CIDR, SCANPY, k-means) by computing adjusted Rand index against known cell-type labels across 34 scRNA-seq datasets, aggregating per-dataset scores into a per-method comparison.

## When to use

Use when comparing clustering accuracy of multiple single-cell methods on scRNA-seq datasets with known cell-type labels, and per-dataset ARI averages are needed to rank methods.

## When NOT to use

- Datasets lack known cell-type labels, so no ground truth exists for computing an adjusted Rand index
- The goal is trajectory/pseudotime or classification evaluation rather than unsupervised clustering accuracy — scDHA.pt and scDHA.class would be the appropriate modules instead
- Input is already a low-dimensional embedding; this skill benchmarks full clustering pipelines from expression matrices

## Inputs

- 34 scRNA-seq datasets with known cell-type labels, retrieved via GEO/SCP accession numbers (e.g. Goolam dataset as matrix with rows as cells, columns as genes)
- Log2-transformed expression matrix (log2(counts + 1)) per dataset
- Ground-truth cell-type label vector per dataset

## Outputs

- Per-dataset adjusted Rand index for each of the 7 clustering methods (scDHA, SC3, SEURAT, SINCERA, CIDR, SCANPY, k-means)
- Per-method average ARI across all 34 datasets (expected: scDHA 0.81, CIDR 0.5)
- Per-method ARI comparison plot and summary table

## How to apply

Retrieve the 34 scRNA-seq datasets (Table 1) using their GEO/SCP accession numbers, each with known cell-type labels. Run scDHA's clustering pipeline — a non-negative kernel autoencoder followed by a stacked Bayesian autoencoder — on log2-transformed counts to obtain unsupervised clusters (e.g. result$cluster from the scDHA R function with a fixed seed for reproducibility). Run the competing methods (SC3, Seurat, SINCERA, CIDR, Scanpy, k-means) on the same datasets with default settings. Compute adjustedRandIndex(cluster, label) for each method-dataset pair using the mclust package, then aggregate per-dataset ARI values into a summary table and per-method comparison plot. Validate the reproduction by confirming the mean ARI matches the reported benchmark: 0.81 for scDHA and 0.5 for the second-best method CIDR.

## Related tools

- **scDHA** (Method under test; produces unsupervised cell clusters via non-negative kernel autoencoder plus stacked Bayesian autoencoder) — github.com/duct317/scDHA
- **CIDR** (Comparator clustering method using implicit imputation and dimensionality reduction, run with default settings) — github.com/VCCRI/CIDR
- **SC3** (Comparator clustering method run with default settings)
- **SEURAT** (Comparator clustering method run with default settings)
- **SINCERA** (Comparator clustering method run with default settings)
- **SCANPY** (Comparator clustering method run with default settings)
- **mclust** (Provides adjustedRandIndex used to score clusters against known cell labels)

## Examples

```
library(scDHA); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1); mclust::adjustedRandIndex(result$cluster, as.character(Goolam$label))
```

## Evaluation signals

- Mean ARI across all 34 datasets reproduces the reported values: 0.81 for scDHA and 0.5 for CIDR
- scDHA shows very low variability in ARI across datasets, dominating all six comparators
- Each per-dataset ARI computed with mclust::adjustedRandIndex(cluster, label) against the known cell types
- Clustering results are reproducible: re-running scDHA with the same seed (e.g. seed = 1) yields identical clusters
- Per-method summary table covers all 34 datasets with no missing ARI entries

## Limitations

- Benchmark scores depend on competitor methods being run with default settings, which may not be optimal for each dataset
- ARI assessment requires known cell-type labels, restricting applicability to annotated reference datasets
- CIDR masks the base stats hclust function; loading packages with their own hclust (e.g. flashClust) after CIDR can cause name-clashing problems during clustering

## Evidence

- [results] scDHA achieves an average ARI of 0.81 across all 34 data sets: [evidence span withheld — anchor status: not_found]
- [abstract] scDHA two-module clustering pipeline: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [readme] README cluster scoring with ARI: "Calculating adjusted Rand Index using mclust package: `mclust::adjustedRandIndex(cluster,label)`"
- [intro] comparator methods listed in intro: "Widely-used methods in this category include SC39, SEURAT10, SINCERA11, CIDR12, and SCANPY13"
- [readme] CIDR ARI usage: [evidence span withheld — anchor status: external_doc]
