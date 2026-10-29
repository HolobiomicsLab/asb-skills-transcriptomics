---
name: kmeans-partition-ensemble-construction
description: Use when a scRNA-seq training gene set must be partitioned into gene
  subspaces for subspace regression imputation, or when a robust gene grouping must
  be selected without a known number of clusters.
when_to_use_negative:
- The goal is clustering cells (e.g. computing ARI against known cell-type labels),
  not grouping genes into subspaces
- The input matrix is not the hyper-geometric-filtered training data (e.g. it still
  contains genes expressed in fewer than two cells or with under one percent non-zero
  entries)
- A known, fixed number of gene clusters must be used rather than a data-driven choice
  of k
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Hosts the second module that performs the perturbation-based gene grouping;
    running scISR triggers this ensemble construction internally
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Provides the k-means implementation used to partition genes in each perturbation
    round
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:9f175da7271538528af04d49063b6345c1318ec9b6dac4a9378131dc0896daee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - scISR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# kmeans-partition-ensemble-construction

## Summary

Constructs a k-means partition ensemble via data perturbation (adding Gaussian noise across a range of cluster counts k) to group genes with similar expression patterns into subspaces, as used in scISR's second module. The optimal k is chosen by maximizing the AUC of the cumulative difference (CDF-difference) between connectivity matrices of original and perturbed partitions.

## When to use

Use when a scRNA-seq training gene set must be partitioned into gene subspaces for subspace regression imputation, or when a robust gene grouping must be selected without a known number of clusters.

## When NOT to use

- The goal is clustering cells (e.g. computing ARI against known cell-type labels), not grouping genes into subspaces
- The input matrix is not the hyper-geometric-filtered training data (e.g. it still contains genes expressed in fewer than two cells or with under one percent non-zero entries)
- A known, fixed number of gene clusters must be used rather than a data-driven choice of k

## Inputs

- Preprocessed training gene-set expression matrix as an R data frame (genes expressed in at least two cells and with at least one percent non-zero-valued entries)
- Range of candidate cluster counts k for the perturbation-clustering cycle

## Outputs

- Selected number of clusters k maximizing the CDF-difference AUC
- Gene-subspace assignment table (one row per training gene with a non-empty group label), e.g. gene_subspace_assignments.csv
- Connectivity matrices from the original and perturbed k-means partitions

## How to apply

Load the preprocessed training gene expression matrix — genes expressed in at least two cells and having at least one percent non-zero entries — into an R data frame. Repeatedly perturb the data by adding Gaussian noise and run k-means over a range of cluster counts k, recording for each gene pair whether it lands in the same cluster to build a connectivity matrix for the original and each perturbed dataset. Compute the cumulative difference distribution (CDF difference) between original and perturbed connectivity matrices and select the k whose area under this curve (AUC) is maximal; this implements the data perturbation technique cited by scISR's second module for automatically grouping genes with similar patterns. Finally, assign each training gene to its subspace under the selected partition.

## Related tools

- **scISR** (Hosts the second module that performs the perturbation-based gene grouping; running scISR triggers this ensemble construction internally) — https://github.com/duct317/scISR
- **R stats** (Provides the k-means implementation used to partition genes in each perturbation round)

## Examples

```
library(scISR); imputed <- scISR(data = raw)
```

## Evaluation signals

- The selected partition is the one maximizing the AUC of the CDF-difference curve between original and perturbed connectivity matrices
- The gene-subspace assignment table contains exactly one row per training gene, each with a non-empty group label
- The input matrix satisfies the filtering rule: genes expressed in at least two cells and at least one percent non-zero entries
- Every gene pair's co-clustering status is recorded consistently in both original and perturbed connectivity matrices
- The resulting group structure feeds the subspace regression model whose output improves k-means ARI over raw data in the README-style assessment

## Limitations

- Requires repeating the perturbation-clustering cycle over many values of k, which increases computational cost
- The grouping depends on the quality of the training set defined by upstream hyper-geometric dropout testing; misclassified training/imputable split degrades the subspaces
- Genes expressed in fewer than two cells or with less than one percent non-zero entries must be removed beforehand, so very sparse genes cannot be assigned a subspace

## Evidence

- [intro] The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups.: "The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups."
- [results] genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries.: "genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [readme] Perform the imputation: imputed <- scISR(data = raw): [evidence span withheld — anchor status: external_doc]
