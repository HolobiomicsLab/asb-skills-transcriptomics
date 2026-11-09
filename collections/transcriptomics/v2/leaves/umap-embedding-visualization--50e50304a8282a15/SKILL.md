---
name: umap-embedding-visualization
description: Use when single-cell scRNA-seq count matrices (e.g., GSE149689 PBMCs)
  have been normalized, PCA-reduced and batch-integrated, and you need a 2D embedding
  to assign cell-type labels and localize gene expression changes to specific immune
  subsets.
when_to_use_negative:
- Input is bulk RNA-seq or a processed differential-expression table (e.g., the counts/anova
  files from the COVID19-severity repo) rather than per-cell single-cell data
- Data have not been batch-corrected (Harmony) and PCA-embedded, since UMAP on uncorrected
  counts will mix severe/mild batch effects into the layout
- The goal is only to query gene expression statistics between severe and mild patients
  via the Streamlit box/strip plots, not to localize expression to cell subsets
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Seurat
  role: QC filtering, count normalization, PCA (top 20 PCs), clustering at resolution
    1, UMAP embedding, cell-type annotation and VlnPlot visualization
- name: Harmony
  role: Batch integration of the top-20-PC embedding prior to UMAP visualization
- name: UMAP
  role: 2D embedding of Harmony-integrated cells for visualization and marker-based
    label assignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:45378866010d961b9a2c9f6d73a2d74908f03b41b059ae0be19b3f6653d9607d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Seurat
  - UMAP
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# umap-embedding-visualization

## Summary

Embeds Harmony-integrated single-cell PBMC data in 2D with UMAP so cell-type clusters (e.g., CD14+ monocytes) can be annotated from canonical marker expression and gene-level dysregulation (MCEMP1, HLA-DRA, ETS1) can be visualized as violin plots per cluster.

## When to use

Use when single-cell scRNA-seq count matrices (e.g., GSE149689 PBMCs) have been normalized, PCA-reduced and batch-integrated, and you need a 2D embedding to assign cell-type labels and localize gene expression changes to specific immune subsets.

## When NOT to use

- Input is bulk RNA-seq or a processed differential-expression table (e.g., the counts/anova files from the COVID19-severity repo) rather than per-cell single-cell data
- Data have not been batch-corrected (Harmony) and PCA-embedded, since UMAP on uncorrected counts will mix severe/mild batch effects into the layout
- The goal is only to query gene expression statistics between severe and mild patients via the Streamlit box/strip plots, not to localize expression to cell subsets

## Inputs

- Raw single-cell RNA-seq count matrix from PBMCs (e.g., GEO accession GSE149689)
- Seurat-normalized expression matrix with top-20 PC embedding
- Harmony-integrated PC embedding of the 48,583 QC-passing cells

## Outputs

- 2D UMAP embedding of the integrated cell population
- Cluster assignments at resolution 1 with cell-type labels (CD14+ monocytes, proliferating cells, other immune lineages)
- Violin plots (Seurat VlnPlot) of MCEMP1, HLA-DRA, and ETS1 expression across annotated cell-type clusters

## How to apply

After applying the paper's quality-control thresholds to retain 48,583 cells, normalize counts in Seurat, run PCA keeping the top 20 principal components, and integrate batches with Harmony on the top-20-PC embedding, cluster at resolution 1 and embed the integrated cells in 2D with UMAP for visualization. Assign cell-type labels — including CD14+ monocytes and proliferating cells — from canonical marker expression on the UMAP. Rationale: batch correction followed by UMAP on the integrated embedding is needed because the severe/mild COVID-19 PBMC data come from multiple batches, and the 2D layout is what enables visually attributing heightened MCEMP1 and reduced HLA-DRA to the CD14+ monocyte cluster. Generate Seurat VlnPlot violin plots of MCEMP1, HLA-DRA, and ETS1 across the annotated clusters to complete the localization analysis.

## Related tools

- **Seurat** (QC filtering, count normalization, PCA (top 20 PCs), clustering at resolution 1, UMAP embedding, cell-type annotation and VlnPlot visualization)
- **Harmony** (Batch integration of the top-20-PC embedding prior to UMAP visualization)
- **UMAP** (2D embedding of Harmony-integrated cells for visualization and marker-based label assignment)

## Examples

```
pbmc <- RunUMAP(harmony_integrated, reduction = "harmony", dims = 1:20); DimPlot(pbmc, reduction = "umap", group.by = "celltype"); VlnPlot(pbmc, features = c("MCEMP1", "HLA-DRA", "ETS1"), group.by = "celltype")
```

## Evaluation signals

- QC retains exactly the reported 48,583 cells before normalization and PCA
- UMAP is computed on the Harmony-integrated top-20-PC embedding, not raw counts or uncorrected PCs
- Annotated UMAP shows a distinct CD14+ monocyte cluster and a proliferating-cell cluster identifiable from canonical markers
- VlnPlot reproduces the reported pattern: significantly heightened MCEMP1 and reduced HLA-DRA primarily in CD14+ monocytes, reduced HLA-DRA broadly across other lineages, and low ETS1 restricted to proliferating cells

## Limitations

- Dataset heterogeneity across the included studies was noted, so future studies are needed to understand heterogeneity in patients influencing MCEMP1 and HLA-DRA expression
- The datasets did not directly evaluate the effect of different SARS-CoV-2 variants on transcriptomic responses
- Only PBMC and BALF single-cell data were analyzed; the role of MCEMP1 in monocytes at the protein level remains uncharacterised

## Evidence

- [abstract] Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [discussion] we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
- [full_text] Embed the integrated cells in 2D with UMAP for visualization and assign cell-type labels (including CD14+ monocytes and proliferating cells) from canonical marker expression.: [evidence span withheld — anchor status: not_found]
- [intro] we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets: "we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
