---
name: cite-seq-multimodal-clustering
description: Use when analyzing CITE-seq data with both transcriptome and surface-protein
  (ADT) modalities and the goal is joint cell-population clustering plus quantitative
  purity scoring in both modalities.
when_to_use_negative:
- Massive CITE-seq datasets with very large cell counts and many ADT features (e.g.,
  170K cells with 228 ADT features) — LinQ-View failed there while Seurat 4 succeeded;
  LinQ-View is best for large datasets with a limited number of ADT features (<50)
- Unimodal scRNA-seq data without a surface-protein modality — the L∞ norm fusion
  of RNA and ADT distance matrices is the core mechanism and cannot be applied
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View
  role: 'Primary toolkit: computes per-modality dissimilarity representations, fuses
    them with an L∞ norm joint distance, performs Louvain clustering, and computes
    purity scores'
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: LinQ-View v0.99
  role: Versioned release of LinQ-View used for the multimodal clustering and purity
    analysis
  repo: https://wilsonimmunologylab.github.io/LinQView
- name: Seurat
  role: Provides the default Louvain community detection clustering used by LinQ-View
    for cell population identification
  repo: https://satijalab.org/seurat/
- name: R
  role: Implementation environment for running the LinQ-View pipeline (R v3.6.3 used
    in the paper)
  repo: https://www.r-project.org/
- name: umap
  role: Non-linear dimension reduction for visualizing joint clusters
  repo: https://github.com/lmcinnes/umap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:61d47da19ce4b168b7592e7ec6a17c0f81820bb64ceb10c62320f7e97a312ca1
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
  - umap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cite-seq-multimodal-clustering

## Summary

Joint clustering of CITE-seq data with LinQ-View, which fuses per-modality cell-cell pairwise distance matrices (transcriptome and surface protein/ADT) using an L∞ norm model and scores cluster purity in both modalities. It resolves modality-exclusive cell subsets (e.g., CD8−/CD8+ NK cells and naive/memory CD4+ T cells) that unimodal RNA- or ADT-only clustering cannot distinguish.

## When to use

Use when analyzing CITE-seq data with both transcriptome and surface-protein (ADT) modalities and the goal is joint cell-population clustering plus quantitative purity scoring in both modalities.

## When NOT to use

- Massive CITE-seq datasets with very large cell counts and many ADT features (e.g., 170K cells with 228 ADT features) — LinQ-View failed there while Seurat 4 succeeded; LinQ-View is best for large datasets with a limited number of ADT features (<50)
- Unimodal scRNA-seq data without a surface-protein modality — the L∞ norm fusion of RNA and ADT distance matrices is the core mechanism and cannot be applied

## Inputs

- Processed CITE-seq Seurat object with transcriptome and surface-protein (ADT) modalities (e.g., GSE100866 CBMC dataset, 8,617 cells)
- Per-modality cell-cell pairwise distance matrices (computed from RNA and ADT modalities)

## Outputs

- Joint cluster assignments per cell (plus unimodal RNA-only and ADT-only clusterings for comparison)
- Quantitative purity score for every cluster in both transcriptome and surface-protein modalities (e.g., figure4_purity_scores.csv)
- t-SNE/UMAP dimension-reduction embeddings for visualization

## How to apply

Load the processed CITE-seq object (e.g., GSE100866, 8,617 cord blood mononuclear cells) in R with both modalities. Run LinQ-View's pipeline: cell QC (exclude cells with <200 or >2500 detected genes; soft-threshold mitochondrial proportion at the 95th percentile with a 10% sealing point), normalization and scaling, then compute cell-cell pairwise distances per modality, applying a sigmoid filter to the ADT distance calculation. Fuse the per-modality distance matrices with the L∞ norm joint-distance model and cluster with Louvain community detection (the Seurat-implemented default in LinQ-View), followed by non-linear dimension reduction (t-SNE/UMAP defaults). Compute LinQ-View's quantitative purity score for every cluster in both the transcriptome and surface protein modalities, then verify that modality-exclusive subsets (e.g., the two NK clusters 5 and 6 with RNA purity 0.9553/0.9434 and ADT purity 0.7822/0.8279 on Dataset 1) are resolved jointly and compare against RNA-only and ADT-only Louvain clusterings as unimodal controls.

## Related tools

- **LinQ-View** (Primary toolkit: computes per-modality dissimilarity representations, fuses them with an L∞ norm joint distance, performs Louvain clustering, and computes purity scores) — https://github.com/WilsonImmunologyLab/LinQView
- **LinQ-View v0.99** (Versioned release of LinQ-View used for the multimodal clustering and purity analysis) — https://wilsonimmunologylab.github.io/LinQView
- **Seurat** (Provides the default Louvain community detection clustering used by LinQ-View for cell population identification) — https://satijalab.org/seurat/
- **R** (Implementation environment for running the LinQ-View pipeline (R v3.6.3 used in the paper)) — https://www.r-project.org/
- **umap** (Non-linear dimension reduction for visualizing joint clusters) — https://github.com/lmcinnes/umap

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("WilsonImmunologyLab/LinQView")
```

## Evaluation signals

- LinQ-View joint clustering resolves all four modality-exclusive subsets (CD8− and CD8+ NK cells; naive and memory CD4+ T cells) on Dataset 1 that RNA-only and ADT-only Louvain clusterings cannot distinguish
- Purity scores fall in expected ranges: the two NK clusters (5 and 6) achieve RNA purity scores of 0.9553 and 0.9434 and ADT purity scores of 0.7822 and 0.8279
- Every cluster in each of the three clusterings (RNA-only, ADT-only, joint) receives a defined purity score in both transcriptome and surface-protein modalities
- Cluster counts match the paper's comparisons (e.g., LinQ-View, MOFA2, and Seurat 4 identified 7, 7, and 8 cell clusters on dataset 2)

## Limitations

- LinQ-View's pairwise distance computation grows quadratically in memory and time with dataset size, limiting time efficiency for very large datasets
- LinQ-View failed on a massive dataset (170K cells with 228 ADT features) where Seurat 4 succeeded
- Best suited to large datasets with a limited number of ADT features (<50); Seurat 4 is recommended for massive CITE-seq datasets with many ADT features

## Evidence

- [intro] L∞ norm fusion: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [intro] purity score metric: "we developed a quantitative metric (purity score) to assess the purity of single-cell populations in both the transcriptome modality and the surface protein modality of CITE-seq data"
- [discussion] sigmoid filter and QC thresholds: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] default clustering method: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [discussion] Dataset 1 identity: "a dataset of 8,617 cord blood mononuclear cells (CBMCs), produced with CITE-seq (Stoeckius et al., 2017). NCBI GEO accession number is GSE100866"
- [readme] LinQ-View core idea from README: "to calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine the dissimilarities from each modality"
