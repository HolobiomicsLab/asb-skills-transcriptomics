---
name: seurat-pca-clustering-workflow
description: Use when raw single-cell RNA-seq count matrices from PBMC (e.g. GEO accessions
  like GSE149689) must be clustered and cell-type annotated to determine which immune
  subsets show differential expression of genes such as MCEMP1 and HLA-DRA.
when_to_use_negative:
- Input is already a processed, annotated single-cell object or a bulk RNA-seq count
  table (the systematic-review datasets like GSE155454/GSE157103 are analysed as bulk
  differential expression, not clustered).
- Only two-condition bulk comparison of severe vs mild COVID-19 gene expression is
  needed — use the Streamlit database query workflow instead of single-cell clustering.
- Raw count matrix lacks batch metadata, making Harmony integration on the PC embedding
  inapplicable.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: QC filtering, count normalization, PCA (top 20 PCs), clustering (resolution
    1), UMAP embedding, and VlnPlot generation of MCEMP1/HLA-DRA/ETS1 across cell
    types
- name: Harmony
  role: Batch integration of Seurat clusters on the top-20-PC embedding prior to clustering
- name: UMAP
  role: 2D embedding of integrated cells for visualization and cell-type annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:360965d516a1258cc98e8624b1528e9e1867eef94220f6bd807d498d6ae647c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Seurat
  - UMAP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# seurat-pca-clustering-workflow

## Summary

A Seurat-based single-cell PBMC pipeline (QC, normalization, top-20-PC PCA, Harmony batch integration, resolution-1 clustering, UMAP embedding, and marker-based cell-type annotation with VlnPlot readouts) used to localise MCEMP1 and HLA-DRA dysregulation to CD14+ monocytes in severe COVID-19.

## When to use

Use when raw single-cell RNA-seq count matrices from PBMC (e.g. GEO accessions like GSE149689) must be clustered and cell-type annotated to determine which immune subsets show differential expression of genes such as MCEMP1 and HLA-DRA.

## When NOT to use

- Input is already a processed, annotated single-cell object or a bulk RNA-seq count table (the systematic-review datasets like GSE155454/GSE157103 are analysed as bulk differential expression, not clustered).
- Only two-condition bulk comparison of severe vs mild COVID-19 gene expression is needed — use the Streamlit database query workflow instead of single-cell clustering.
- Raw count matrix lacks batch metadata, making Harmony integration on the PC embedding inapplicable.

## Inputs

- Raw single-cell RNA-seq count matrix from PBMCs (e.g. GEO GSE149689)
- Sample/batch metadata for Harmony integration
- QC thresholds from the paper's methods section

## Outputs

- Seurat object with Harmony-integrated top-20-PC embedding
- Cluster assignments at resolution 1 and UMAP 2D embedding
- Cell-type annotations (CD14+ monocytes, proliferating cells, etc.)
- Violin plots of MCEMP1, HLA-DRA and ETS1 expression per annotated cluster

## How to apply

Download the raw PBMC count matrix from GEO, apply the paper's QC thresholds in Seurat to retain ~48,583 cells, then normalize counts and run PCA retaining the top 20 principal components. Integrate batches with Harmony on the top-20-PC embedding and cluster cells at resolution 1, since the paper's cell-subset assignments depend on this integrated embedding. Embed cells in 2D with UMAP and assign cell-type labels — including CD14+ monocytes and proliferating cells — from canonical marker expression. Finally, use Seurat's VlnPlot to plot MCEMP1, HLA-DRA and ETS1 across the annotated clusters; the run is successful only if heightened MCEMP1 and reduced HLA-DRA appear predominantly in the CD14+ monocyte cluster, with reduced HLA-DRA also seen broadly in other lineages and low ETS1 restricted to proliferating cells.

## Related tools

- **Seurat** (QC filtering, count normalization, PCA (top 20 PCs), clustering (resolution 1), UMAP embedding, and VlnPlot generation of MCEMP1/HLA-DRA/ETS1 across cell types)
- **Harmony** (Batch integration of Seurat clusters on the top-20-PC embedding prior to clustering)
- **UMAP** (2D embedding of integrated cells for visualization and cell-type annotation)

## Examples

```
Rscript -e 'pbmc <- RunPCA(pbmc, npcs = 20); pbmc <- RunHarmony(pbmc, "batch"); pbmc <- FindClusters(pbmc, resolution = 1); pbmc <- RunUMAP(pbmc, reduction = "harmony", dims = 1:20); VlnPlot(pbmc, c("MCEMP1", "HLA-DRA", "ETS1"), group.by = "celltype")'
```

## Evaluation signals

- Cell count after QC matches the reported ~48,583 PBMCs retained under the paper's thresholds.
- Harmony integration is run on the top-20-PC embedding and clustering is performed at resolution 1 (not default parameters).
- Violin plots show heightened MCEMP1 predominantly in the CD14+ monocyte cluster.
- HLA-DRA is reduced in CD14+ monocytes and broadly reduced across other immune lineages.
- ETS1 expression is low and restricted to proliferating cells, matching the reported pattern.

## Limitations

- The included datasets did not directly evaluate the effect of different SARS-CoV-2 variants on transcriptomic responses to severe COVID-19.
- Heterogeneity across study datasets was noted, so patient-level differences influencing MCEMP1 and HLA-DRA expression are not fully resolved.
- Cell-type labels rely on canonical marker expression rather than a reference atlas, so annotation of rare subsets may be imprecise.

## Evidence

- [results] Single-cell analysis of PBMCs revealed that heightened MCEMP1 and reduced HLA-DRA expression in severe COVID-19 patients were seen predominantly in CD14+ monocytes, with reduced HLA-DRA also broadly detected across other immune cell lineages.: [evidence span withheld — anchor status: not_found]
- [methods] In Seurat, apply the quality-control thresholds specified in the methods to retain 48,583 cells, then normalize counts and run PCA keeping the top 20 principal components.: [evidence span withheld — anchor status: not_found]
- [methods] Integrate batches with Harmony on the top-20-PC embedding and cluster the cells at resolution 1.: [evidence span withheld — anchor status: not_found]
- [abstract] Single-cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [abstract] The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets.: "The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets."
