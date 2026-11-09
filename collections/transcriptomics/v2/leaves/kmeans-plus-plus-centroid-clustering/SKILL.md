---
name: kmeans-plus-plus-centroid-clustering
description: Use when running the CCSF framework on single-cell gene expression or
  tabular genomic data and a fixed set of centroid locations is needed for cell-centroid
  similarity computation, dimensionality reduction, or ablation comparisons against
  PCA/MDS/GDA variants.
when_to_use_negative:
- The dataset contains data points from only one class — CCSF then behaves like kernel
  PCA and clustering adds no discriminative benefit
- The analysis goal is plain dimensionality reduction without class structure (e.g.,
  simple PCA visualization), where centroid-based similarity is unnecessary
- The number of target clusters/classes cannot be estimated or learned and no labels
  exist to validate numClass
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CCSF
  role: Python/MATLAB package whose CCSF class performs k-means++ centroid location
    (n_clusters/numClass) before cell-centroid similarity and Gram matrix computation
  repo: github.com/xinglab-ai/ccsf
- name: Splatter
  role: Simulator generating the 1,300 simulated datasets on which clustered CCSF
    variants are benchmarked with DeMAP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100840
    title: Leveraging cell-cell similarity for high-performance spatial and temporal
      cellular mappings from gene expression data
  source_content_hash: sha256:5083c82eef8feab887507b5c8043aed1782f915365dcab8048f7b1fe23d4aa4b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinglab-ai/ccsf
  tools:
  - CCSF
  - Splatter
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# k-means++ centroid clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Locates P cluster centroids within a CCSF pipeline by minimizing the distance between centroids and all data points, providing the cell-centroid structure on which cell-centroid similarity and the Gram matrix are computed. The choice of P (numClass) plays the role that the number of principal components plays in PCA.

## When to use

Use when running the CCSF framework on single-cell gene expression or tabular genomic data and a fixed set of centroid locations is needed for cell-centroid similarity computation, dimensionality reduction, or ablation comparisons against PCA/MDS/GDA variants.

## When NOT to use

- The dataset contains data points from only one class — CCSF then behaves like kernel PCA and clustering adds no discriminative benefit
- The analysis goal is plain dimensionality reduction without class structure (e.g., simple PCA visualization), where centroid-based similarity is unnecessary
- The number of target clusters/classes cannot be estimated or learned and no labels exist to validate numClass

## Inputs

- Gene expression count matrix (e.g., CSV of simulated or scRNA-seq data, rows = cells, columns = genes)
- Z-scored data sample (up to 2,000 randomly selected points) for metric/numClass learning
- Class labels (e.g., tcga_label.csv) for downstream class-discriminative optimization

## Outputs

- Set of P centroid locations (n_clusters = numClass) used for cell-centroid similarity computation
- Cluster assignment of cells used in the CCSF cMAP/cPHATE/cSpaRc embeddings
- Per-variant DeMAP scores when centroids are held fixed across ablation variants (ablation_comparison.csv)

## How to apply

Instantiate the CCSF object with the learned or preset number of classes (n_clusters=numClass, e.g., numClass=33 for organoid/TCGA-style data, numClass=7 for BDTNP spatial mapping), mirroring the choice of the number of principal components in PCA. If the number of classes is unknown, learn it via manifolder.metric_learning on a z-scored random sample of 2,000 data points (data with more than 2,000 points are subsampled to 2,000 for metric learning). Cluster with k-means++ initialization (numInitCls = floor(data.shape[0]/500) initial classes) under a fixed random_state for reproducibility. In ablation studies, keep the resulting k-means++ centroid locations fixed across all dimensionality-reduction variants (linear, GDA-Gaussian, GDA-polynomial, QDA, MDS, PCA) so only the class-discriminative optimization step changes.

## Related tools

- **CCSF** (Python/MATLAB package whose CCSF class performs k-means++ centroid location (n_clusters/numClass) before cell-centroid similarity and Gram matrix computation) — github.com/xinglab-ai/ccsf
- **Splatter** (Simulator generating the 1,300 simulated datasets on which clustered CCSF variants are benchmarked with DeMAP)

## Examples

```
manifolder = CCSF(n_clusters=numClass, num_comp=num_comp, metric=metric, random_state=rngState); embedding_CPHATE = manifolder.cPHATE(data=feed_data)
```

## Evaluation signals

- Centroid locations (k-means++) remain fixed across all ablation variants (linear, GDA-Gaussian, GDA-polynomial, QDA, MDS, PCA) so only the optimization step differs
- ablation_comparison.csv contains one DeMAP row per variant and the linear (CCSF) variant attains the highest DeMAP on Splatter simulations
- numInitCls equals floor(data.shape[0]/500) and, when datasets exceed 2,000 points, metric learning uses exactly 2,000 randomly sampled points
- random_state/rngState is fixed (e.g., 0) and repeated initializations change DEMaP by <1% (zebrafish) or produce no change (Drosophila)
- numClass choice behaves like choosing the number of principal components in PCA — verify downstream cMAP/cPHATE components = numClass - 1

## Limitations

- If the dataset contains data points from only one class, CCSF works similarly to kernel PCA; its only benefits over PCA are data linearization and computational speed
- The number of classes (numClass) must be chosen similarly to choosing the number of principal components in PCA — a misspecified numClass affects all downstream similarity and reduction steps
- Datasets with more than 2,000 data points are subsampled to 2,000 random points for metric learning, so very large datasets rely on sampling for centroid/metric initialization

## Evidence

- [intro] P centroids are located by minimizing the distance between them and all the data points: "(2) P centroids are located by minimizing the distance between them and all the data points."
- [readme] numClass choice analogous to PCA components: "numClass=33 # Choice of the numClass is similar to choosing the number of principal components in PCA"
