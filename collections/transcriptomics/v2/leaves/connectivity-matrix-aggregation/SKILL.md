---
name: connectivity-matrix-aggregation
description: Use when a preprocessed gene-by-cell expression matrix must be split
  into gene subspaces before regression-based imputation, and the cluster count k
  is unknown and must be chosen from the data structure itself.
when_to_use_negative:
- The goal is cell clustering (e.g., k-means on PCA of cells to compute ARI against
  known labels) rather than grouping genes into subspaces.
- Zero-valued dropout entries have not yet been separated into training vs. imputable
  data by the hypothesis-testing module, so the training gene set is not defined.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Provides the data perturbation technique used in its second module to group
    genes with similar patterns into smaller groups via connectivity-matrix aggregation
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Provides the k-means partitioning and distribution functions (e.g., lm/stats
    routines) used in the scISR implementation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:434181b870551d80c1beb6ade7b8922dc6069845fb1a9bf3f71a5c1f2b79e41f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - scISR
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# connectivity-matrix-aggregation

## Summary

Determines the number of gene subspaces and partitions training genes into groups of similar expression patterns by comparing connectivity matrices built from original vs. Gaussian-noise-perturbed k-means partitions of scRNA-seq data, selecting the partition with maximal CDF-difference AUC (as in scISR's second module).

## When to use

Use when a preprocessed gene-by-cell expression matrix must be split into gene subspaces before regression-based imputation, and the cluster count k is unknown and must be chosen from the data structure itself.

## When NOT to use

- The goal is cell clustering (e.g., k-means on PCA of cells to compute ARI against known labels) rather than grouping genes into subspaces.
- Zero-valued dropout entries have not yet been separated into training vs. imputable data by the hypothesis-testing module, so the training gene set is not defined.

## Inputs

- Preprocessed training gene set expression matrix in an R data frame (genes expressed in at least two cells and with at least one percent non-zero entries)
- Range of candidate cluster counts k for k-means partitions
- Gaussian noise perturbation specification for the training gene set

## Outputs

- Connectivity matrices from original-data partitions and from each perturbed dataset's partitions
- CDF-difference curve between original and perturbed connectivity matrices with its AUC per candidate k
- Gene-subspace assignment table (one row per training gene with a non-empty group label), e.g., gene_subspace_assignments.csv

## How to apply

Load the preprocessed training gene set expression matrix (genes expressed in at least two cells and with at least one percent non-zero entries) into an R data frame. Run the data perturbation technique: add Gaussian noise to the training gene set, partition genes with k-means over a range of cluster counts k, repeating the perturbation-clustering cycle. Build a connectivity matrix from the partitions of the original data and, in parallel, connectivity matrices from partitions of each perturbed dataset, recording for each pair of genes whether they land in the same cluster. Compute the cumulative difference distribution (CDF difference) between the original and perturbed connectivity matrices and select the number of clusters whose area under this curve (AUC) is maximal. Under the selected partition, assign each training gene to its subspace group and write the gene-subspace assignment table (e.g., gene_subspace_assignments.csv).

## Related tools

- **scISR** (Provides the data perturbation technique used in its second module to group genes with similar patterns into smaller groups via connectivity-matrix aggregation) — https://github.com/duct317/scISR
- **R stats** (Provides the k-means partitioning and distribution functions (e.g., lm/stats routines) used in the scISR implementation)

## Examples

```
library(scISR); data('Goolam'); raw <- Goolam$data; imputed <- scISR(data = raw)
```

## Evaluation signals

- The selected partition is the one maximizing the CDF-difference AUC between original and perturbed connectivity matrices.
- gene_subspace_assignments.csv contains exactly one row per training gene, each with a non-empty group label.
- The input matrix was pre-filtered to genes expressed in at least two cells with at least one percent non-zero entries before perturbation clustering.
- Gene grouping is driven by pair-wise similarity of expression patterns recorded in the connectivity matrices (same-cluster co-assignment).
- Running scISR end-to-end afterward should yield imputed data whose k-means clusters on 50-component PCA have a higher adjustedRandIndex against labels than the raw-data clustering.

## Limitations

- The scISR article does not state a version for the R stats package or the scISR software, limiting reproducibility.
- The perturbation-clustering range of k and its sensitivity are not exhaustively specified in the main text; some evidence is deferred to supplementary sections not included here.
- Repository has no changelog ('_No changelog found._'), limiting reproducibility of the repository content.

## Evidence

- [intro] The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups.: "The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups."
- [results] Filtering criteria for input genes: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [readme] README install and package availability: "Using CRAN: `install.packages("scISR")`"
- [readme] README example invocation on Goolam data: "imputed <- scISR(data = raw)"
