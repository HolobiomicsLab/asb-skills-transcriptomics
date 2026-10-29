---
name: linf-norm-joint-distance-fusion
description: Use when analyzing CITE-seq data with RNA and surface-protein (ADT) modalities
  where modality fusion by L∞ norm of pairwise cell-cell distances is needed, especially
  with routine numbers of ADT features (<50).
when_to_use_negative:
- 'Massive CITE-seq datasets with many ADT features (e.g. 170K cells with 228 ADT
  features): pairwise distance computation grows quadratically with cell number, LinQ-View
  failed where Seurat 4 succeeded'
- 'Datasets with a large number of ADT features (e.g. ≥50): LinQ-View is best suited
  for large datasets with a limited number of ADT features (<50)'
- 'Analyses where batch effects between multiple runs are uncorrected: negative-control
  runs without Seurat 3 batch correction over-split cell groups (24 vs 21 groups on
  PBMC 1K + PBMC 10K)'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: LinQ-View
  role: Implements the per-modality pairwise distance calculation, sigmoid-filtered
    ADT distances, and L∞ norm joint distance fusion for CITE-seq integration
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat v3.9.9
  role: Performs pre-processing (QC, normalization, scaling, HVG, PCA) and default
    Louvain community-detection clustering on the fused distances; Seurat 3 batch
    correction for multi-dataset runs
  repo: https://satijalab.org/seurat/
- name: umap v0.2.6.0
  role: Non-linear dimension reduction of the joint distance representation for visualization
  repo: https://github.com/lmcinnes/umap
- name: ggplot2 v3.3.2
  role: Rendering comparison figures of modality contributions and cluster separations
  repo: https://dplyr.tidyverse.org/
- name: ROGUE
  role: Quantitative cluster purity assessment of populations identified from the
    fused distances
  repo: https://github.com/PaulingLiu/ROGUE
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:a48998c49f9993661f0d2cbbbf0589a588193ee2fd8ebc22030184800c7645ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R
  - ROGUE
  - Seurat
  - Seurat v3.9.9
  - ggplot2
  - ggplot2 v3.3.2
  - umap v0.2.6.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# linf-norm-joint-distance-fusion

## Summary

Fuse per-modality cell-cell pairwise distance matrices from CITE-seq RNA and ADT data into a single joint distance using the LinQ-View L∞ norm model, optionally applying a sigmoid filter to the ADT distances so no single surface protein dominates the integration.

## When to use

Use when analyzing CITE-seq data with RNA and surface-protein (ADT) modalities where modality fusion by L∞ norm of pairwise cell-cell distances is needed, especially with routine numbers of ADT features (<50).

## When NOT to use

- Massive CITE-seq datasets with many ADT features (e.g. 170K cells with 228 ADT features): pairwise distance computation grows quadratically with cell number, LinQ-View failed where Seurat 4 succeeded
- Datasets with a large number of ADT features (e.g. ≥50): LinQ-View is best suited for large datasets with a limited number of ADT features (<50)
- Analyses where batch effects between multiple runs are uncorrected: negative-control runs without Seurat 3 batch correction over-split cell groups (24 vs 21 groups on PBMC 1K + PBMC 10K)

## Inputs

- Pre-processed CITE-seq RNA expression matrix (normalized, scaled, PCA-reduced) from raw count matrices (e.g. GSE100866 CBMC, 10x pbmc_1k_protein_v3)
- Pre-processed ADT/surface-protein expression matrix (normalized, scaled) from the same cells
- Per-modality cell-cell pairwise distance matrices with shared cell barcodes

## Outputs

- Joint L∞-norm fused cell-cell distance matrix over all cells
- Cell cluster assignments from Seurat Louvain community detection
- UMAP embedding for visualization (e.g. ggplot2 v3.3.2 figures)
- Per-modality contribution percentages to inter-cluster distances (e.g. ADT 71.2%/RNA 28.8% unfiltered vs ADT 43.6%/RNA 56.4% sigmoid-filtered)

## How to apply

After Seurat-style pre-processing (cell QC excluding <200 or >2500 detected genes, mitochondrial soft threshold at the 95th percentile with a 10% sealing point, normalization, scaling, HVG identification, PCA), compute cell-cell pairwise distance matrices separately for the transcriptome and ADT modalities. Optionally apply a sigmoid filter to the ADT distance calculation to prevent variations in a single surface protein from dominating — in the CBMC (GSE100866) memory vs naive CD4+ T cell comparison this shifts the inter-subset distance share from ADT 71.2%/RNA 28.8% (unfiltered) to ADT 43.6%/RNA 56.4% (filtered). Fuse the modality-specific matrices into the joint distance via the L∞ norm model, then cluster with Seurat's default Louvain community detection and embed with UMAP (umap v0.2.6.0). Verify fused-matrix dimensions match the shared cell barcodes across modalities and that downstream cluster purity can be quantified with the ROGUE-based purity score.

## Related tools

- **LinQ-View** (Implements the per-modality pairwise distance calculation, sigmoid-filtered ADT distances, and L∞ norm joint distance fusion for CITE-seq integration) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat v3.9.9** (Performs pre-processing (QC, normalization, scaling, HVG, PCA) and default Louvain community-detection clustering on the fused distances; Seurat 3 batch correction for multi-dataset runs) — https://satijalab.org/seurat/
- **umap v0.2.6.0** (Non-linear dimension reduction of the joint distance representation for visualization) — https://github.com/lmcinnes/umap
- **ggplot2 v3.3.2** (Rendering comparison figures of modality contributions and cluster separations) — https://dplyr.tidyverse.org/
- **ROGUE** (Quantitative cluster purity assessment of populations identified from the fused distances) — https://github.com/PaulingLiu/ROGUE

## Evaluation signals

- Unfiltered run on CBMC (GSE100866) must yield ADT ≈71.2% / RNA ≈28.8% of the memory vs naive CD4+ T cell inter-subset distance, with the subsets not separated
- Sigmoid-filtered run must yield ADT ≈43.6% / RNA ≈56.4% with memory/naive CD4+ T cell subsets separated (Figure 2 match)
- Fused joint distance must be the L∞ (max-like) combination of per-modality distance matrices with identical cell dimensions/barcodes
- On PBMC 1K + PBMC 10K combined data, expect 24 cell groups without batch correction (batch-affected clusters 11, 12, 16, 22) and 21 groups after Seurat 3 batch correction
- ROGUE purity score should be computable per cluster in both transcriptome and surface protein modalities

## Limitations

- Memory usage and complexity of pairwise distance calculation grow quadratically as dataset size increases, limiting time efficiency (LinQ-View failed on a 170K-cell, 228-ADT-feature dataset where Seurat 4 succeeded)
- LinQ-View is best suited for large datasets with a limited number of ADT features (e.g., <50)
- The pre-defined parameter K for the ROGUE purity score is technology-dependent and its value is not specified in the text

## Evidence

- [intro] L∞ norm fusion of modality distance matrices: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] Sigmoid filter on ADT distances: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] L∞ norm joint distance model: "we introduced an L∞ norm model to calculate the joint distance"
- [readme] README design rationale: "The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality"
- [discussion] ADT feature count scope: "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
