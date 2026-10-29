---
name: single-cell-gene-perturbation-clustering
description: Use when a scRNA-seq matrix has been split into training and imputable
  data (via hyper-geometric dropout testing) and genes must be partitioned into subspaces
  for subspace regression imputation.
when_to_use_negative:
- The input is raw unfiltered scRNA-seq counts that have not yet been filtered (genes
  expressed in less than two cells removed) or split into training/imputable data
  by the hyper-geometric testing module
- The goal is to impute zero values directly — this skill only defines gene subspaces;
  imputation is done by the third scISR module via subspace regression
- The aim is clustering cells rather than genes — this perturbation-clustering procedure
  groups genes with similar expression patterns, not cells
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Implements the data perturbation module that automatically groups genes with
    similar patterns into smaller groups prior to subspace regression imputation
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Provides the k-means clustering used within each perturbation-clustering cycle
    over a range of cluster counts k
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:b3fd4973d0b6f9d385904eb6ce158884756e54c49d18afe5e6ec679e3fbc5221
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - scISR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-gene-perturbation-clustering

## Summary

Module 2 of the scISR imputation pipeline: uses a data perturbation technique to automatically group training genes with similar expression patterns into smaller gene subspaces before subspace regression imputation.

## When to use

Use when a scRNA-seq matrix has been split into training and imputable data (via hyper-geometric dropout testing) and genes must be partitioned into subspaces for subspace regression imputation.

## When NOT to use

- The input is raw unfiltered scRNA-seq counts that have not yet been filtered (genes expressed in less than two cells removed) or split into training/imputable data by the hyper-geometric testing module
- The goal is to impute zero values directly — this skill only defines gene subspaces; imputation is done by the third scISR module via subspace regression
- The aim is clustering cells rather than genes — this perturbation-clustering procedure groups genes with similar expression patterns, not cells

## Inputs

- Preprocessed training gene set expression matrix (genes expressed in at least two cells and with at least one percent non-zero entries), as an R data frame
- Training/imputable data split produced by scISR's hyper-geometric dropout testing module

## Outputs

- Gene-subspace assignment table (gene_subspace_assignments.csv) with one row per training gene and a non-empty group label
- Selected number of clusters k maximizing the CDF-difference AUC

## How to apply

Load the preprocessed training gene set expression matrix (genes expressed in at least two cells and having at least one percent non-zero entries) into an R data frame. Run the data perturbation technique: add Gaussian noise to the training genes and partition them with k-means, repeating the perturbation-clustering cycle over a range of cluster counts k. Build a connectivity matrix from the partitions of the original data and, in parallel, connectivity matrices from the partitions of each perturbed dataset, recording for each gene pair whether they were assigned to the same cluster. Compute the cumulative difference distribution (CDF difference) between original and perturbed connectivity matrices and select the k whose area under this curve (AUC) is maximal. Finally, assign each training gene to its subspace under the selected partition and write the gene-subspace assignment table (e.g. gene_subspace_assignments.csv).

## Related tools

- **scISR** (Implements the data perturbation module that automatically groups genes with similar patterns into smaller groups prior to subspace regression imputation) — https://github.com/duct317/scISR
- **R stats** (Provides the k-means clustering used within each perturbation-clustering cycle over a range of cluster counts k)

## Examples

```
library(scISR); data('Goolam'); raw <- Goolam$data; imputed <- scISR(data = raw)
```

## Evaluation signals

- The reported partition must be the one maximizing the CDF-difference AUC between original and perturbed connectivity matrices
- The gene-subspace assignment table contains one row per training gene, each with a non-empty group label
- Only genes passing the filters (expressed in at least two cells; at least one percent non-zero-valued entries) appear in the assignment table
- Every training gene receives exactly one subspace group under the selected partition (a valid partition, not a partial assignment)

## Limitations

- The perturbation-clustering cycles over a range of cluster counts k and relies on the CDF-difference AUC criterion; no fixed k is guaranteed a priori
- Only applicable to the training data identified by scISR's hypothesis-testing module; imputable-data genes are not grouped by this skill
- No version of the R stats package or scISR software is stated in the article, limiting reproducibility

## Evidence

- [intro] Second module groups genes via perturbation: "The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups."
- [intro] Third module uses learned gene patterns: "The third module imputes missing values affected by dropout events (identified in the first module) by learning the gene patterns in each gene group"
- [readme] Training/imputable split precedes regression: "Based on the result obtained from hyper-geometric testing, the original dataset is segregated into two including training data and imputable data."
- [results] Gene filtering thresholds: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
