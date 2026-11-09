---
name: cross-method-clustering-performance-comparison
description: Use when the goal is to reproduce or extend a supervised-by-labels clustering
  benchmark that compares multiple scRNA-seq clustering tools on datasets with known
  cell types using ARI.
when_to_use_negative:
- Datasets lack known cell-type labels, so ARI against a ground truth cannot be computed.
- The task is dimension reduction, visualization, classification, or pseudo-time inference
  rather than unsupervised cell segregation/clustering comparison.
- Only a single clustering method needs to be run on one dataset without any comparative
  benchmarking.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: 'Primary clustering method under evaluation: non-negative kernel autoencoder
    plus stacked Bayesian autoencoder producing unsupervised cell clusters'
  repo: github.com/duct317/scDHA
- name: SC3
  role: Comparator clustering method run with default settings on the same 34 datasets
- name: SEURAT
  role: Comparator clustering method run with default settings on the same 34 datasets
- name: SINCERA
  role: Comparator clustering method run with default settings on the same 34 datasets
- name: CIDR
  role: Comparator clustering method (implicit imputation plus dimensionality reduction),
    second-best with mean ARI 0.5
  repo: github.com/VCCRI/CIDR
- name: SCANPY
  role: Comparator clustering method run with default settings on the same 34 datasets
- name: k-means
  role: Baseline clustering comparator
- name: mclust
  role: R package used to compute the adjusted Rand index against known cell labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:c95f7653b07334c028caad041d901bfd1f1d19fb6a91722311598e5e6cc2be65
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
  - SC3
  - SCANPY
  - SEURAT
  - SINCERA
  - k-means
  - mclust
  - scDHA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cross-method-clustering-performance-comparison

## Summary

Benchmark scRNA-seq clustering methods (scDHA, SC3, Seurat, SINCERA, CIDR, Scanpy, k-means) by computing the adjusted Rand index (ARI) of unsupervised clusters against known cell-type labels across 34 datasets with known cell types.

## When to use

Use when the goal is to reproduce or extend a supervised-by-labels clustering benchmark that compares multiple scRNA-seq clustering tools on datasets with known cell types using ARI.

## When NOT to use

- Datasets lack known cell-type labels, so ARI against a ground truth cannot be computed.
- The task is dimension reduction, visualization, classification, or pseudo-time inference rather than unsupervised cell segregation/clustering comparison.
- Only a single clustering method needs to be run on one dataset without any comparative benchmarking.

## Inputs

- 34 scRNA-seq gene expression matrices (rows as cells/samples, columns as genes) with known cell-type labels, identified by GEO/SCP accession numbers in Table 1
- Preprocessed count matrices, log2-transformed (log2(data + 1)) before clustering

## Outputs

- Per-dataset adjusted Rand index (ARI) for each of the 7 clustering methods
- Summary table of aggregated per-method ARI values (e.g. scDHA mean 0.81, CIDR mean 0.5)
- Per-method ARI comparison plot across the 34 datasets

## How to apply

Retrieve the 34 scRNA-seq datasets listed in Table 1 using their GEO/SCP accession numbers. Run the scDHA clustering pipeline — a non-negative kernel autoencoder (which removes genes or components with insignificant contributions based on encoder weight distribution) followed by a stacked Bayesian autoencoder that projects data onto a low-dimensional space — on each dataset to obtain unsupervised cell clusters, e.g. via scDHA(data, seed = 1) after log2(data + 1) transformation. Run the six competing methods (SC3, Seurat, SINCERA, CIDR, Scanpy, k-means) on the same datasets with default settings. Compute the adjusted Rand index of each method's clusters against the known cell labels for every dataset (e.g. with mclust::adjustedRandIndex). Aggregate per-dataset ARI values into a summary table and plot the per-method comparison; validate by confirming reproduced means match the reported benchmark — mean ARI of 0.81 for scDHA and 0.5 for the second-best method, CIDR.

## Related tools

- **scDHA** (Primary clustering method under evaluation: non-negative kernel autoencoder plus stacked Bayesian autoencoder producing unsupervised cell clusters) — github.com/duct317/scDHA
- **SC3** (Comparator clustering method run with default settings on the same 34 datasets)
- **SEURAT** (Comparator clustering method run with default settings on the same 34 datasets)
- **SINCERA** (Comparator clustering method run with default settings on the same 34 datasets)
- **CIDR** (Comparator clustering method (implicit imputation plus dimensionality reduction), second-best with mean ARI 0.5) — github.com/VCCRI/CIDR
- **SCANPY** (Comparator clustering method run with default settings on the same 34 datasets)
- **k-means** (Baseline clustering comparator)
- **mclust** (R package used to compute the adjusted Rand index against known cell labels)

## Examples

```
library(scDHA); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1); mclust::adjustedRandIndex(result$cluster, as.character(Goolam$label))
```

## Evaluation signals

- Reproduced mean ARI for scDHA across all 34 datasets equals 0.81 with very low variability.
- Reproduced mean ARI for CIDR, the second-best method, equals 0.5.
- Per-dataset ARI values are computed against the known cell labels using adjustedRandIndex (e.g. mclust::adjustedRandIndex(cluster, label)).
- Every method is run on the identical set of 34 datasets with competing methods at default settings, so the comparison is paired per dataset.
- The scDHA clustering result is reproducible when the seed is fixed (result <- scDHA(data, seed = 1)).

## Limitations

- Benchmark validity depends entirely on the quality of known cell-type labels in the 34 datasets; ARI is a label-dependent metric.
- Competing methods were run with default settings, which may not reflect their optimally tuned performance.
- Discussion section claims accuracy expectations rather than demonstrated limits; no explicit limitations section is acknowledged in the article.

## Evidence

- [results] scDHA achieves an average ARI of 0.81 across all 34 data sets with very low variability, while the second best method, CIDR, has an average ARI of only 0.5.: [evidence span withheld — anchor status: not_found]
- [abstract] The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data.: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [intro] Widely-used methods in this category include SC39, SEURAT10, SINCERA11, CIDR12, and SCANPY13: "Widely-used methods in this category include SC39, SEURAT10, SINCERA11, CIDR12, and SCANPY13"
- [readme] Calculating adjusted Rand Index using mclust package: mclust::adjustedRandIndex(cluster,label): "Calculating adjusted Rand Index using mclust package: `mclust::adjustedRandIndex(cluster,label)`"
- [readme] Generating clustering result: result <- scDHA(data, seed = 1): "Generating clustering result: `result <- scDHA(data, seed = 1)`"
