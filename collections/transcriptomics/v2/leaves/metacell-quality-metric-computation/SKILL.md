---
name: metacell-quality-metric-computation
description: Use when a constructed metacell partition must be quantitatively assessed
  for quality, especially when no cell-type annotations are available and a latent-space-independent
  metric (INV) is needed.
when_to_use_negative:
- The goal is to compare metacells against an annotated atlas or project metacells
  onto a single-cell UMAP — use mc_projection() instead of the QC metric functions.
- Cell-type annotations are unavailable and a supervised metric like purity is requested
  — purity requires an annotation vector; only INV is annotation-free.
- The task is interactive exploration of an already-built metacell model rather than
  metric computation — MCView shiny-app exploration is the wrong tool here.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetacellAnalysisToolkit
  role: Provides mc_purity(), mc_compactness(), mc_separation(), mc_INV(), get_diffusion_comp(),
    and qc_boxplot() for metric computation and visualization
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: Preprocesses the single-cell object (NormalizeData LogNormalize, FindVariableFeatures
    2500, ScaleData, RunPCA npcs=50, RunUMAP dims 1:50) to enable diffusion-map computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:7ff617acd771684a81e7d841f384808092bbd2aacd10185166b2e8e993de44ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  - Seurat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metacell-quality-metric-computation

## Summary

Compute four per-metacell quality metrics — purity, compactness, separation, and inner normalized variance (INV) — using MetacellAnalysisToolkit functions on packaged metacell/single-cell objects, and visualize them with qc_boxplot(). The key insight is that INV is the only metric requiring no cell-type annotation or latent-space embedding.

## When to use

Use when a constructed metacell partition must be quantitatively assessed for quality, especially when no cell-type annotations are available and a latent-space-independent metric (INV) is needed.

## When NOT to use

- The goal is to compare metacells against an annotated atlas or project metacells onto a single-cell UMAP — use mc_projection() instead of the QC metric functions.
- Cell-type annotations are unavailable and a supervised metric like purity is requested — purity requires an annotation vector; only INV is annotation-free.
- The task is interactive exploration of an already-built metacell model rather than metric computation — MCView shiny-app exploration is the wrong tool here.

## Inputs

- Metacell object with cell-to-metacell membership annotations (e.g. CD34_mc from MetacellAnalysisToolkit)
- Single-cell Seurat object with cell-type annotation (e.g. CD34_sc) for purity computation
- Membership data frame mapping single cells to metacells
- Low-dimensional embedding (diffusion-map components from PCA, dims 1:30) for compactness and separation

## Outputs

- Per-metacell quality metric columns (purity, compactness, separation, INV) written to metacell_qc_metrics.csv
- QC boxplots per metric, overall and split by cell type (qc_boxplot output)

## How to apply

Load the packaged example datasets CD34_mc (metacells) and CD34_sc (single cells) from MetacellAnalysisToolkit. Compute purity with mc_purity() using the single-cell celltype annotation as the most-abundant-category proportion per metacell. Preprocess the single-cell object with Seurat (NormalizeData with LogNormalize, FindVariableFeatures nfeatures=2500, ScaleData, RunPCA npcs=50, RunUMAP on dims 1:50 with n.neighbors=15, min.dist=0.5), then compute diffusion-map components from the PCA with get_diffusion_comp(dims=1:30) so compactness (variance of diffusion components within the metacell) and separation (distance to the closest metacell) can be calculated via mc_compactness() and mc_separation(). Compute the annotation-free, latent-space-independent INV with mc_INV(group.label='membership'), defined as the mean-normalized variance of gene expression within each metacell. Render QC boxplots with qc_boxplot(), overall and split.by='celltype', and write the four per-metacell metric columns to metacell_qc_metrics.csv.

## Related tools

- **MetacellAnalysisToolkit** (Provides mc_purity(), mc_compactness(), mc_separation(), mc_INV(), get_diffusion_comp(), and qc_boxplot() for metric computation and visualization) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Preprocesses the single-cell object (NormalizeData LogNormalize, FindVariableFeatures 2500, ScaleData, RunPCA npcs=50, RunUMAP dims 1:50) to enable diffusion-map computation)

## Examples

```
library(MetacellAnalysisToolkit); mc_data <- MetacellAnalysisToolkit::CD34_mc; mc_data$INV <- mc_INV(cell.membership = membership_df, sc.obj = sc_data, group.label = "membership"); qc_boxplot(mc.obj = mc_data, qc.metrics = "purity")
```

## Evaluation signals

- All four metric columns (purity, compactness, separation, INV) are populated for every metacell in CD34_mc with no missing values
- Boxplots are generated for each of the four metrics, both overall and split.by='celltype'
- INV is computed without any low-dimensional embedding, whereas purity, compactness, and separation each depend on annotations or the diffusion-map latent space
- Purity values lie in [0,1] as the proportion of the most abundant cell type within each metacell
- The CSV metacell_qc_metrics.csv contains exactly one row per metacell with the four named metric columns

## Limitations

- Purity requires prior cell-type annotation of single cells; compactness and separation depend on the diffusion-map latent space, so their values are not comparable across embeddings
- Only INV is independent of the latent space used for metacell construction, so metric rankings may disagree and INV should anchor annotation-free QC
- Diffusion-map components are computed from the first 30 PCA dims of the Seurat preprocessing; changes to normalization or variable-feature selection (nfeatures=2500) will shift compactness/separation values

## Evidence

- [methods] Inner normalized variance (INV) is the only metacell quality metric that requires no prior knowledge (such as cell-type annotation) and is independent of the latent space used to build metacells.: "The **inner normalized variance (INV)** of a metacell is the mean-normalized variance of gene expression within the metacell"
- [methods] Compactness and separation are defined on the diffusion-map latent space.: "The **compactness** of a metacell is the variance of the components within the metacell."
- [methods] Separation is the distance to the closest metacell in the embedding.: "The **separation** of a metacell is the distance to the closest metacell"
- [methods] Purity is the proportion of the most abundant cell type in the metacell.: "the **purity** of a metacell is the proportion of the most abundant cell type within the metacell"
- [methods] Purity is computed via mc_purity with the single-cell annotation.: "mc_data$purity <- mc_purity(membership = mc_data@misc$cell_membership$membership, annotation = sc_data$celltype)"
- [methods] QC metrics are visualized with qc_boxplot.: "qc_boxplot(mc.obj = mc_data, qc.metrics = "purity")"
- [abstract] The toolkit is a fully integrated pipeline for building, visualizing and evaluating metacells.: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
- [readme] The toolkit pairs construction tools with an R package for metacell quality control and visualization.: "Toolkit for metacell analysis. It consists of the `MATK` command line tool to easily identify metacells with either [SEACells](https://github.com/dpeerlab/SEACells),"
