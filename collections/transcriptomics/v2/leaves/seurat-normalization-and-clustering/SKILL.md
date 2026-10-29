---
name: seurat-normalization-and-clustering
description: Use when the input is a raw single-cell count matrix (e.g. from GEO accessions
  like GSE124395 or GSE63473, or 10x filtered_gene_bc_matrices) and you need normalized,
  clustered, UMAP-embedded cells as input to marker-based cell-type annotation.
when_to_use_negative:
- The data is already a processed/scaled Seurat object — go directly to sctype_wrapper/run_sctype
  instead of re-clustering
- The aim is supervised cell-type classification requiring a labelled reference dataset
  (e.g. CaSTLe, ACTINN) rather than unsupervised marker-based annotation
- 'Only variable features were scaled: sctype_score requires the full-gene scale.data
  slot, so a pipeline that scaled only the 2000 vst features will break downstream
  annotation'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Performs normalization (LogNormalize, scale.factor = 10000), vst variable-feature
    selection, scaling of all genes, PCA, FindNeighbors/FindClusters clustering, and
    UMAP embedding prior to ScType scoring
- name: ScType
  role: Downstream consumer of the Seurat scale.data matrix via sctype_score / run_sctype
    for fully-automated cell-type annotation
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:c1eba07bd98649d2880e6fe8c063ed28505af673b3080ea212316f0fc08291c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# seurat-normalization-and-clustering

## Summary

Standard Seurat v4 pre-processing that turns a raw scRNA-seq count matrix into a clustered, UMAP-embedded Seurat object whose per-cluster scaled expression feeds downstream ScType cell-type scoring. It fixes LogNormalize (scale.factor = 10000), vst variable-feature selection, PCA-based graph clustering, and 10-PC UMAP embedding as the canonical recipe used in this paper.

## When to use

Use when the input is a raw single-cell count matrix (e.g. from GEO accessions like GSE124395 or GSE63473, or 10x filtered_gene_bc_matrices) and you need normalized, clustered, UMAP-embedded cells as input to marker-based cell-type annotation.

## When NOT to use

- The data is already a processed/scaled Seurat object — go directly to sctype_wrapper/run_sctype instead of re-clustering
- The aim is supervised cell-type classification requiring a labelled reference dataset (e.g. CaSTLe, ACTINN) rather than unsupervised marker-based annotation
- Only variable features were scaled: sctype_score requires the full-gene scale.data slot, so a pipeline that scaled only the 2000 vst features will break downstream annotation

## Inputs

- Raw scRNA-seq count matrix (10x filtered_gene_bc_matrices directory read via Read10X, or GEO count matrices e.g. GSE124395 human liver, GSE63473 mouse retina)
- Optional QC thresholds (nFeature_RNA, percent.mt) from Seurat tutorial visualizations

## Outputs

- Seurat object with RNA assay scale.data slot (all genes scaled), PCA reduction, clusters at resolution 0.8, and UMAP embedding
- UMAP visualization via DimPlot(reduction = 'umap') and ElbowPlot for PC choice

## How to apply

Load counts with CreateSeuratObject (min.cells = 3, min.features = 200), optionally compute percent.mt with PercentageFeatureSet(pattern = '^MT-') and subset on QC metrics (nFeature_RNA > 200 & < 2500 & percent.mt < 5). Normalize with NormalizeData(normalization.method = 'LogNormalize', scale.factor = 10000), select 2000 variable features with FindVariableFeatures(selection.method = 'vst'), then ScaleData over all genes (features = rownames) so the full scale.data slot is available for ScType scoring. Run PCA, choose ~10 PCs based on the ElbowPlot, cluster with FindNeighbors/FindClusters (dims = 1:10, resolution = 0.8) and embed with RunUMAP(dims = 1:10). The rationale, per the ScType paper and README, is that sctype_score consumes the scaled expression matrix from the RNA assay's scale.data slot, so all genes — not just variable features — must be scaled.

## Related tools

- **Seurat** (Performs normalization (LogNormalize, scale.factor = 10000), vst variable-feature selection, scaling of all genes, PCA, FindNeighbors/FindClusters clustering, and UMAP embedding prior to ScType scoring)
- **ScType** (Downstream consumer of the Seurat scale.data matrix via sctype_score / run_sctype for fully-automated cell-type annotation) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000); pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000); pbmc <- ScaleData(pbmc, features = rownames(pbmc)); pbmc <- FindNeighbors(pbmc, dims = 1:10); pbmc <- FindClusters(pbmc, resolution = 0.8); pbmc <- RunUMAP(pbmc, dims = 1:10)
```

## Evaluation signals

- ElbowPlot supports the choice of 10 PCs used for FindNeighbors and RunUMAP (dims = 1:10)
- pbmc[['percent.mt']] is populated and, if filtering is applied, subsets used nFeature_RNA > 200 & < 2500 & percent.mt < 5
- ScaleData was run on features = rownames(pbmc) (all genes), so the scale.data slot contains every gene needed by gene_sets_prepare/sctype_score
- Clustering at resolution = 0.8 yields the expected cluster granularity (e.g. 17 clusters in the human liver atlas, distinguishable amacrine/bipolar subtypes in mouse retina)
- DimPlot on the UMAP shows separable clusters consistent with the marker-based annotations assigned later

## Limitations

- ScType's z-score transformation of expression is acknowledged as possibly suboptimal for single-cell transcriptomic data
- Drop-out events in scRNA-seq data are not corrected in the current version of the pipeline
- Cluster resolution and PC count (0.8, 10 PCs) were tuned for the paper's datasets; other tissues may require re-inspection of the ElbowPlot and resolution

## Evidence

- [full_text] Normalize with LogNormalize (scale.factor = 10000), select 2000 variable features (vst), scale all genes, run PCA, cluster with FindNeighbors/FindClusters on the first 10 PCs at resolution 0.8: [evidence span withheld — anchor status: not_found]
- [discussion] We utilized Seurat v4.0 for data processing and normalization.: "We utilized Seurat v4.042 for data processing and normalization."
- [readme] pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000): "pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000)
pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000)"
- [readme] pbmc <- FindNeighbors(pbmc, dims = 1:10): "# Check number of PC components (we selected 10 PCs for downstream analysis, based on Elbow plot)
ElbowPlot(pbmc)"
