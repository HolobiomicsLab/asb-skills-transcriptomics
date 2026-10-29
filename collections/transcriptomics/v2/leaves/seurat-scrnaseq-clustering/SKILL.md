---
name: seurat-scrnaseq-clustering
description: Use when starting from raw 10x scRNA-seq counts (e.g., Cell Ranger filtered_gene_bc_matrices
  or an RDS Seurat object) and unsupervised clusters are needed as input for ScType
  marker-based cell-type annotation or SNV-score assignment.
when_to_use_negative:
- Input is already a clustered or pre-annotated Seurat object — pass it directly to
  run_sctype/ScType instead of re-clustering
- Only the annotation step is needed and the wrapper run_sctype handles processing
  — avoid manually redoing NormalizeData/ScaleData with incompatible settings
- The data are single-cell DNA-seq alignments; VarTrix variant assignment operates
  on BAM/CRAM, not on Seurat clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat v4.0
  role: R package used for scRNA-seq data processing and normalization (NormalizeData,
    ScaleData, FindVariableFeatures, PCA, FindNeighbors, FindClusters, RunUMAP) prior
    to ScType annotation
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: ScType
  role: Downstream consumer of the scaled Seurat matrix; sctype_score/run_sctype annotate
    the Seurat clusters using positive and negative marker gene sets
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: vartrix
  role: Links called SNVs to the cell barcodes belonging to the Seurat clusters to
    compute per-cluster ScType SNV scores
  repo: https://github.com/10XGenomics/vartrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:abd551d8e2970a1da3d517c701164c388627efd17243320b5ecf96ef7b1a2fe9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  - Seurat v4.0
  - Strelka2
  - samtools
  - vartrix
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# seurat-scrnaseq-clustering

## Summary

Preprocess and cluster a single scRNA-seq dataset with Seurat (LogNormalize to scale.factor 10000, vst variable-feature selection, PCA, kNN graph clustering at resolution ~0.8, UMAP) to produce unsupervised clusters that downstream ScType cell-type annotation can label.

## When to use

Use when starting from raw 10x scRNA-seq counts (e.g., Cell Ranger filtered_gene_bc_matrices or an RDS Seurat object) and unsupervised clusters are needed as input for ScType marker-based cell-type annotation or SNV-score assignment.

## When NOT to use

- Input is already a clustered or pre-annotated Seurat object — pass it directly to run_sctype/ScType instead of re-clustering
- Only the annotation step is needed and the wrapper run_sctype handles processing — avoid manually redoing NormalizeData/ScaleData with incompatible settings
- The data are single-cell DNA-seq alignments; VarTrix variant assignment operates on BAM/CRAM, not on Seurat clustering

## Inputs

- Raw 10x Genomics scRNA-seq gene-barcode count matrix (filtered_gene_bc_matrices / mtx from Cell Ranger)
- Seurat object stored as .RDS (e.g., processed AML patient sample or pbmc3k example RDS)

## Outputs

- Clustered Seurat object with cluster assignments (Seurat metadata column), normalized/scaled expression assay, PCA and UMAP reductions
- Cell x cluster structure used downstream for sctype_score annotation and per-cluster SNV score aggregation

## How to apply

Initialize a Seurat object from the 10x counts with CreateSeuratObject (min.cells = 3, min.features = 200), optionally compute percent.mt with PercentageFeatureSet(pattern = "^MT-") and filter on QC metrics such as nFeature_RNA 200–2500 and percent.mt < 5. Normalize with NormalizeData(normalization.method = "LogNormalize", scale.factor = 10000), select 2000 variable features with FindVariableFeatures(selection.method = "vst"), then ScaleData over all genes (features = rownames) since ScType expects a scaled matrix and calls sctype_score with scaled = TRUE. Run PCA and pick the number of principal components from the ElbowPlot (10 PCs in the reference PBMC example), build the neighbor graph with FindNeighbors and cluster with FindClusters(resolution = 0.8), then visualize with RunUMAP on the same PCs. In the AML workflow these clusters (e.g., CD34+ HSC/MPP, ISG+ blasts, CD24+ CD66+ neutrophils, memory CD8+ T cells) are subsequently annotated by ScType using ScTypeDB (Immune system tissue) and linked to per-cluster SNV scores via vartrix.

## Related tools

- **Seurat v4.0** (R package used for scRNA-seq data processing and normalization (NormalizeData, ScaleData, FindVariableFeatures, PCA, FindNeighbors, FindClusters, RunUMAP) prior to ScType annotation) — https://github.com/IanevskiAleksandr/sc-type
- **ScType** (Downstream consumer of the scaled Seurat matrix; sctype_score/run_sctype annotate the Seurat clusters using positive and negative marker gene sets) — https://github.com/IanevskiAleksandr/sc-type
- **vartrix** (Links called SNVs to the cell barcodes belonging to the Seurat clusters to compute per-cluster ScType SNV scores) — https://github.com/10XGenomics/vartrix

## Examples

```
lapply(c("dplyr","Seurat","HGNChelper"), library, character.only = T); pbmc <- CreateSeuratObject(counts = pbmc.data, project = "pbmc3k", min.cells = 3, min.features = 200); pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000)
```

## Evaluation signals

- The scaled matrix used by sctype_score was produced with ScaleData over all rownames(pbmc) genes, and sctype_score is called with scaled = TRUE (mismatched scaling invalidates annotation)
- The ElbowPlot justifies the number of PCs (10 in the reference PBMC example) used for both FindNeighbors and RunUMAP — both must use the same dims
- Clusters are stable and UMAP visualization shows discrete populations (DimPlot reduction = "umap") before annotation
- Downstream ScType assignment yields biologically coherent labels (e.g., immune cell types from ScTypeDB 'Immune system'), and in the AML case malignant CD34+ HSC/MPP and ISG+ blast clusters carry higher SNV scores than non-malignant neutrophils and memory CD8+ T cells (SNV score < 20)
- QC filtering (optional subset on nFeature_RNA > 200 & nFeature_RNA < 2500 & percent.mt < 5) does not remove the populations of interest

## Limitations

- ScType's z-score transformation for combining multiple markers may be suboptimal for single-cell transcriptomic data
- Drop-out events in scRNA-seq data are not corrected in the current version, which can affect marker-based annotation
- QC filtering step is optional and must be guided by QC metric visualizations; the README leaves it commented out by default

## Evidence

- [discussion] Seurat v4.0 for data processing and normalization: "We utilized Seurat v4.042 for data processing and normalization."
- [readme] README normalize, scale, PCA and cluster steps: "pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000); pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000)"
- [readme] README cluster and UMAP commands: "pbmc <- FindNeighbors(pbmc, dims = 1:10); pbmc <- FindClusters(pbmc, resolution = 0.8); pbmc <- RunUMAP(pbmc, dims = 1:10)"
- [readme] README mitochondrial percentage and optional QC filter: "pbmc[["percent.mt"]] <- PercentageFeatureSet(pbmc, pattern = "^MT-"); # pbmc <- subset(pbmc, subset = nFeature_RNA > 200 & nFeature_RNA < 2500 & percent.mt < 5)"
- [readme] Scaled data requirement for sctype_score: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
