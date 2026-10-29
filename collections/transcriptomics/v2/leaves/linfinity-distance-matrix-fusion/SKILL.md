---
name: linfinity-distance-matrix-fusion
description: Use when CITE-seq data with both transcriptome and surface-protein (ADT)
  modalities has been preprocessed and per-modality cell-cell distance matrices must
  be fused for joint Louvain clustering or dimensionality reduction.
when_to_use_negative:
- Datasets with a very large number of ADT features (>50) or massive cell counts (e.g.
  170K cells with 228 ADT features), where Seurat 4 is the better option and LinQ-View
  failed
- Unimodal scRNA-seq data with no surface-protein modality, since there is no second
  distance matrix to fuse
- Cases where a feature-level integration (e.g. Seurat 4 WNN) is preferred over distance-matrix-level
  fusion
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View
  role: Toolkit implementing the sigmoid filter, linear alpha-rescaling, and L-infinity
    joint distance fusion for CITE-seq modalities
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: Provides the Louvain community-detection clustering applied downstream of
    the fused distance matrix (default in LinQ-View)
  repo: https://satijalab.org/seurat/
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:f6d36df8b07862a119d21f3497aec963f608f97b9c5bbc0c0d8b2eb166df326d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - LinQ-View v0.99
  - R
  - Seurat
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# linfinity-distance-matrix-fusion

## Summary

LinQ-View's method for combining per-modality cell-cell pairwise distance matrices from CITE-seq mRNA and surface-protein (ADT) data into a single joint distance matrix, using a sigmoid filter on ADT distances, linear alpha-rescaling, and an elementwise L-infinity (maximum) fusion.

## When to use

Use when CITE-seq data with both transcriptome and surface-protein (ADT) modalities has been preprocessed and per-modality cell-cell distance matrices must be fused for joint Louvain clustering or dimensionality reduction.

## When NOT to use

- Datasets with a very large number of ADT features (>50) or massive cell counts (e.g. 170K cells with 228 ADT features), where Seurat 4 is the better option and LinQ-View failed
- Unimodal scRNA-seq data with no surface-protein modality, since there is no second distance matrix to fuse
- Cases where a feature-level integration (e.g. Seurat 4 WNN) is preferred over distance-matrix-level fusion

## Inputs

- Symmetric cell-cell pairwise distance matrix computed from the mRNA (transcriptome) modality (e.g. Euclidean distances in PCA space)
- Symmetric cell-cell pairwise distance matrix computed from the surface-protein (ADT) modality
- CITE-seq object with transcriptome and surface-protein modalities (e.g. Seurat object from GSE100866, 8,617 CBMCs)

## Outputs

- Joint cell-cell distance matrix Djoint (elementwise max of rescaled modality matrices)
- Scalar weighting parameter alpha in [0,1]
- Sigmoid-filtered ADT distance matrix with parameters n=10, k=0.5

## How to apply

First apply a vectorized sigmoid filter F(x)=x/(1+e^{-n(x-k)}) with n=10 and k=0.5 elementwise to the ADT Euclidean cell-cell distance matrix, which the paper introduces to dampen spurious distance variation in the ADT modality. Compute cell-cell pairwise distances for each modality after standard pre-processing (QC, normalization, scaling, PCA). Rescale the matrices linearly: DRNA'=alpha*DRNA and DADT'=(1-alpha)*DADT, solving for alpha in [0,1] by univariate convex minimization (e.g. R's optimize()) on a scalar objective such as fused-distance dispersion. Fuse by elementwise maximum, Djoint=max(DRNA', DADT'), per the L-infinity norm model, then feed Djoint into Louvain community detection (Seurat default) and t-SNE/UMAP visualization.

## Related tools

- **LinQ-View** (Toolkit implementing the sigmoid filter, linear alpha-rescaling, and L-infinity joint distance fusion for CITE-seq modalities) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (Provides the Louvain community-detection clustering applied downstream of the fused distance matrix (default in LinQ-View)) — https://satijalab.org/seurat/

## Examples

```
F <- function(x, n=10, k=0.5) x/(1+exp(-n*(x-k))); D_joint <- pmax(alpha*D_RNA, (1-alpha)*F(D_ADT))  # L-infinity fusion per LinQ-View
```

## Evaluation signals

- Sigmoid filter evaluated at x=k=0.5 exhibits the 0.5*k midpoint behavior; output is monotone and bounded for the ADT distance matrix
- alpha=1 boundary reduces the fused matrix to the pure RNA distances; alpha=0 reduces it to the pure (sigmoid-filtered) ADT distances
- Djoint is elementwise >= both rescaled modality matrices (L-infinity max property) on small synthetic symmetric matrices
- Downstream Louvain clustering on Djoint resolves modality-exclusive subsets (e.g. CD8- / CD8+ NK; naive / memory CD4+ T cells on Dataset 1) that unimodal RNA- or ADT-only clustering cannot distinguish
- Cluster purity scores computed per cluster in both transcriptome and surface-protein modalities (e.g. NK clusters 5 and 6: RNA purity 0.9553 and 0.9434, ADT purity 0.7822 and 0.8279)

## Limitations

- Pairwise distance computation scales quadratically with cell count: memory usage and complexity grow quadratically as dataset size increases, limiting time efficiency
- LinQ-View (including this fusion) failed on a massive dataset of 170K cells with 228 ADT features where Seurat 4 succeeded
- Best suited to large datasets with a limited number of ADT features (<50)

## Evidence

- [intro] LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] we introduced an L∞ norm model to calculate the joint distance: "we introduced an L∞ norm model to calculate the joint distance"
- [discussion] we introduced a sigmoid filter into the distance calculation of the ADT modality: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] calculating cell-cell pair-wise distances for each modality, joint distance calculation, cell population clustering, non-linear dimension reduction and visualization: "calculating cell-cell pair-wise distances for each modality, joint distance calculation, cell population clustering, non-linear dimension reduction and visualization"
- [readme] calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine the dissimilarities from each modality: "calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine the dissimilarities from each modality"
