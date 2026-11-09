---
name: nearest-neighbour-distance-computation
description: 'Use when building the distance layer of a rare-cell detection pipeline:
  you have a large scRNA-seq expression matrix and need per-cell nearest-neighbour
  distances (D<m,k>) feeding skewness>k>2 gap statistics.'
when_to_use_negative:
- The goal is comprehensive characterisation of major cell types rather than rare-cell
  detection — toolkits such as Seurat or Scater are the article's reference there
- The input is not a single-cell expression matrix but, e.g., an already-clustered
  or aggregated profile where per-cell neighbour distances are meaningless
- Computational resources permit exhaustive exact methods and the dataset is small
  enough that RaceID-style approaches are feasible without the unrealistic running
  time GapClust avoids
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: R package whose pipeline defines the vst feature selection and consumes the
    D(m,k) distance matrix to compute derivative statistics for rare-cell gap detection
  repo: github.com/fabotao/GapClust
- name: irlba
  role: Performs truncated/approximate PCA of the feature-selected sparse expression
    matrix to obtain the low-dimensional embedding (>=2.3.3 per README)
- name: rflann
  role: 'Builds the nearest-neighbour index over PCA embeddings and queries the K
    nearest neighbours for every cell (>=1.8.4, GitHub: YeeJeremy/rflann)'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:ab413e0392ab30d05ab152b8489b5b2357da0b211aea1b1c37bd153165656ae2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  - irlba
  - rflann
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nearest-neighbour-distance-computation

## Summary

Computes the per-cell K nearest-neighbour distance matrix D(m,k) from a low-dimensional PCA embedding of scRNA-seq data, providing the distance information from which GapClust derives first-order (∆D<m,k>) and second-order (∆∆D<m,k>) derivative statistics for rare-cell gap detection.

## When to use

Use when building the distance layer of a rare-cell detection pipeline: you have a large scRNA-seq expression matrix and need per-cell nearest-neighbour distances (D<m,k>) feeding skewness>k>2 gap statistics.

## When NOT to use

- The goal is comprehensive characterisation of major cell types rather than rare-cell detection — toolkits such as Seurat or Scater are the article's reference there
- The input is not a single-cell expression matrix but, e.g., an already-clustered or aggregated profile where per-cell neighbour distances are meaningless
- Computational resources permit exhaustive exact methods and the dataset is small enough that RaceID-style approaches are feasible without the unrealistic running time GapClust avoids

## Inputs

- scRNA-seq expression matrix (cells x genes, large/sparse)
- Number of neighbours K for the nearest-neighbour query
- Feature gene set selected by GapClust's vst transformation

## Outputs

- PCA embedding of cells (truncated/approximate PCA via irlba)
- distance_matrix.csv: D(m,k) matrix of m cells x k neighbour distances
- First-order ∆D<m,k> and second-order ∆∆D<m,k> derivative distance statistics per cell

## How to apply

Load the scRNA-seq expression matrix in R and apply GapClust's variance-stabilising transformation (vst) to select informative feature genes. Project the feature-selected profiles into low-dimensional space using truncated/approximate PCA from the irlba package, which handles large sparse matrices. Build a nearest-neighbour index over the PCA embeddings with the rflann package (>=1.8.4, from YeeJeremy/rflann) and query the K nearest neighbours for every cell. Assemble the returned neighbour distances into the D(m,k) matrix (m cells x k neighbours), from which first-order (∆D<m,k>) and second-order (∆∆D<m,k>) derivative distance statistics are computed; the second-order score captures the inter-cluster gap so that a rare-cluster candidate k is accepted when skewness<k> > 2.

## Related tools

- **GapClust** (R package whose pipeline defines the vst feature selection and consumes the D(m,k) distance matrix to compute derivative statistics for rare-cell gap detection) — github.com/fabotao/GapClust
- **irlba** (Performs truncated/approximate PCA of the feature-selected sparse expression matrix to obtain the low-dimensional embedding (>=2.3.3 per README))
- **rflann** (Builds the nearest-neighbour index over PCA embeddings and queries the K nearest neighbours for every cell (>=1.8.4, GitHub: YeeJeremy/rflann))

## Examples

```
devtools::install_github("fabotao/GapClust"); # then, within GapClust: irlba PCA on vst-selected features -> rflann index -> query K nearest neighbours -> D(m,k) -> distance_matrix.csv
```

## Evaluation signals

- distance_matrix.csv exists and has dimensions m cells x k neighbours
- Nearest-neighbour distance structure is consistent with GapClust's design criterion: for a cell in a minor cluster Cn, distances to neighbours outside Cn are far greater than to neighbours inside it
- Downstream gap statistics reproduce: a rare-cluster candidate k is accepted when skewness<k> > 2
- The PCA step uses an implementation suited to large sparse matrices (irlba), not a dense full SVD
- Package version prerequisites satisfied: Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1, R >= 3.4.0

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold beyond skewness>k>2) are reported in the article's discussion section
- Skewness>k>2 is the article's stated acceptance criterion; deviations from the expected bimodal distance structure (no clear inter-cluster gap) will undermine detection of minor clusters
- Results for many real datasets are deferred to supplementary figures not included in the main text, limiting direct validation material

## Evidence

- [methods] GapClust's first step is to obtain K nearest neighbours for all cells; for each cell m, the first-order (∆D<m,k>) and second-order (∆∆D<m,k>) derivative distance statistics are then computed: [evidence span withheld — anchor status: not_found]
- [intro] the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to: "for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater"
- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information.: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [readme] README package prerequisites list rflann and irlba with minimum versions: "rflann >=1.8.4 (GitHub: YeeJeremy/rflann)
irlba >=2.3.3"
- [discussion] Candidate k acceptance criterion stated in the discussion: "Candidate k can be identified if the skewness<k> > 2."
