---
name: cdf-difference-optimality-assessment
description: Use when running scISR's data perturbation module on a filtered training
  gene-by-cell expression matrix and you must decide how many gene-subspace groups
  to retain before subspace regression imputation.
when_to_use_negative:
- The goal is clustering cells rather than grouping genes into subspaces — scISR's
  k-means cell clustering with the true number of cell types is a downstream evaluation
  step, not this skill.
- The goal is identifying dropout-affected zero entries — that is scISR's first module
  (hyper-geometric/over- and under-representation testing with significance threshold
  0.01), which runs before perturbation clustering.
- The data have not been filtered and rescaled (log2 transformation for value ranges
  larger than 100), so perturbation clustering would run on genes expressed in fewer
  than two cells.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scISR
  role: Package whose second module performs the data perturbation technique to automatically
    group genes with similar patterns into smaller groups; selected partition defines
    gene subspaces for subspace regression imputation
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Provides the k-means clustering used to partition genes during each perturbation-clustering
    cycle and the lm function for the subsequent regression
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:979af48c359f117a3641a96249dd07a0657c5a0b5271151b90efaae4e5a3773b
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cdf-difference-optimality-assessment

## Summary

Selects the optimal number of gene-subspace clusters k in scISR's second module by comparing connectivity matrices of the original versus Gaussian-noise-perturbed training data, and choosing the partition whose cumulative difference distribution (CDF-difference) has maximal area under the curve (AUC).

## When to use

Use when running scISR's data perturbation module on a filtered training gene-by-cell expression matrix and you must decide how many gene-subspace groups to retain before subspace regression imputation.

## When NOT to use

- The goal is clustering cells rather than grouping genes into subspaces — scISR's k-means cell clustering with the true number of cell types is a downstream evaluation step, not this skill.
- The goal is identifying dropout-affected zero entries — that is scISR's first module (hyper-geometric/over- and under-representation testing with significance threshold 0.01), which runs before perturbation clustering.
- The data have not been filtered and rescaled (log2 transformation for value ranges larger than 100), so perturbation clustering would run on genes expressed in fewer than two cells.

## Inputs

- Preprocessed scRNA-seq training gene expression matrix (R data frame; genes expressed in at least two cells with at least one percent non-zero entries)
- Range of candidate cluster counts k
- Gaussian noise perturbation scheme for the training data

## Outputs

- Selected number of clusters k maximizing the CDF-difference AUC
- Gene-subspace assignment table written to gene_subspace_assignments.csv (one row per training gene with a non-empty group label)
- Connectivity matrices for original and perturbed datasets

## How to apply

Load the preprocessed training gene expression matrix — genes retained after removing those expressed in fewer than two cells and those with less than one percent non-zero-valued entries — into an R data frame. Repeatedly add Gaussian noise to the data and partition genes with k-means over a range of candidate cluster counts k, building a connectivity matrix from the original-data partitions and, in parallel, connectivity matrices from each perturbed dataset, recording for each gene pair whether the two genes share a cluster. Compute the cumulative difference distribution (CDF difference) between the original and perturbed connectivity matrices, and select the k whose CDF-difference area under the curve (AUC) is maximal; rationale is that perturbation exposes the data's natural structure, so the partition most stable under noise best groups genes with similar expression patterns. Finally assign each training gene to its subspace group under the selected partition and write the assignment table to gene_subspace_assignments.csv.

## Related tools

- **scISR** (Package whose second module performs the data perturbation technique to automatically group genes with similar patterns into smaller groups; selected partition defines gene subspaces for subspace regression imputation) — https://github.com/duct317/scISR
- **R stats** (Provides the k-means clustering used to partition genes during each perturbation-clustering cycle and the lm function for the subsequent regression)

## Examples

```
library(scISR); data('Goolam'); raw <- Goolam$data; imputed <- scISR(data = raw)
```

## Evaluation signals

- The reported partition must be the one maximizing the CDF-difference AUC over the tested range of k, not an arbitrary k.
- gene_subspace_assignments.csv contains one row per training gene and every gene carries a non-empty group label.
- Input matrix respects the filters: no gene expressed in fewer than two cells and no gene with less than one percent non-zero-valued entries.
- Connectivity matrices record same-cluster membership for every gene pair in both original and each perturbed dataset, so CDF-difference curves are computable for all k.
- Downstream sanity check: the gene groups feed scISR's third module (subspace regression using Pearson correlation to select the 10 most correlated genes), so group labels should exist for all imputable-gene regression inputs.

## Limitations

- No version is stated for the R stats package or the scISR software, limiting reproducibility of the perturbation-clustering implementation.
- Key robustness evidence is deferred to supplementary figures (e.g., S19, S21, S24) not included, so sensitivity of the selected k is not fully documented in the main text.
- The repository has no changelog ('_No changelog found._'), limiting reproducibility of the repository content used for this module.

## Evidence

- [intro] The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups.: "The second module utilizes a data perturbation technique26 to automatically group genes with similar patterns into smaller groups."
- [results] genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries.: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [abstract] The scISR method first uses a hypothesis-testing technique to identify zero-valued entries that are most likely affected by dropout events: "The scISR method first uses a hypothesis-testing technique to identify zero-valued entries that are most likely affected by dropout events"
- [readme] Next, training data is used for constructing a generalize linear regression model that is used for imputation on the imputable data.: "Next, training data is used for constructing a generalize linear regression model that is used for imputation on the imputable data."
