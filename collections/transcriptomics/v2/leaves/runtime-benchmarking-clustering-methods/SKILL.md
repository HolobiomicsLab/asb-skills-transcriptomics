---
name: runtime-benchmarking-clustering-methods
description: Use when you need to measure or verify the runtime and cluster-count
  scalability of an scRNA-seq clustering method on a large (100k+) cell dataset, with
  sparse big-data mode enabled.
when_to_use_negative:
- The dataset is small enough that standard in-memory scCAN applies and a big-data
  sparse-mode benchmark is unnecessary.
- The goal is clustering accuracy (ARI against known labels) rather than scalability/runtime
  measurement — use the vignette's labeled example datasets instead.
- The clustering method under test has no sparse big-data mode, so a million-cell
  run is not feasible.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scCAN
  role: The clustering method being benchmarked; run with sparse=TRUE big-data mode,
    r.seed=1, k=10:50 eigengap search
  repo: https://cran.r-project.org/package=scCAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:19d4945ad7b8c5f66622ffcbd8423bfced5e4bf07c1c6780a17d8b16c0ed214c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# runtime-benchmarking-clustering-methods

## Summary

Benchmark a single-cell clustering method's scalability by running it on a million-cell dataset (e.g., the 1,300,774-cell Brain 1.3M 10X Genomics dataset) and recording the discovered cluster count and elapsed wall-clock runtime against reported reference values.

## When to use

Use when you need to measure or verify the runtime and cluster-count scalability of an scRNA-seq clustering method on a large (100k+) cell dataset, with sparse big-data mode enabled.

## When NOT to use

- The dataset is small enough that standard in-memory scCAN applies and a big-data sparse-mode benchmark is unnecessary.
- The goal is clustering accuracy (ARI against known labels) rather than scalability/runtime measurement — use the vignette's labeled example datasets instead.
- The clustering method under test has no sparse big-data mode, so a million-cell run is not feasible.

## Inputs

- TENxBrainData / Brain 1.3M 10X Genomics dataset (GSE93421), 1,300,774 cells x 27,998 genes, as a sparse count matrix
- scCAN R package installed via devtools::install_github('bangtran365/scCAN') or CRAN

## Outputs

- sccan_1M_brain_results.csv containing discovered cluster count and elapsed runtime in minutes
- Cluster label vector (result$cluster) for all 1.3M cells

## How to apply

Load the TENxBrainData dataset (1,300,774 cells x 27,998 genes) from Bioconductor or the 10X Genomics 1M_neurons download and convert counts to a sparse matrix per the scCAN vignette. Preprocess with log2(data + 1) transformation when max(data) > 100, then run scCAN in big-data mode with sparse=TRUE and r.seed=1, letting the eigengap search span k=10:50; Module 3 clusters a 5000-cell randomly selected training set and assigns the remaining cells via k-NN classification. Record the number of discovered clusters and the elapsed running time of the run. Write both values to sccan_1M_brain_results.csv and compare against the reference of 19 clusters and 51.37449 minutes, flagging any discrepancy beyond reasonable tolerance. The rationale is that Module 3's train/classify split is what makes million-cell runs tractable while still estimating cluster number via the eigengap method.

## Related tools

- **scCAN** (The clustering method being benchmarked; run with sparse=TRUE big-data mode, r.seed=1, k=10:50 eigengap search) — https://cran.r-project.org/package=scCAN

## Examples

```
library(scCAN); result <- sccan_run_sparse(data, sparse = TRUE, r.seed = 1, k = 10:50); cluster <- result$cluster
```

## Evaluation signals

- Discovered cluster count equals the reported 19 clusters for the 1.3M-cell Brain dataset.
- Recorded runtime is close to the reported 51.37449 minutes; discrepancies beyond reasonable tolerance are flagged in sccan_1M_brain_results.csv.
- Run configuration matches the reference: sparse=TRUE, r.seed=1, eigengap search over k=10:50, 5000-cell training set in Module 3.
- Input matrix dimensions match 1,300,774 cells x 27,998 genes before preprocessing.
- Results CSV contains both cluster count and runtime fields populated.

## Limitations

- Runtime is machine-dependent, so the 51.37449-minute reference is only reproducible on comparable hardware; treat runtime as approximate.
- Module 3 big-data mode uses a 5000-cell training set with k-NN classification of remaining cells, so cluster assignments for non-training cells are approximations rather than direct spectral clustering of the full fused network.
- No software version numbers are stated for scCAN or the compared tools (CIDR, SEURAT3, Monocle3, SHARP, SCANPY) in the article sections, complicating exact reproducibility.

## Evidence

- [results] scCAN clustered the 1.3 million-cell brain dataset into 19 clusters, completing the analysis in about 51 minutes (51.37449 minutes): [evidence span withheld — anchor status: not_found]
- [methods] we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells: "we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells"
- [results] The Brain 1.3M dataset was downloaded from the 10X Genomics website (https://support.10xgenomics.com/single-cell-gene-expression/datasets/1.3.0/1M_neurons).: "The Brain 1.3M dataset was downloaded from the 10X Genomics website (https://support.10xgenomics.com/single-cell-gene-expression/datasets/1.3.0/1M_neurons)."
- [readme] if(max(data)>100) data <- log2(data + 1): "Add log transformation: `if(max(data)>100) data <- log2(data + 1)`"
