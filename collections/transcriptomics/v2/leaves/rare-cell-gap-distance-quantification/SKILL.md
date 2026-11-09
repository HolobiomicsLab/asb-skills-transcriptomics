---
name: rare-cell-gap-distance-quantification
description: Use when you have a large scRNA-seq expression matrix and need per-cell
  nearest-neighbour distance statistics (D<m,k>, ∆D<m,k>, ∆∆D<m,k>, skewness<k>) to
  detect rare cell clusters via inter-cluster gaps.
when_to_use_negative:
- If the input is already a feature table or low-dimensional embedding with no gene
  expression data, the vst-based feature selection step is unnecessary and this skill's
  first stage does not apply
- When the goal is major/abundant cell type characterisation with toolkits like Seurat
  or Scater rather than rare (minor) cluster detection
edam_operation: http://edamontology.org/operation_3795
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GapClust
  role: 'Main algorithm: defines the vst feature selection, derivative distance statistics,
    and skewness>k>2 acceptance criterion'
  repo: github.com/fabotao/GapClust
- name: irlba
  role: Performs truncated/approximate PCA to project feature-selected expression
    profiles into low-dimensional space, suited to large sparse matrices (irlba >=
    2.3.3)
- name: rflann
  role: Builds a nearest-neighbour index over PCA embeddings and queries the K nearest
    neighbours for every cell (rflann >= 1.8.4)
  repo: YeeJeremy/rflann
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:98c0ae4b6f34779ab52226c6b096d78e11b35ce8c878606947e8b431eab7f579
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  - irlba
  - rflann
  edam_operation: http://edamontology.org/operation_3795
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# rare-cell-gap-distance-quantification

## Summary

Quantifies the inter-cluster gap used by GapClust for rare cell detection by building a per-cell K-nearest-neighbour distance matrix D(m,k) over a low-dimensional PCA embedding, then computing first-order (∆D<m,k>) and second-order (∆∆D<m,k>) derivative distance statistics and a skewness score per candidate cluster size k.

## When to use

Use when you have a large scRNA-seq expression matrix and need per-cell nearest-neighbour distance statistics (D<m,k>, ∆D<m,k>, ∆∆D<m,k>, skewness<k>) to detect rare cell clusters via inter-cluster gaps.

## When NOT to use

- If the input is already a feature table or low-dimensional embedding with no gene expression data, the vst-based feature selection step is unnecessary and this skill's first stage does not apply
- When the goal is major/abundant cell type characterisation with toolkits like Seurat or Scater rather than rare (minor) cluster detection

## Inputs

- scRNA-seq gene-by-cell expression matrix (large, typically sparse)
- Feature gene set selected by GapClust's variance-stabilising transformation (vst)
- Truncated PCA embedding of cells (computed via irlba)
- Number of nearest neighbours K per cell

## Outputs

- distance_matrix.csv — per-cell K-nearest-neighbour distances D(m,k), m cells x k neighbours
- First-order derivative distance statistics ∆D<m,k>
- Second-order derivative distance statistics ∆∆D<m,k>
- Per-candidate cluster-size skewness scores (skewness<k>) with candidate k flagged when skewness<k> > 2

## How to apply

Load the scRNA-seq expression matrix into R and apply GapClust's variance-stabilising transformation (vst) to select informative feature genes. Project the feature-selected expression profile into low-dimensional space using truncated/approximate PCA from the irlba package (>= 2.3.3), which suits large sparse matrices. Build a nearest-neighbour index over the PCA embeddings with the rflann package (>= 1.8.4, from YeeJeremy/rflann) and query the K nearest neighbours for every cell. Assemble the returned neighbour distances into the D(m,k) matrix (m cells x k neighbours, e.g. written to distance_matrix.csv), then compute the first-order (∆D<m,k>) and second-order (∆∆D<m,k>) derivative distance statistics from it. The rationale: for a data point in a minor cluster Cn, the distance to nearest neighbours outside Cn is far greater than to neighbours inside it, and a second-order derivative type distance score fully utilises this gap information; a rare-cluster candidate k is accepted when skewness<k> > 2.

## Related tools

- **GapClust** (Main algorithm: defines the vst feature selection, derivative distance statistics, and skewness>k>2 acceptance criterion) — github.com/fabotao/GapClust
- **irlba** (Performs truncated/approximate PCA to project feature-selected expression profiles into low-dimensional space, suited to large sparse matrices (irlba >= 2.3.3))
- **rflann** (Builds a nearest-neighbour index over PCA embeddings and queries the K nearest neighbours for every cell (rflann >= 1.8.4)) — YeeJeremy/rflann

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- distance_matrix.csv exists and its D(m,k) structure is m cells x k neighbours with plausible nearest-neighbour distances relative to the PCA embedding scale
- For cells belonging to a rare cluster, distances to neighbours outside the cluster are markedly greater than to neighbours inside it (the gap signature GapClust exploits)
- Rare-cluster candidate k is accepted only when skewness<k> > 2 computed from the distance distribution
- Second-order derivative statistics ∆∆D<m,k> highlight the distance-change variation at the cluster boundary, consistent with GapClust's design criterion
- Install prerequisites satisfied per README: R >= 3.4.0, Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1

## Limitations

- No parameter settings (e.g. K, skewness threshold values beyond the stated skewness<k> > 2) are reported in the discussion section
- Related methods compared lack version numbers (GiniClust, RaceID, CellSIUS, FiRE, EDGE), so reproducibility of benchmark context is limited
- Existing alternatives fail at scale: RaceID precluded by unrealistic running time on thousands of cells, and GiniClust reported to fail processing very large datasets — distances must therefore be computed efficiently (rflann/irlba) for ultra-large datasets

## Evidence

- [intro] the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information.: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [discussion] Candidate k can be identified if the skewness<k> > 2.: "Candidate k can be identified if the skewness<k> > 2."
- [readme] GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out through delicately designed statistics.: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out"
- [readme] rflann >=1.8.4 (GitHub: YeeJeremy/rflann): "rflann >=1.8.4 (GitHub: YeeJeremy/rflann)"
