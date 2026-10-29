---
name: knn-label-transfer-large-dataset
description: Use when the scRNA-seq matrix exceeds scCAN's 5,000-cell training threshold
  and full network fusion on all cells is impractical, but per-cell cluster assignments
  are still needed for every cell.
when_to_use_negative:
- Dataset has 5,000 or fewer cells — scCAN clusters the full matrix directly without
  the train/test split (Module 3 is unnecessary)
- A pre-trained reference annotation exists and only label propagation to a query
  set is needed — this skill's k-NN step transfers labels only from scCAN's own training-subsample
  clusters, not from an external reference atlas
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAN
  role: Implements the Module 3 train/test split, k-nn SC and wMetaC selection of
    the best latent matrix, and k-NN label transfer of training cluster labels to
    the testing cells
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Source of the wMetaC (weighted meta-clustering) algorithm used to merge k-nn
    SC cluster assignments across latent matrices before label transfer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:c90e79c52f93ce2cf5ee48df3ae325ff9894fad4d3f75090b784852723507a0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# knn-label-transfer-large-dataset

## Summary

ScCAN's big-data strategy (Module 3): cluster only a random training subsample with network fusion and spectral clustering, then transfer labels to all remaining cells with a k-NN classifier, making autoencoder+SNF clustering tractable for datasets far above the 5,000-cell threshold (e.g. Zilionis, 34,558 cells).

## When to use

Use when the scRNA-seq matrix exceeds scCAN's 5,000-cell training threshold and full network fusion on all cells is impractical, but per-cell cluster assignments are still needed for every cell.

## When NOT to use

- Dataset has 5,000 or fewer cells — scCAN clusters the full matrix directly without the train/test split (Module 3 is unnecessary)
- A pre-trained reference annotation exists and only label propagation to a query set is needed — this skill's k-NN step transfers labels only from scCAN's own training-subsample clusters, not from an external reference atlas

## Inputs

- Normalized scRNA-seq expression matrix (cells x genes), e.g. Zilionis GSE127465 with 34,558 cells
- samp.size parameter (default 5,000 training cells; raised to 10,000 for rare-cell recovery)
- Latent representations from the stacked variational autoencoder (latent dimension d=15) for the training cells

## Outputs

- Per-cell cluster assignments for all cells (training clusters merged with k-NN-transferred testing labels)
- Estimated number of cell types from eigengap search (i = 2..15), e.g. 9 for Zilionis
- CSV of per-cell assignments, e.g. sccan_zilionis_clusters.csv

## How to apply

Because n (e.g. 34,558 cells in the Zilionis lung dataset) exceeds the 5,000-cell threshold, run scCAN Module 3: randomly split cells into a training set of samp.size cells (default 5,000; the task card raised it to samp.size=10,000 to resolve rare tRBC cells) and a testing set of the remaining cells. Cluster the training set with Module 2 — Gaussian similarity networks (sigma=0.5, n.neighbors=10) fused SNF-style, eigengap cluster-number estimation searching i from 2 to 15, and spectral clustering — after selecting the best latent representation via k-nn spectral clustering (k-nn SC) and weighted meta-clustering (wMetaC from SHARP) with ARI comparison. Classify each held-out testing cell with the k-NN algorithm using the training cluster labels, then merge training and testing assignments into the final clustering. Rationale: subsampling bounds the cost of building and fusing similarity networks while k-NN on the latent representations extends labels cheaply; increasing samp.size preserves rare populations that a 5,000-cell draw can miss.

## Related tools

- **scCAN** (Implements the Module 3 train/test split, k-nn SC and wMetaC selection of the best latent matrix, and k-NN label transfer of training cluster labels to the testing cells) — https://cran.r-project.org/package=scCAN
- **SHARP** (Source of the wMetaC (weighted meta-clustering) algorithm used to merge k-nn SC cluster assignments across latent matrices before label transfer)

## Examples

```
library(scCAN); if(max(data)>100) data <- log2(data+1); result <- scCAN(data, r.seed = 1); cluster <- result$cluster
```

## Evaluation signals

- tRBC F1 score on Zilionis with samp.size=10,000 reproduces the reported value of approximately 0.9
- Estimated cluster number from the eigengap method matches the 9 true Zilionis cell types
- Every testing cell receives a transferred label from a training cluster (no unassigned cells after merging train and test assignments)
- ARI comparison across latent matrices via k-nn SC plus wMetaC selects a consistent best latent representation across random seeds (r.seed)

## Limitations

- Default samp.size=5,000 can miss rare cell types (tRBC separation on Zilionis required samp.size=10,000 to reach F1 = 0.9), so small draws bias against rare populations
- No software version for scCAN or SHARP is stated in the article, and no accession/version pinning is available for reproducibility
- Label quality depends on the subsample: cells in the testing set inherit labels only through k-NN on the latent space, so errors in the training clustering propagate

## Evidence

- [methods] Module 3 big-data split: "we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells"
- [methods] k-NN label transfer: [evidence span withheld — anchor status: not_found]
- [methods] k-nn SC for latent matrix selection: [evidence span withheld — anchor status: not_found]
- [methods] wMetaC step: "The wMetaC algorithm is conducted through 5 steps"
- [readme] scCAN invocation: "result <- scCAN(data, r.seed = 1)"
