---
name: big-data-train-test-knn-clustering
description: Use when the scRNA-seq input matrix has more than 5,000 cells (e.g. Zilionis
  with 34,558 cells) and a full network-fusion spectral clustering run is too expensive,
  so clustering must be learned on a subsample and extended to the rest.
when_to_use_negative:
- The dataset has 5,000 or fewer cells — scCAN can run network fusion and spectral
  clustering on all cells directly without the train/test split
- The input is raw unnormalized counts that have not been preprocessed (scCAN expects
  an expression matrix; apply log2(x+1) if max > 100 per the README)
- The goal is supervised cell-type classification rather than unsupervised clustering
  — k-NN here only propagates training-cluster labels, not known annotations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAN
  role: 'Implements the full pipeline: min-max scaling, autoencoder feature selection,
    stacked VAE, network fusion, eigengap spectral clustering on the training set,
    and k-NN classification of testing cells'
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Benchmark clustering method compared against scCAN for accuracy and scalability
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:3226771ee35c81d19b21b9a42ca57b8c89c2d88e2772e2540de45b6541f31b70
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# big-data-train-test-knn-clustering

## Summary

scCAN's Module 3 strategy for scaling spectral clustering to large scRNA-seq datasets: randomly split cells into a 5,000-cell training set and a testing set, cluster the training cells with network fusion and eigengap-based spectral clustering, then assign the remaining cells via k-NN classification.

## When to use

Use when the scRNA-seq input matrix has more than 5,000 cells (e.g. Zilionis with 34,558 cells) and a full network-fusion spectral clustering run is too expensive, so clustering must be learned on a subsample and extended to the rest.

## When NOT to use

- The dataset has 5,000 or fewer cells — scCAN can run network fusion and spectral clustering on all cells directly without the train/test split
- The input is raw unnormalized counts that have not been preprocessed (scCAN expects an expression matrix; apply log2(x+1) if max > 100 per the README)
- The goal is supervised cell-type classification rather than unsupervised clustering — k-NN here only propagates training-cluster labels, not known annotations

## Inputs

- Normalized scRNA-seq expression matrix with cells as rows and genes as columns (e.g. Zilionis GSE127465, 34,558 human lung cells)
- Optional cell-type annotation labels for held-out evaluation (e.g. tRBC labels for F1 scoring)

## Outputs

- Cluster assignments for all cells: spectral-clustering labels for the 5,000 training cells plus k-NN labels for the testing cells
- Estimated number of cell types from the eigengap method
- Per-cluster labels enabling downstream sub-clustering (e.g. stage-two clusters 2_1, 2_2)

## How to apply

When n exceeds 5,000 cells, split the data into a training set of 5,000 randomly selected cells and a testing set containing the remaining cells. Run the standard scCAN pipeline on the training cells: min-max rescaling to 0–1, feature selection with the non-negative-kernel autoencoder keeping the top 5,000 genes, stacked VAE with latent dimension d=15, then SNF-based network fusion (k=30 neighbors) and eigengap-determined spectral clustering. Use the resulting training cluster labels to classify each testing cell with a k-NN algorithm. This trains the expensive network-fusion step only on the subsample while propagating labels to the full dataset; the same logic supports multi-stage (two-stage) re-clustering of individual clusters to resolve rare cell types.

## Related tools

- **scCAN** (Implements the full pipeline: min-max scaling, autoencoder feature selection, stacked VAE, network fusion, eigengap spectral clustering on the training set, and k-NN classification of testing cells) — https://cran.r-project.org/package=scCAN
- **SHARP** (Benchmark clustering method compared against scCAN for accuracy and scalability)

## Examples

```
library(scCAN); result <- scCAN(data, r.seed = 1); cluster <- result$cluster
```

## Evaluation signals

- Verify the training set contains exactly 5,000 randomly selected cells and the testing set the remainder
- Confirm eigengap-based spectral clustering converges to a plausible number of clusters on the fused network
- Reproduce the reported result on Zilionis: two-stage clustering with default samp.size=5,000 separates rare tRBC cells with F1 = 1 (within numerical tolerance)
- Check every cell (training + testing) receives exactly one cluster label with no unassigned cells
- Compute adjusted Rand Index against known labels; the README example prints ARI from scCAN::adjustedRandIndex

## Limitations

- k-NN classification of testing cells depends on training-cluster quality; errors in the 5,000-cell subsample propagate to all remaining cells
- Random subsampling introduces seed dependence — set r.seed for reproducibility (README uses r.seed = 1)
- Very rare cell types may be missed in a single stage; the article recommends multi-stage (two-stage) re-clustering to detect them
- No software version numbers are stated for scCAN or the compared tools in the article

## Evidence

- [methods] we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells: "we split the cells into two sets: a training set of 5000 randomly selected cells and a testing set that consists of the remaining cells"
- [methods] we use k-NN algorithm to classify the remaining cells using cluster's labels obtained from the training data: [evidence span withheld — anchor status: not_found]
- [methods] we use the eigengap method48 to determine the number of clusters: "we use the eigengap method48 to determine the number of clusters"
- [results] In the two-stage clustering with the default samp.size=5,000, stage two split cluster 2 into sub-clusters 2_1 and 2_2, completely separating the rare tRBC cells from the rest with an F1 score of 1.: [evidence span withheld — anchor status: not_found]
- [readme] Add log transformation: `if(max(data)>100) data <- log2(data + 1)`: "if(max(data)>100) data <- log2(data + 1)"
