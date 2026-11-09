---
name: adt-distance-sigmoid-filtering
description: Use when analyzing CITE-seq data with a limited number of ADT features
  (<50) where the ADT modality disproportionately dominates the joint cell-cell distance
  and masks transcriptomic differences between subsets.
when_to_use_negative:
- Datasets with a large number of ADT features (e.g., >50, or 228 ADT features on
  170K cells) — LinQ-View is best suited for large datasets with a limited number
  of ADT features (<50) and failed on massive datasets where Seurat 4 succeeded
- Analysis goals that are unimodal (RNA-only or ADT-only), where per-modality distances
  suffice and joint L∞ fusion is unnecessary
- Datasets already processed into a joint embedding or clustering, where the per-modality
  pairwise distance matrices required for filtering are unavailable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View
  role: Toolkit implementing the sigmoid-filtered ADT distance calculation, linear
    α rescaling, and L∞ joint distance fusion
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: Performs the Louvain community detection clustering and dimension reduction
    on the fused joint distance matrix
  repo: https://satijalab.org/seurat/
- name: R
  role: Implementation language for the sigmoid filter F(x)=x/(1+e^{-n(x-k)}) and
    distance matrix computation
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:53de64da9fd30fe8bfa0f5d957c0a42cc92022dbc665b08ec703a2cf3aab812a
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
yes_rate: 1.0
n_claims: 11
---

# adt-distance-sigmoid-filtering

## Summary

Apply LinQ-View's elementwise sigmoid filter F(x)=x/(1+e^{-n(x-k)}) with n=10 and k=0.5 to the ADT cell-cell Euclidean distance matrix before L∞-norm fusion with the RNA distance matrix, preventing a single surface marker from dominating the joint distance in CITE-seq data.

## When to use

Use when analyzing CITE-seq data with a limited number of ADT features (<50) where the ADT modality disproportionately dominates the joint cell-cell distance and masks transcriptomic differences between subsets.

## When NOT to use

- Datasets with a large number of ADT features (e.g., >50, or 228 ADT features on 170K cells) — LinQ-View is best suited for large datasets with a limited number of ADT features (<50) and failed on massive datasets where Seurat 4 succeeded
- Analysis goals that are unimodal (RNA-only or ADT-only), where per-modality distances suffice and joint L∞ fusion is unnecessary
- Datasets already processed into a joint embedding or clustering, where the per-modality pairwise distance matrices required for filtering are unavailable

## Inputs

- Pre-processed ADT modality cell-cell pairwise Euclidean distance matrix (from CITE-seq surface protein data, e.g. GSE100866 CBMCs)
- RNA modality cell-cell pairwise distance matrix for subsequent L∞ fusion
- Sigmoid filter parameters n=10 and k=0.5

## Outputs

- Sigmoid-filtered ADT distance matrix (elementwise F(x) applied)
- Joint distance matrix Djoint = max(DRNA', DADT') used for Louvain clustering and UMAP/t-SNE embedding
- Per-cluster purity scores and cluster assignments in both transcriptome and surface protein modalities

## How to apply

First compute the per-modality cell-cell pairwise Euclidean distance matrices from the pre-processed RNA and ADT modalities (after QC excluding <200 or >2500 detected genes, normalization, scaling, HVG identification, and PCA). Apply the sigmoid filter elementwise to the ADT distance matrix as F(x)=x/(1+e^{-n(x-k)}) with the paper's parameters n=10 and k=0.5; the filter suppresses large ADT distances contributed by single surface markers. Fuse the filtered ADT distances with the RNA distances via the linear rescaling DRNA'=αDRNA, DADT'=(1-α)DADT (α solved by univariate convex minimization over [0,1]) and the L∞ norm Djoint=max(DRNA', DADT'). Then cluster the joint distance matrix with Seurat's Louvain community detection and embed with UMAP/t-SNE. Rationale: on Dataset 1 (GSE100866 CBMCs), the filter shifted the memory vs naive CD4+ T cell inter-subset distance from ADT-dominated (71.2%) to RNA-influenced (RNA 56.4%), enabling subset separation.

## Related tools

- **LinQ-View** (Toolkit implementing the sigmoid-filtered ADT distance calculation, linear α rescaling, and L∞ joint distance fusion) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (Performs the Louvain community detection clustering and dimension reduction on the fused joint distance matrix) — https://satijalab.org/seurat/
- **R** (Implementation language for the sigmoid filter F(x)=x/(1+e^{-n(x-k)}) and distance matrix computation) — https://www.r-project.org/

## Examples

```
sigmoid_filter <- function(x) x/(1+exp(-10*(x-0.5))); D_ADT_filtered <- sigmoid_filter(D_ADT)
```

## Evaluation signals

- Unit test: the sigmoid output at x=k=0.5 with n=10 equals 0.25 (the 0.5·k midpoint behavior), and boundary cases α=0/α=1 reduce to pure-ADT and pure-RNA distances
- Invariant: Djoint = max(DRNA', DADT') must be elementwise ≥ both rescaled modality matrices
- On Dataset 1 (GSE100866), the unfiltered run should give ADT ≈71.2% / RNA ≈28.8% of the memory vs naive CD4+ T cell inter-subset distance, while the filtered run should give ADT ≈43.6% / RNA ≈56.4% with the subsets separated
- Filtered run must resolve the four modality-exclusive subsets (CD8− and CD8+ NK cells; naive and memory CD4+ T cells) matching Figure 4, with NK cluster purity scores near 0.9553/0.9434 (RNA) and 0.7822/0.8279 (ADT)
- Output CSV (e.g. figure4_purity_scores.csv) exists and contains purity scores for all three clusterings (RNA-only, ADT-only, joint) with populated cluster assignments

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time and memory efficiency on large datasets
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features, whereas Seurat 4 succeeded
- LinQ-View is best suited for datasets with a limited number of ADT features (e.g., <50); with many surface markers, the modality weighting may be unreliable

## Evidence

- [discussion] Sigmoid filter applied to ADT distance calculation: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] Joint distance calculation using L∞ norm model: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [abstract] LinQ-View prevents single surface marker domination: "LinQ-View efficiently generates accurate cell clusters, especially in CITE-seq data with routine numbers of surface protein features"
- [readme] LinQ-View core idea of combining per-modality dissimilarities: "calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine the dissimilarities from each modality"
