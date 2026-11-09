---
name: gene-subspace-assignment
description: Use when a preprocessed scRNA-seq training gene set (genes expressed
  in at least two cells with at least one percent non-zero entries) must be partitioned
  into gene subspaces so that dropout-imputation regression can be run within each
  group.
when_to_use_negative:
- The zero entries have not yet been screened by the hyper-geometric over-/under-representation
  testing (significance threshold 0.01) — grouping must be run on the training data
  only, not the whole matrix including imputable entries.
- Genes expressed in fewer than two cells or with less than one percent non-zero-valued
  entries have not been filtered out — including them distorts the perturbation clustering.
- The goal is direct imputation without gene grouping — this skill is an intermediate
  step of scISR, not a standalone imputer; use scImpute, SAVER, MAGIC, etc. for other
  imputation paradigms.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scISR
  role: R package implementing the data-perturbation technique that groups genes with
    similar expression patterns into subspaces used for subspace-regression imputation
  repo: https://github.com/duct317/scISR
- name: R stats
  role: stats R package supplying the k-means and lm functions used in the perturbation-clustering
    cycle and the per-subspace regression downstream
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:088f673e2e564358982ae66d93b9dbc8523d282a7185f8f48baae65fd3d5e8a2
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-subspace-assignment

## Summary

Groups training genes into co-expression subspaces via scISR's data-perturbation clustering: Gaussian-noise perturbation plus repeated k-means, connectivity matrices, and selection of the partition maximizing the CDF-difference AUC. These subspaces are the regression neighborhoods that the downstream imputation module draws predictors from.

## When to use

Use when a preprocessed scRNA-seq training gene set (genes expressed in at least two cells with at least one percent non-zero entries) must be partitioned into gene subspaces so that dropout-imputation regression can be run within each group.

## When NOT to use

- The zero entries have not yet been screened by the hyper-geometric over-/under-representation testing (significance threshold 0.01) — grouping must be run on the training data only, not the whole matrix including imputable entries.
- Genes expressed in fewer than two cells or with less than one percent non-zero-valued entries have not been filtered out — including them distorts the perturbation clustering.
- The goal is direct imputation without gene grouping — this skill is an intermediate step of scISR, not a standalone imputer; use scImpute, SAVER, MAGIC, etc. for other imputation paradigms.

## Inputs

- Preprocessed training-gene expression matrix from scRNA-seq data (genes expressed in at least two cells, at least one percent non-zero entries), e.g., the Goolam (E-MTAB-3321) CPM matrix
- Binary imputable-entry mask or training/imputable split produced by the upstream hyper-geometric hypothesis-testing module
- Range of candidate cluster counts k for the perturbation-clustering cycle

## Outputs

- Gene-subspace assignment table (one row per training gene with a non-empty group label), e.g., gene_subspace_assignments.csv
- Connectivity matrices from the original and perturbed partitions, with the CDF-difference AUC per candidate k
- Selected partition (k maximizing CDF-difference AUC) used by the subspace-regression imputation module

## How to apply

Load the preprocessed training-gene expression matrix (genes expressed in at least two cells and with at least one percent non-zero entries) into an R data frame; if the value range exceeds 100, rescale with a base-2 log transform. Run the perturbation technique from the scISR package: add Gaussian noise to the training genes and partition with k-means (from the R stats package), repeating the perturbation-clustering cycle over a range of cluster counts k. Build a connectivity matrix recording, for each gene pair, whether they share a cluster in the original partition, and parallel connectivity matrices from each perturbed dataset's partitions; compute the cumulative difference (CDF difference) distribution between original and perturbed matrices. Select the number of clusters k whose area under the CDF-difference curve (AUC) is maximal, then assign each training gene to its subspace group under that partition and write the table (e.g., gene_subspace_assignments.csv). Rationale: perturbation reveals the data's natural structure, and genes grouped by pairwise expression-pattern similarity form the local neighborhoods in which Pearson-correlated predictors (top 10 genes) are selected for the lm-based regression that imputes imputable dropout entries.

## Related tools

- **scISR** (R package implementing the data-perturbation technique that groups genes with similar expression patterns into subspaces used for subspace-regression imputation) — https://github.com/duct317/scISR
- **R stats** (stats R package supplying the k-means and lm functions used in the perturbation-clustering cycle and the per-subspace regression downstream)

## Examples

```
library(scISR); data('Goolam'); raw <- Goolam$data; imputed <- scISR(data = raw)
```

## Evaluation signals

- The reported partition is the one maximizing the area under the cumulative-difference (CDF difference) curve between original and perturbed connectivity matrices.
- The gene-subspace assignment table contains exactly one row per training gene, each with a non-empty group label.
- Downstream, each imputable gene is assigned to its nearest subspace by Euclidean distance to the subspace centroid, and imputed entries hold nonzero fitted values from the stats::lm regression.
- Final acceptance check is improved k-means ARI against known cell labels using the true number of cell types k — e.g., 0.6855 versus 0.5124 on Goolam.
- Output matrices keep the same gene-by-cell dimensions as the input matrix after rescaling (log2 transform applied when the value range exceeds 100).

## Limitations

- The version of the R stats package and of the scISR software is not stated in the article, limiting exact reproducibility.
- Robustness evidence for the pipeline is deferred to supplementary figures (S19, S21, S24) not included in the main text.
- The scISR repository carries no changelog ('_No changelog found._'), limiting reproducibility of repository content across versions.

## Evidence

- [intro] Data perturbation module to group genes with similar patterns: "The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups."
- [results] Log2 rescaling threshold: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] Gene filtering criterion: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [discussion] Pearson-correlated predictor selection: [evidence span withheld — anchor status: not_found]
- [readme] Training/imputable split from README: "Next, training data is used for constructing a generalize linear regression model that is used for imputation on the imputable data."
