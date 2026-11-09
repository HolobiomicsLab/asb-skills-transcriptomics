---
name: eigengap-cluster-number-estimation
description: Use when clustering scRNA-seq data with scCAN (or a similar spectral/network-fusion
  pipeline) and the true number of cell types is unknown, so k must be estimated from
  the fused similarity network before spectral clustering.
when_to_use_negative:
- The number of clusters is known a priori or fixed by the experimental design — pass
  k directly instead of estimating it
- The input is raw high-dimensional counts rather than the fused similarity network
  from scCAN's Modules 1–2; the eigengap method here assumes spectral decomposition
  of the network-fused matrix
- The dataset's expected cell-type count exceeds the scanned eigengap range (e.g.,
  more than 15 types under the default 2:15 scan) without extending the range
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAN
  role: R package implementing the full three-module pipeline in which the eigengap
    method estimates cluster number (Module 2) before spectral clustering; available
    on CRAN and GitHub
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Provides wMetaC weighted meta-clustering used alongside k-nn spectral clustering
    to select the best latent matrix after cluster-number estimation
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:65a8709ee77d6e9d7844f4929891232032d2f0ac0576d0add8a4e696f81e429d
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# eigengap-cluster-number-estimation

## Summary

Determines the number of single-cell clusters by applying the eigengap method to the Laplacian (L = D - A) of the network-fused cell-similarity matrix produced by scCAN, scanning i from 2 to 15 and selecting the smallest i among the two highest eigengaps.

## When to use

Use when clustering scRNA-seq data with scCAN (or a similar spectral/network-fusion pipeline) and the true number of cell types is unknown, so k must be estimated from the fused similarity network before spectral clustering.

## When NOT to use

- The number of clusters is known a priori or fixed by the experimental design — pass k directly instead of estimating it
- The input is raw high-dimensional counts rather than the fused similarity network from scCAN's Modules 1–2; the eigengap method here assumes spectral decomposition of the network-fused matrix
- The dataset's expected cell-type count exceeds the scanned eigengap range (e.g., more than 15 types under the default 2:15 scan) without extending the range

## Inputs

- Network-fused cell-similarity matrix P (or the individual SNF-style similarity networks built from each stacked-VAE latent representation)
- Normalized scRNA-seq expression matrix (cells as rows, genes as columns), pre-filtered by the non-negative-kernel autoencoder
- Eigengap search range (default i = 2:15; widened, e.g., k = 10:50, for big-data runs)

## Outputs

- Estimated number of clusters (number of true cell types)
- Per-cell cluster assignments from spectral clustering of the fused network (e.g., sccan_zilionis_clusters.csv, cluster_assignments.csv)

## How to apply

After scCAN's Module 1 (min-max rescaling to [0,1], non-negative-kernel autoencoder feature selection of the top 5000 genes with non-zero coefficients) and stacked variational autoencoder (latent dimension d=15), build a cell-similarity network for each latent matrix using a Gaussian kernel (sigma=0.5) and fuse them with the SNF-style update P(v) = S(v) x (sum of other P(k)/(n-1)) x S(v)^T. Compute the Laplacian L = D - A of the fused network and apply the eigengap method, scanning the user-controlled hyperparameter i from 2 to 15; select the smallest i among the two highest eigengaps to avoid overestimating the number of cell types. The selected i becomes k for the subsequent spectral clustering of the fused network P. In scCAN's big-data mode (n > 5000), the eigengap search can span a wider range (e.g., k=10:50 for the 1.3M-cell 10X brain dataset), where Module 3 clusters only a 5000-cell training set and classifies the rest via k-NN.

## Related tools

- **scCAN** (R package implementing the full three-module pipeline in which the eigengap method estimates cluster number (Module 2) before spectral clustering; available on CRAN and GitHub) — https://cran.r-project.org/package=scCAN
- **SHARP** (Provides wMetaC weighted meta-clustering used alongside k-nn spectral clustering to select the best latent matrix after cluster-number estimation)

## Examples

```
result <- scCAN(data, r.seed = 1); cluster <- result$cluster
```

## Evaluation signals

- Eigengap-estimated cluster count matches the known number of true cell types (e.g., 9 for the Zilionis lung dataset; matches known Pollen cell types)
- Downstream ARI against published cell_type1 labels reaches ~0.92 on Pollen (tolerance ±0.05)
- Big-data run on the 1.3M-cell 10X brain dataset (sparse=TRUE, r.seed=1, k=10:50) yields the expected 19 clusters
- Rare-cell separation validated by F1 score (tRBC F1 ≈ 0.9 on Zilionis with samp.size=10,000)
- Selection rule respected: the smallest i among the two highest eigengaps is chosen, avoiding cluster-number overestimation

## Limitations

- The default eigengap scan i = 2:15 caps the detectable number of cell types; large datasets need a widened range (e.g., k = 10:50 for the 1.3M-cell brain dataset)
- The scan range is a user-controlled hyperparameter, so results depend on the chosen bounds
- Estimation operates on the fused network produced by the autoencoder pipeline, so errors in feature selection (top 5000 genes) or the VAE latent space propagate to the cluster-number estimate
- No software version numbers for scCAN or compared tools (CIDR, SEURAT3, Monocle3, SHARP, SCANPY) are stated in the article

## Evidence

- [methods] we use the eigengap method48 to determine the number of clusters: "we use the eigengap method48 to determine the number of clusters"
- [methods] we use spectral clustering49 to partition the cells in the fused network P: "we use spectral clustering49 to partition the cells in the fused network P"
- [abstract] scCAN: (1) correctly estimates the number of true cell types: "scCAN: (1) correctly estimates the number of true cell types"
- [readme] result <- scCAN(data, r.seed = 1): "result <- scCAN(data, r.seed = 1)"
