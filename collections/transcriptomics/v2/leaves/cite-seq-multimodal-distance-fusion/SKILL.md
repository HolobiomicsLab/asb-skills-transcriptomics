---
name: cite-seq-multimodal-distance-fusion
description: Use when analyzing CITE-seq data with a limited number of ADT features
  (e.g. <50) where unimodal or naive joint analysis fails to separate subsets (e.g.
  memory vs naive CD4+ T cells), or when distinguishing antigen-specific B cell populations
when_to_use_negative:
- Massive CITE-seq datasets with many ADT features (e.g. 170K cells with 228 ADT features)
  — LinQ-View failed on such data while Seurat 4 succeeded
- Large datasets with >50 ADT features — the paper states LinQ-View is the best option
  only for datasets with a limited number of ADT features
- Unimodal scRNA-seq data with no surface-protein modality — the L∞ fusion of RNA
  and ADT distance matrices is inapplicable
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View
  role: 'core toolkit: computes per-modality pairwise distances, applies the sigmoid
    filter to ADT distances, and fuses them with the L∞ norm joint model'
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: preprocessing (QC, normalization, scaling, PCA) and default Louvain community
    detection clustering of the fused distances
  repo: https://satijalab.org/seurat/
- name: ROGUE
  role: quantitative cluster purity assessment of the resulting cell populations in
    both modalities
  repo: https://github.com/PaulingLiu/ROGUE
- name: ggplot2
  role: rendering UMAP/cluster figures (e.g. Figure 2 sigmoid-filter comparison, Figure
    7 B cell cluster plots)
  repo: https://ggplot2.tidyverse.org/
- name: umap
  role: non-linear dimension reduction of the fused distance matrix for visualization
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:f85e2583f194c0112e22bf2026d44ddbe0c84fce53a397bf698a436131a12afc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R
  - ROGUE
  - Seurat
  - dplyr
  - ggplot2
  - umap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# CITE-seq multimodal distance fusion (L∞ norm with sigmoid-filtered ADT distances)

## Summary

Fuse per-modality cell-cell pairwise distance matrices from CITE-seq transcriptome and surface-protein (ADT) data using LinQ-View's L∞ norm joint model, applying a sigmoid filter to ADT distances so a single surface marker cannot dominate, then cluster with Seurat's Louvain community detection and embed with UMAP/t-SNE.

## When to use

Use when analyzing CITE-seq data with a limited number of ADT features (e.g. <50) where unimodal or naive joint analysis fails to separate subsets (e.g. memory vs naive CD4+ T cells), or when distinguishing antigen-specific B cell populations

## When NOT to use

- Massive CITE-seq datasets with many ADT features (e.g. 170K cells with 228 ADT features) — LinQ-View failed on such data while Seurat 4 succeeded
- Large datasets with >50 ADT features — the paper states LinQ-View is the best option only for datasets with a limited number of ADT features
- Unimodal scRNA-seq data with no surface-protein modality — the L∞ fusion of RNA and ADT distance matrices is inapplicable

## Inputs

- CITE-seq RNA count matrix (e.g. GSE100866 CBMC 8,617 cells; 10x pbmc_1k_protein_v3; Mendeley 3jdywv5jrv/3 peripheral blood CD19+ B cells from 10 COVID-19 patients)
- Matching ADT (surface protein) count matrix for the same cells
- Modality-specific preprocessing parameters (QC cutoffs, mitochondrial soft threshold)

## Outputs

- Fused joint cell-cell distance matrix combining RNA and ADT modalities
- Louvain cluster assignments per cell (e.g. LinQ-View clusters)
- UMAP/t-SNE embeddings and per-modality distance-contribution percentages
- Cluster assignment table (e.g. linqview_dataset6_bcell_clusters.csv with cluster, UMAP, sample fields)

## How to apply

Pre-process both modalities per LinQ-View defaults: cell QC excluding cells with <200 or >2500 detected genes, apply a mitochondrial soft threshold set to the 95th percentile of the dataset distribution with a 10% sealing point as maximum, then normalize, scale, identify HVGs, and run PCA for both RNA and ADT matrices. Compute cell-cell pairwise distances for each modality separately and fuse them with the L∞ norm joint model; apply the sigmoid filter to the ADT modality distance calculation to prevent a single surface marker from dominating (this shifts the memory vs naive CD4+ T cell distance contribution from ADT 71.2%/RNA 28.8% to ADT 43.6%/RNA 56.4%). Cluster the fused distances with the default Louvain community detection algorithm implemented in Seurat and embed with UMAP (and optionally t-SNE). Optionally assess cluster purity with the ROGUE statistic. In the SARS-CoV-2 B cell application (Dataset 6, Mendeley 3jdywv5jrv/3), spike-specific cells enrich in memory-like LinQ-View clusters 6, 7, and 9 with increased somatic hypermutation and IgA/G class switching, while naive-like RNA clusters 0/1 partition into LinQ-View clusters 0, 3, 4, 5.

## Related tools

- **LinQ-View** (core toolkit: computes per-modality pairwise distances, applies the sigmoid filter to ADT distances, and fuses them with the L∞ norm joint model) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (preprocessing (QC, normalization, scaling, PCA) and default Louvain community detection clustering of the fused distances) — https://satijalab.org/seurat/
- **ROGUE** (quantitative cluster purity assessment of the resulting cell populations in both modalities) — https://github.com/PaulingLiu/ROGUE
- **ggplot2** (rendering UMAP/cluster figures (e.g. Figure 2 sigmoid-filter comparison, Figure 7 B cell cluster plots)) — https://ggplot2.tidyverse.org/
- **umap** (non-linear dimension reduction of the fused distance matrix for visualization)

## Examples

```
devtools::install_github("WilsonImmunologyLab/LinQView")
```

## Evaluation signals

- Without the sigmoid filter, ADT contributes ≈71.2% and RNA ≈28.8% of the memory vs naive CD4+ T cell inter-subset distance, and the subsets are not separated
- With the sigmoid filter, contributions shift to ADT ≈43.6% / RNA ≈56.4% and memory/naive CD4+ T cell subsets are separated (Figure 2)
- QC removes all cells with <200 or >2500 detected genes and applies the mitochondrial soft threshold at the 95th percentile with a 10% sealing point
- On Dataset 6, naive-like RNA clusters 0/1 split into LinQ-View clusters 0, 3, 4, 5, and spike-specific B cells enrich in memory-like clusters 6, 7, 9 with increased somatic hypermutation and IgA/G class switching (Figure 7)
- Output CSV exists with cluster, UMAP, and sample fields populated for all cells

## Limitations

- Pairwise distance computation scales quadratically with cell number, limiting time efficiency as dataset size grows
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features where Seurat 4 succeeded
- Best suited to datasets with a limited number of ADT features (<50); key clustering-comparison results appear only in supplemental figures (S4, S5, S7)
- The pre-defined parameter K for the ROGUE score is technology-dependent but its value is not specified in the text

## Evidence

- [intro] L∞ fusion mechanism: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] sigmoid filter: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [abstract] performance claim: "we demonstrate that LinQ-View efficiently generates accurate cell clusters, especially in CITE-seq data with routine numbers of surface protein features"
- [readme] README workflow rationale: "calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine the dissimilarities from each modality"
- [discussion] default clustering: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
