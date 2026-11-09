---
name: sparse-expression-matrix-processing
description: Use when a large, sparse single-cell expression matrix must be reduced
  to per-cell nearest-neighbour distance statistics (D(m,k)) so GapClust can score
  second-order distance changes for rare cell detection.
when_to_use_negative:
- Input is not a raw/large scRNA-seq expression matrix — e.g. when rare cell types
  are already known and GapClust should simply be rerun with those cells removed to
  discover hidden clusters
- When the analysis targets comprehensive characterisation of major cell types rather
  than rare cell detection — toolkits like Seurat or Scater are the article's designated
  choice for that task
- Datasets small enough that exact nearest-neighbour computation is cheap — the irlba/rflann
  approximate machinery targets ultra-large, sparse datasets
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GapClust
  role: Provides the vst variance-stabilising feature selection and the derivative
    distance statistics (∆D<m,k>, ∆∆D<m,k>) computed from the D(m,k) matrix
  repo: github.com/fabotao/GapClust
- name: irlba
  role: Truncated/approximate PCA of the feature-selected sparse expression matrix
    (>= 2.3.3)
- name: rflann
  role: Builds a nearest-neighbour index over the PCA embeddings and queries the K
    nearest neighbours for every cell (>= 1.8.4)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:099e2c57d88bc1e217fc2bd36cf1874f45437b9f1788fc167a4589b8233a3d14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  - irlba
  - rflann
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sparse-expression-matrix-processing

## Summary

Converts a large sparse scRNA-seq expression matrix into the per-cell K nearest-neighbour distance information (the D(m,k) matrix) that GapClust needs for gap-based rare cell detection, via variance-stabilising feature selection, approximate PCA, and approximate nearest-neighbour search.

## When to use

Use when a large, sparse single-cell expression matrix must be reduced to per-cell nearest-neighbour distance statistics (D(m,k)) so GapClust can score second-order distance changes for rare cell detection.

## When NOT to use

- Input is not a raw/large scRNA-seq expression matrix — e.g. when rare cell types are already known and GapClust should simply be rerun with those cells removed to discover hidden clusters
- When the analysis targets comprehensive characterisation of major cell types rather than rare cell detection — toolkits like Seurat or Scater are the article's designated choice for that task
- Datasets small enough that exact nearest-neighbour computation is cheap — the irlba/rflann approximate machinery targets ultra-large, sparse datasets

## Inputs

- scRNA-seq expression matrix (large, sparse cell x gene count matrix)
- R environment with GapClust (devtools::install_github("fabotao/GapClust"))

## Outputs

- distance_matrix.csv containing the D(m,k) matrix of m cells x k nearest-neighbour distances on the top-50 PCA embedding
- Low-dimensional (truncated PCA) embedding of feature-selected expression profiles

## How to apply

Load the scRNA-seq count matrix into R and apply GapClust's variance-stabilising transformation (vst) to select informative feature genes. Project the feature-selected profile into low-dimensional space with truncated/approximate PCA implemented by irlba, which is suited to large sparse matrices (README pins irlba >= 2.3.3, rflann >= 1.8.4, Seurat >= 3.1.0, e1071 >= 1.7-0.1). Build a nearest-neighbour index over the PCA embeddings with rflann (YeeJeremy/rflann) and query the K nearest neighbours for every cell. Assemble the returned neighbour distances into the D(m,k) matrix (m cells x k neighbours) and export it as distance_matrix.csv; GapClust then computes the first-order (∆D<m,k>) and second-order (∆∆D<m,k>) derivative distance statistics from it, accepting a rare-cluster candidate k when skewness<k> > 2. The rationale is that for cells in a minor cluster Cn, distances to neighbours outside the cluster far exceed distances to neighbours inside it, so the gap is captured by these derivative statistics.

## Related tools

- **GapClust** (Provides the vst variance-stabilising feature selection and the derivative distance statistics (∆D<m,k>, ∆∆D<m,k>) computed from the D(m,k) matrix) — github.com/fabotao/GapClust
- **irlba** (Truncated/approximate PCA of the feature-selected sparse expression matrix (>= 2.3.3))
- **rflann** (Builds a nearest-neighbour index over the PCA embeddings and queries the K nearest neighbours for every cell (>= 1.8.4))

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- distance_matrix.csv exists and has m rows (one per cell) x k columns (one per queried neighbour) on the top-50 PCA embedding
- Nearest-neighbour distance structure in distance_matrix.csv is consistent with GapClust's design criterion: a rare-cluster candidate k is accepted when skewness<k> > 2 computed from the distance distribution
- PCA step ran on the vst-selected feature genes, not all genes, and used irlba (suitable for large sparse matrices)
- Neighbour index was built with rflann over the PCA embeddings, and distances for cells in known minor clusters show a marked jump between in-cluster and out-of-cluster neighbours
- Package versions satisfy the README prerequisites: Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1, R >= 3.4.0

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported for the real-dataset runs in the article's discussion section
- GapClust's scope is rare cell detection; major cell types should be characterised with toolkits such as Seurat or Scater instead
- Previously identified rare cell types should be removed before rerunning to discover additional hidden clusters

## Evidence

- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information.: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [intro] Design rationale: distances to neighbours outside a minor cluster exceed distances to neighbours inside it.: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn"
- [readme] README prerequisites for the R environment used in this skill.: "Seurat >=  3.1.0
rflann >=1.8.4 (GitHub: YeeJeremy/rflann)
irlba >=2.3.3
e1071 >= 1.7-0.1"
- [discussion] Candidate acceptance threshold used on the distance distribution.: "Candidate k can be identified if the skewness<k> > 2."
