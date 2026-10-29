---
name: seurat-clustering-pbmc-workflow
description: Use when a raw 10x filtered gene-barcode matrix (e.g. pbmc3k_filtered_gene_bc_matrices)
  must be normalized, scaled, and clustered in Seurat before marker-based cell-type
  annotation such as ScType.
when_to_use_negative:
- Input is already a processed/annotated Seurat object with scaled data — use the
  run_sctype wrapper instead of re-clustering
- The dataset is not human or mouse PBMC-like scRNA-seq from a supported tissue in
  the ScType marker database
- A supervised reference-based annotator (e.g. CaSTLe, ACTINN) is required because
  labeled training data exist
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat v4.0
  role: 'Data processing and normalization: CreateSeuratObject, NormalizeData (LogNormalize,
    scale.factor = 10000), ScaleData, RunPCA, FindNeighbors/FindClusters (dims 1:10,
    resolution 0.8)'
- name: ScType
  role: 'Downstream consumer of the scaled matrix and clusters: gene_sets_prepare
    and sctype_score_ assign cell types to the Seurat clusters'
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: dplyr
  role: R data manipulation library loaded alongside Seurat in the workflow
- name: openxlsx
  role: Used to read the ScTypeDB_full.xlsx marker database in the reimplementation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:56cb0709b1d35976ed6f029acbfe75f9139160ac38905a1cbc43cf6adf1554d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  - Seurat v4.0
  - dplyr
  - openxlsx
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# seurat-clustering-pbmc-workflow

## Summary

Seurat v4.0 preprocessing and clustering workflow for the PBMC 3k dataset that produces the scaled expression matrix and cluster identities required for downstream ScType cell-type annotation.

## When to use

Use when a raw 10x filtered gene-barcode matrix (e.g. pbmc3k_filtered_gene_bc_matrices) must be normalized, scaled, and clustered in Seurat before marker-based cell-type annotation such as ScType.

## When NOT to use

- Input is already a processed/annotated Seurat object with scaled data — use the run_sctype wrapper instead of re-clustering
- The dataset is not human or mouse PBMC-like scRNA-seq from a supported tissue in the ScType marker database
- A supervised reference-based annotator (e.g. CaSTLe, ACTINN) is required because labeled training data exist

## Inputs

- 10x Genomics filtered gene-barcode matrix (pbmc3k_filtered_gene_bc_matrices.tar.gz, hg19) loaded via Read10X
- ScType marker database file (ScTypeDB_full.xlsx or ScTypeDB_short.xlsx) restricted to a tissue type such as 'Immune system'

## Outputs

- Seurat object with scale.data matrix (all genes scaled), PCA reduction, and Seurat cluster assignments from FindClusters(resolution = 0.8)
- Cluster-level input matrix for sctype_score_ (scaled = TRUE) yielding per-cluster cell-type scores (es.max)

## How to apply

Load the 10x matrix with Read10X and initialize the object with CreateSeuratObject(counts = pbmc.data, project = "pbmc3k", min.cells = 3, min.features = 200). Optionally compute percent.mt via PercentageFeatureSet(pattern = "^MT-") and subset on QC metrics. Normalize with LogNormalize at scale.factor = 10000, then ScaleData on all genes (features = rownames(pbmc)) so that every gene, not only variable features, is available to the ScType z-score step. Run PCA on variable features and cluster with FindNeighbors(dims = 1:10) and FindClusters(resolution = 0.8); the number of PCs (10) is chosen from the ElbowPlot. The resulting scale.data slot and Seurat cluster IDs are the direct inputs to gene_sets_prepare and sctype_score (scaled = TRUE), which z-score genes and aggregate positive/negative marker enrichment per cluster.

## Related tools

- **Seurat v4.0** (Data processing and normalization: CreateSeuratObject, NormalizeData (LogNormalize, scale.factor = 10000), ScaleData, RunPCA, FindNeighbors/FindClusters (dims 1:10, resolution 0.8))
- **ScType** (Downstream consumer of the scaled matrix and clusters: gene_sets_prepare and sctype_score_ assign cell types to the Seurat clusters) — https://github.com/IanevskiAleksandr/sc-type
- **dplyr** (R data manipulation library loaded alongside Seurat in the workflow)
- **openxlsx** (Used to read the ScTypeDB_full.xlsx marker database in the reimplementation)

## Examples

```
pbmc <- CreateSeuratObject(counts = Read10X(data.dir = "./filtered_gene_bc_matrices/hg19/"), project = "pbmc3k", min.cells = 3, min.features = 200); pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000); pbmc <- ScaleData(pbmc, features = rownames(pbmc)); pbmc <- FindNeighbors(pbmc, dims = 1:10); pbmc <- FindClusters(pbmc, resolution = 0.8)
```

## Evaluation signals

- CreateSeuratObject parameters match min.cells = 3 and min.features = 200
- Normalization used LogNormalize with scale.factor = 10000 and ScaleData covered all genes (features = rownames(pbmc))
- Clustering produced stable clusters with FindNeighbors(dims = 1:10) and FindClusters(resolution = 0.8), with 10 PCs justified by ElbowPlot
- Downstream es.max scores from sctype_score_ (scaled = TRUE, gs = gs_positive, gs2 = gs_negative) reproduce identical cluster-level cell-type labels versus the reference run
- Clusters whose ScType score falls below ncells/4 are labeled 'Unknown'

## Limitations

- ScType uses z-score transformation, which may be suboptimal for single-cell transcriptomic data (drop-out events are not corrected in the current version)
- QC-based cell filtering is optional and left to the analyst per the Seurat PBMC3k tutorial (nFeature_RNA > 200 & nFeature_RNA < 2500 & percent.mt < 5)
- Only 10 PCs and resolution 0.8 were validated on PBMC 3k; other datasets need ElbowPlot-guided re-tuning

## Evidence

- [discussion] We utilized Seurat v4.042 for data processing and normalization.: "We utilized Seurat v4.042 for data processing and normalization."
- [readme] CreateSeuratObject parameters: "pbmc <- CreateSeuratObject(counts = pbmc.data, project = "pbmc3k", min.cells = 3, min.features = 200)"
- [readme] NormalizeData call: "pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000)"
- [readme] Clustering parameters: "pbmc <- FindNeighbors(pbmc, dims = 1:10)
pbmc <- FindClusters(pbmc, resolution = 0.8)"
- [readme] Scaled matrix is ScType input: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
