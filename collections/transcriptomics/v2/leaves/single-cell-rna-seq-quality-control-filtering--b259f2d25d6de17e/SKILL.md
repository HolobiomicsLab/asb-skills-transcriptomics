---
name: single-cell-rna-seq-quality-control-filtering
description: Use when a raw single-cell RNA-seq count matrix from peripheral blood
  (e.g. GSE149689 PBMCs) must be QC-filtered and normalized in Seurat prior to Harmony
  integration, UMAP embedding, and cell-type-level differential expression.
when_to_use_negative:
- Input is bulk RNA-seq data (the article's 7 transcriptomics datasets such as GSE155454
  and GSE157103 are bulk, not single-cell)
- A processed/annotated single-cell object is already provided — QC-filtering would
  discard valid cells
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Applies the QC thresholds to retain 48,583 cells, normalizes counts, runs
    PCA (top 20 PCs), clusters, and generates VlnPlot violin plots of MCEMP1/HLA-DRA/ETS1
- name: Harmony
  role: Batch integration on the top-20-PC Seurat embedding before clustering at resolution
    1
- name: UMAP
  role: 2D embedding of the integrated cells for visualization of annotated cell types
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:42741f78d9970bd6a8b6319843565baf04e42190fc390ea3bef8ed8109491f26
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - R
  - SCALA
  - Seurat
  - UMAP
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-rna-seq-quality-control-filtering

## Summary

Quality-control and filtering of raw single-cell RNA-seq count matrices (e.g. GEO PBMC datasets such as GSE149689) in Seurat, applying study-specified QC thresholds to retain a clean cell population before normalization, PCA, batch integration, and cell-type annotation.

## When to use

Use when a raw single-cell RNA-seq count matrix from peripheral blood (e.g. GSE149689 PBMCs) must be QC-filtered and normalized in Seurat prior to Harmony integration, UMAP embedding, and cell-type-level differential expression.

## When NOT to use

- Input is bulk RNA-seq data (the article's 7 transcriptomics datasets such as GSE155454 and GSE157103 are bulk, not single-cell)
- A processed/annotated single-cell object is already provided — QC-filtering would discard valid cells

## Inputs

- Raw single-cell RNA-seq count matrix from GEO (e.g. GSE149689 PBMC dataset)
- Study-specified QC thresholds (methods section) for cell filtering

## Outputs

- QC-filtered Seurat object retaining 48,583 cells
- Normalized expression matrix with top-20-PC embedding
- Harmony-integrated clusters (resolution 1) with UMAP 2D embedding
- Cell-type annotations (CD14+ monocytes, proliferating cells, other lineages)
- VlnPlot violin plots of MCEMP1, HLA-DRA, and ETS1 across annotated clusters

## How to apply

Download the raw PBMC count matrix (e.g. GSE149689 from GEO) and apply the quality-control thresholds specified in the study's methods in Seurat, retaining 48,583 cells. Normalize the filtered counts and run PCA keeping the top 20 principal components, since these PCs capture the dominant variance used for downstream integration. Integrate batches with Harmony on the top-20-PC embedding and cluster the cells at resolution 1, then embed with UMAP for visualization. Assign cell-type labels (including CD14+ monocytes and proliferating cells) from canonical marker expression, and validate the pipeline by generating VlnPlot violin plots of MCEMP1, HLA-DRA, and ETS1 across annotated clusters.

## Related tools

- **Seurat** (Applies the QC thresholds to retain 48,583 cells, normalizes counts, runs PCA (top 20 PCs), clusters, and generates VlnPlot violin plots of MCEMP1/HLA-DRA/ETS1)
- **Harmony** (Batch integration on the top-20-PC Seurat embedding before clustering at resolution 1)
- **UMAP** (2D embedding of the integrated cells for visualization of annotated cell types)

## Evaluation signals

- Filtered object retains exactly the QC-passed cell count (48,583 cells for GSE149689 under the methods' thresholds)
- PCA embedding retains the top 20 principal components as input to Harmony integration
- Post-integration clusters at resolution 1 recover CD14+ monocytes and proliferating cells identifiable by canonical marker expression
- VlnPlot reproduces the reported pattern: heightened MCEMP1 and reduced HLA-DRA primarily in CD14+ monocytes, reduced HLA-DRA broadly across other lineages
- Low ETS1 expression is restricted to proliferating cells

## Limitations

- Dataset heterogeneity was noted; patient heterogeneity influencing MCEMP1 and HLA-DRA expression is not fully understood
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been validated; prospective studies are needed
- The role of MCEMP1 in monocytes, including gene regulation, function, and protein expression after SARS-CoV-2 infection, remains uncharacterized

## Evidence

- [abstract] Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [other] In Seurat, apply the quality-control thresholds specified in the methods to retain 48,583 cells, then normalize counts and run PCA keeping the top 20 principal components.: [evidence span withheld — anchor status: not_found]
- [other] Integrate batches with Harmony on the top-20-PC embedding and cluster the cells at resolution 1.: [evidence span withheld — anchor status: not_found]
- [discussion] we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
- [abstract] Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19.: "Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19."
