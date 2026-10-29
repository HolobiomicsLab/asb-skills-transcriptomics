---
name: cd4-t-cell-subset-cluster-visualization
description: Use when analyzing CITE-seq data with a limited number of ADT features
  (e.g., <50) where a single surface protein modality dominates the joint distance,
  and you need to test whether applying the sigmoid filter to ADT distances separates
  RNA-defined subsets (e.g., memory vs naive CD4+ T cells).
when_to_use_negative:
- The dataset is massive (e.g., ~170K cells with 228 ADT features) or has a high number
  of ADT features — LinQ-View's quadratic pairwise distance computation failed there,
  and the paper recommends Seurat 4 instead
- The dataset has more than ~50 ADT features — the paper states LinQ-View is best
  suited to large datasets with a limited number of ADT features (<50)
- Only unimodal scRNA-seq data is available — the L∞ fusion and sigmoid filter are
  defined for the ADT distance calculation in CITE-seq data
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LinQ-View
  role: Computes per-modality cell-cell pairwise distances, applies the sigmoid filter
    to ADT distances, and fuses them with the L∞ norm joint model
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: Performs preprocessing (QC, normalization, scaling, HVG, PCA) and default
    Louvain community detection clustering on the fused joint distances
  repo: https://satijalab.org/seurat/
- name: umap
  role: Non-linear dimension reduction and embedding of the fused joint distances
    for visualizing CD4+ subset separation
  repo: https://github.com/lmcinnes/umap
- name: ggplot2
  role: Renders the Figure 2 before/after comparison of unfiltered vs filtered cluster
    embeddings and modality contribution percentages
  repo: https://ggplot2.tidyverse.org/
- name: R
  role: Runtime environment (paper used R v3.6.3) in which the LinQ-View pipeline,
    clustering, embedding, and plotting are executed
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:7a711d96f90bdedd5fd47b15d7df584b0c2bf47d8479cfa3d90544f1b5e4f6ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R
  - Seurat
  - ggplot2
  - umap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cd4-t-cell-subset-cluster-visualization

## Summary

Reproduces the LinQ-View sigmoid-filter effect on CITE-seq clustering by computing per-modality cell-cell distances, fusing them with an L∞ norm with and without a sigmoid filter on the ADT distances, and visualizing how memory vs naive CD4+ T cell subsets separate and how the RNA/ADT contribution to their inter-subset distance shifts (ADT 71.2%→43.6%, RNA 28.8%→56.4%).

## When to use

Use when analyzing CITE-seq data with a limited number of ADT features (e.g., <50) where a single surface protein modality dominates the joint distance, and you need to test whether applying the sigmoid filter to ADT distances separates RNA-defined subsets (e.g., memory vs naive CD4+ T cells).

## When NOT to use

- The dataset is massive (e.g., ~170K cells with 228 ADT features) or has a high number of ADT features — LinQ-View's quadratic pairwise distance computation failed there, and the paper recommends Seurat 4 instead
- The dataset has more than ~50 ADT features — the paper states LinQ-View is best suited to large datasets with a limited number of ADT features (<50)
- Only unimodal scRNA-seq data is available — the L∞ fusion and sigmoid filter are defined for the ADT distance calculation in CITE-seq data

## Inputs

- RNA and ADT count matrices from the 8,617-cell CBMC CITE-seq dataset (GEO accession GSE100866)
- Seurat object with preprocessed RNA and ADT modalities (QC'd, normalized, scaled, HVGs identified, PCA computed)
- Per-modality cell-cell pairwise distance matrices (RNA and ADT)

## Outputs

- L∞-norm fused joint distance matrix (two variants: with and without sigmoid filter on ADT distances)
- Louvain cluster assignments and UMAP embedding for each joint-distance variant
- ADT vs RNA contribution percentages to the memory vs naive CD4+ T cell inter-subset distance (unfiltered: 71.2%/28.8%; filtered: 43.6%/56.4%)
- Figure 2 comparison plot rendered with ggplot2 showing subset separation with vs without the sigmoid filter

## How to apply

Download the 8,617-cell CBMC CITE-seq dataset (GSE100866) and load RNA and ADT count matrices into R. Preprocess both modalities per LinQ-View defaults: cell QC excluding cells with <200 or >2500 detected genes, a 95th-percentile mitochondrial soft threshold with 10% sealing, normalization, scaling, HVG identification, and PCA. Compute per-modality cell-cell pairwise distance matrices and fuse them with the L∞ norm using the LinQ-View joint model WITHOUT the sigmoid filter; cluster with Seurat Louvain community detection, embed with UMAP, and quantify the ADT vs RNA share of the inter-subset distance between memory and naive CD4+ T cells. Repeat the joint distance calculation with the sigmoid filter applied to the ADT modality distances, re-cluster, re-embed, and recompute the contribution percentages. Render the before/after comparison (Figure 2) with ggplot2, validating that the unfiltered run yields ADT ≈71.2% / RNA ≈28.8% with unseparated subsets and the filtered run yields ADT ≈43.6% / RNA ≈56.4% with separated subsets.

## Related tools

- **LinQ-View** (Computes per-modality cell-cell pairwise distances, applies the sigmoid filter to ADT distances, and fuses them with the L∞ norm joint model) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (Performs preprocessing (QC, normalization, scaling, HVG, PCA) and default Louvain community detection clustering on the fused joint distances) — https://satijalab.org/seurat/
- **umap** (Non-linear dimension reduction and embedding of the fused joint distances for visualizing CD4+ subset separation) — https://github.com/lmcinnes/umap
- **ggplot2** (Renders the Figure 2 before/after comparison of unfiltered vs filtered cluster embeddings and modality contribution percentages) — https://ggplot2.tidyverse.org/
- **R** (Runtime environment (paper used R v3.6.3) in which the LinQ-View pipeline, clustering, embedding, and plotting are executed) — https://www.r-project.org/

## Examples

```
devtools::install_github("WilsonImmunologyLab/LinQView")
```

## Evaluation signals

- Unfiltered run must yield ADT ≈71.2% / RNA ≈28.8% of the inter-subset distance between memory and naive CD4+ T cells, with the two subsets NOT separated in the UMAP embedding
- Filtered run must yield ADT ≈43.6% / RNA ≈56.4%, with memory and naive CD4+ subsets separated, matching Figure 2
- QC invariant: all cells with fewer than 200 or more than 2500 detected genes are excluded before distance computation
- Cluster assignments must come from the default Louvain community detection implemented in Seurat, and embeddings from UMAP (or t-SNE), per LinQ-View defaults
- The 8,617-cell CBMC dataset must resolve to the expected structure with the sigmoid filter shifting the modality balance toward RNA dominance

## Limitations

- LinQ-View's memory usage and pairwise distance complexity grow quadratically with cell number, limiting time efficiency on large datasets
- LinQ-View failed on a massive dataset (170K cells with 228 ADT features) where Seurat 4 succeeded
- Key clustering-comparison results (e.g., Figure S4A/S5/S7 cluster counts vs MOFA2 and Seurat 4) are only in supplemental figures and may not be reproducible from the main text alone
- The pre-defined parameter K for the ROGUE purity score is technology-dependent but its value is not specified in the text

## Evidence

- [discussion] Sigmoid filter shifts modality balance: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] QC thresholds for cells: "all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded"
- [discussion] Default clustering method: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [discussion] ADT feature count scope limit: "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
- [readme] L∞ fusion rationale: "combining dissimilarity representations may help emphasize different information from each modality to maximally capture and infer cell heterogeneity"
