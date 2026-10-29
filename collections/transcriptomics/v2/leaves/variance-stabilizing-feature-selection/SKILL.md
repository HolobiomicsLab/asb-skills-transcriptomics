---
name: variance-stabilizing-feature-selection
description: Use when starting a GapClust rare-cell detection run on a raw scRNA-seq
  expression matrix loaded into R, and you need to select informative feature genes
  via variance-stabilising transformation prior to PCA embedding.
when_to_use_negative:
- The input is already a feature-selected or dimensionality-reduced embedding, not
  a raw scRNA-seq expression matrix.
- The goal is characterising abundant major cell types with toolkits such as Seurat
  or Scater rather than detecting rare cells.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: Provides the variance-stabilising transformation (vst) used to select informative
    feature genes from the scRNA-seq expression matrix
  repo: github.com/fabotao/GapClust
- name: irlba
  role: Performs truncated/approximate PCA on the feature-selected expression profile;
    suited to large sparse matrices (>= 2.3.3)
- name: rflann
  role: Builds the nearest-neighbour index over the PCA embeddings and queries K nearest
    neighbours for every cell (>= 1.8.4)
- name: Seurat
  role: R package prerequisite for GapClust (>= 3.1.0)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:27283fc44e2564470df46975126626fae25e642f32dc4eef7611f0663472cb83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  - Seurat
  - irlba
  - rflann
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# variance-stabilizing-feature-selection

## Summary

Apply GapClust's variance-stabilising transformation (vst) to a scRNA-seq expression matrix in R to select informative feature genes before low-dimensional projection, so that downstream nearest-neighbour distance statistics reflect true inter-cluster gaps rather than mean-dependent technical variance.

## When to use

Use when starting a GapClust rare-cell detection run on a raw scRNA-seq expression matrix loaded into R, and you need to select informative feature genes via variance-stabilising transformation prior to PCA embedding.

## When NOT to use

- The input is already a feature-selected or dimensionality-reduced embedding, not a raw scRNA-seq expression matrix.
- The goal is characterising abundant major cell types with toolkits such as Seurat or Scater rather than detecting rare cells.

## Inputs

- scRNA-seq expression matrix (count matrix) loadable in R
- GapClust R package installed from GitHub (fabotao/GapClust)

## Outputs

- Feature-selected expression profile (informative genes) ready for truncated PCA via irlba
- Low-dimensional PCA embedding consumed by rflann K-nearest-neighbour indexing

## How to apply

Load the scRNA-seq expression matrix into R (>= 3.4.0) and run GapClust's variance-stabilising transformation (vst) to pick informative feature genes. The rationale is that rare-cluster detection depends on the gap between a minor cluster and its neighbouring abundant cluster, so feature selection must remove mean-dependent dispersion before distances are computed. Feed the feature-selected expression profile into approximate PCA (irlba >= 2.3.3) to obtain the low-dimensional embedding used for K-nearest-neighbour querying with rflann (>= 1.8.4). Prerequisite packages per the README are Seurat >= 3.1.0 and e1071 >= 1.7-0.1, and GapClust installs via devtools::install_github('fabotao/GapClust'). The downstream acceptance criterion is skewness<k> > 2 computed from the D(m,k) neighbour-distance matrix, so feature selection quality directly affects this statistic.

## Related tools

- **GapClust** (Provides the variance-stabilising transformation (vst) used to select informative feature genes from the scRNA-seq expression matrix) — github.com/fabotao/GapClust
- **irlba** (Performs truncated/approximate PCA on the feature-selected expression profile; suited to large sparse matrices (>= 2.3.3))
- **rflann** (Builds the nearest-neighbour index over the PCA embeddings and queries K nearest neighbours for every cell (>= 1.8.4))
- **Seurat** (R package prerequisite for GapClust (>= 3.1.0))

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- distance_matrix.csv exists and contains an m cells x k neighbours nearest-neighbour distance matrix D(m,k) built from the vst-selected features
- The nearest-neighbour distance structure is consistent with GapClust's design criterion: a rare-cluster candidate k is accepted when skewness<k> > 2
- Feature gene selection was performed with GapClust's vst on the expression matrix before any PCA projection was run
- PCA embedding was computed with irlba (approximate/truncated PCA) rather than a dense PCA, appropriate for large sparse matrices
- R environment meets the stated prerequisites: R >= 3.4.0, Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in the article sections provided, so defaults may need tuning.
- GapClust's scope is rare cell detection; major cell types are characterised with toolkits such as Seurat and Scater instead.
- To discover additional hidden clusters, GapClust must be rerun with previously identified rare cell types removed.

## Evidence

- [other] Load the scRNA-seq expression matrix into R and apply GapClust's variance-stabilising transformation (vst) to select informative feature genes.: [evidence span withheld — anchor status: not_found]
- [intro] the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours"
- [abstract] Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [readme] rflann >=1.8.4 (GitHub: YeeJeremy/rflann) irlba >=2.3.3 e1071 >= 1.7-0.1: "Seurat >=  3.1.0
rflann >=1.8.4 (GitHub: YeeJeremy/rflann)
irlba >=2.3.3
e1071 >= 1.7-0.1"
- [discussion] Candidate k can be identified if the skewness<k> > 2.: "Candidate k can be identified if the skewness<k> > 2."
