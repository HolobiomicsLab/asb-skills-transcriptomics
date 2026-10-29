---
name: similarity-network-fusion
description: Use when multiple low-dimensional latent matrices of single-cell data
  are available and a single robust cell-cluster assignment must be derived by fusing
  per-representation cell-similarity networks.
when_to_use_negative:
- The input expression matrix has not been normalized and min-max rescaled — the networks
  would be built from unfiltered, unscaled data rather than VAE latents
- Fewer than two latent representations exist, so there is nothing to fuse via the
  SNF update
- The goal is de novo feature/gene selection rather than cell clustering — Module
  1's non-negative kernel autoencoder (top 5000 genes, bottleneck 50) is the correct
  stage
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scCAN
  role: R package implementing the SNF-based network fusion and spectral clustering
    (Module 2) over VAE latent representations
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Source of the wMetaC weighted meta-clustering used to select the best latent
    matrix via k-nn spectral clustering and ARI comparison
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:ad370e1b6ad2f0de69b40ee137bf53493908bb4cbc1aaa8c095fd5252d0138cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# similarity-network-fusion

## Summary

Applies SNF-style iterative fusion of multiple cell-similarity networks built from scCAN's stacked-VAE latent representations, then partitions the fused network with spectral clustering to assign cells to clusters.

## When to use

Use when multiple low-dimensional latent matrices of single-cell data are available and a single robust cell-cluster assignment must be derived by fusing per-representation cell-similarity networks.

## When NOT to use

- The input expression matrix has not been normalized and min-max rescaled — the networks would be built from unfiltered, unscaled data rather than VAE latents
- Fewer than two latent representations exist, so there is nothing to fuse via the SNF update
- The goal is de novo feature/gene selection rather than cell clustering — Module 1's non-negative kernel autoencoder (top 5000 genes, bottleneck 50) is the correct stage

## Inputs

- Multiple latent representation matrices of cells produced by the stacked variational autoencoder (d=15)
- Already-normalized scRNA-seq expression matrix (cells as rows, genes as columns), min-max rescaled to [0,1]

## Outputs

- Fused cell-similarity network P
- Per-cell cluster assignments (cluster_assignments.csv / sccan_zilionis_clusters.csv)
- Estimated number of cell types from the eigengap method

## How to apply

After the stacked variational autoencoder (latent dimension d=15, re-parameterization trick z = mu + sigma*N(0,1)) produces multiple latent representation matrices, build one cell-similarity network per latent matrix using a Gaussian similarity kernel (sigma=0.5) with local affinity from n.neighbors=10 nearest neighbors (task_006 reports k=30 neighbors). Iteratively fuse the networks with the SNF-style update P(v) = S(v) x (sum of other P(k)/(n-1)) x S(v)^T, adapted from the SNF method. Estimate the number of clusters with the eigengap method on the Laplacian L = D - A, searching i from 2 to 15 and choosing the smallest i among the two highest eigengaps to avoid overestimation, then run spectral clustering on the fused network P to partition cells. For datasets exceeding 5,000 cells (e.g., Zilionis, 34,558 cells), cluster only a random training subsample (samp.size=5,000 default; 10,000 for rare types) and classify the remaining testing cells with k-NN using the training cluster labels.

## Related tools

- **scCAN** (R package implementing the SNF-based network fusion and spectral clustering (Module 2) over VAE latent representations) — https://cran.r-project.org/package=scCAN
- **SHARP** (Source of the wMetaC weighted meta-clustering used to select the best latent matrix via k-nn spectral clustering and ARI comparison)

## Examples

```
library(scCAN); result <- scCAN(data, r.seed = 1); cluster <- result$cluster; ari <- round(scCAN::adjustedRandIndex(cluster,label), 2)
```

## Evaluation signals

- Adjusted Rand Index (ARI) between predicted cluster assignments and published cell-type labels (e.g., Pollen) matches the reported values
- Eigengap-estimated number of clusters matches the known number of cell types (e.g., 9 true Zilionis cell types)
- Rare-type separation reproduces reported scores (tRBC F1 ≈ 0.9 with samp.size=10,000)
- Fused network P is a valid similarity matrix after iterative SNF updates converge
- Cluster labels for training and k-NN-classified testing cells merge into a complete per-cell assignment file

## Limitations

- Default 5,000-cell training subsample can miss rare cell types; samp.size must be raised (e.g., 10,000) to separate rare populations like tRBC
- Eigengap search is bounded by the user-controlled hyperparameter i from 2 to 15; cluster counts outside this range cannot be detected
- No software version is stated for scCAN or SHARP in the methods, hindering exact reproducibility

## Evidence

- [methods] SNF update formula: [evidence span withheld — anchor status: not_found]
- [methods] SNF provenance: "The network fusion approach is adapted from SNF method45–47"
- [methods] Eigengap and spectral clustering: "we use the eigengap method48 to determine the number of clusters"
- [methods] Spectral clustering on fused network: "we use spectral clustering49 to partition the cells in the fused network P"
- [methods] Big-data train/test split: "we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells"
- [readme] README example invocation: "result <- scCAN(data, r.seed = 1)"
- [methods] k-NN classification of testing cells: [evidence span withheld — anchor status: not_found]
