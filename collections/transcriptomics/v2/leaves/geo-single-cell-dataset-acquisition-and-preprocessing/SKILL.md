---
name: geo-single-cell-dataset-acquisition-and-preprocessing
description: Use when benchmarking cell-type annotation methods on the ScType paper
  datasets, or when standardized GEO/10x scRNA-seq download plus Seurat v4.0 normalization
  and clustering is needed as input to annotation.
when_to_use_negative:
- Input is already a normalized, scaled, and clustered Seurat object — proceed directly
  to gene_sets_prepare()/sctype_score() or the run_sctype() wrapper
- The task is malignant vs non-malignant cell discrimination via single-cell SNV calling
  (vartrix), which is a separate downstream ScType capability, not preprocessing
- Datasets are not human or mouse scRNA-seq count matrices compatible with Seurat's
  Read10X/CreateSeuratObject pipeline
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: 'Seurat v4.0 used for data processing: LogNormalize (scale.factor=10000),
    2000 vst variable features, ScaleData, RunPCA, FindNeighbors/FindClusters (resolution=0.8
    for PBMC3k)'
- name: R
  role: Open-source R environment in which ScType and the Seurat preprocessing pipeline
    are implemented
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: dplyr
  role: R library loaded alongside Seurat for data manipulation during preprocessing
- name: ScType
  role: Downstream consumer of the preprocessed/clustered Seurat objects via sctype_score()
    and run_sctype()
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:ca15912b88345c8461e307ec0accdf166cfec56358253becd7d402c1545c42a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - SCINA
  - ScType
  - Seurat
  - dplyr
  - scCATCH
  - scSorter
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# geo-single-cell-dataset-acquisition-and-preprocessing

## Summary

Acquire the six human and mouse scRNA-seq benchmark datasets (five from GEO plus the 10x PBMC3k filtered gene-barcode matrices) and preprocess them in Seurat v4.0 with LogNormalize, 2000 variable features, scaling, PCA, and clustering so they are ready for ScType cell-type annotation and method comparison.

## When to use

Use when benchmarking cell-type annotation methods on the ScType paper datasets, or when standardized GEO/10x scRNA-seq download plus Seurat v4.0 normalization and clustering is needed as input to annotation.

## When NOT to use

- Input is already a normalized, scaled, and clustered Seurat object — proceed directly to gene_sets_prepare()/sctype_score() or the run_sctype() wrapper
- The task is malignant vs non-malignant cell discrimination via single-cell SNV calling (vartrix), which is a separate downstream ScType capability, not preprocessing
- Datasets are not human or mouse scRNA-seq count matrices compatible with Seurat's Read10X/CreateSeuratObject pipeline

## Inputs

- GEO datasets: GSE124395 (human liver), GSE67835 (human brain), GSE85241 (human pancreas), GSE63269 (mouse lung), GSE63473 (mouse retina)
- 10x Genomics pbmc3k filtered_gene_bc_matrices.tar.gz
- Cell barcode matrices in 10x Matrix Market format

## Outputs

- Normalized and scaled Seurat v4.0 objects with PCA embeddings and cluster assignments per dataset
- Clustered datasets ready for sctype_score() annotation and baseline method comparison

## How to apply

Download the five GEO datasets (GSE124395 human liver, GSE67835 human brain, GSE85241 human pancreas, GSE63269 mouse lung, GSE63473 mouse retina) and the 10x pbmc3k filtered gene-barcode matrices tarball. Build a Seurat v4.0 object (min.cells=3, min.features=200) and normalize with LogNormalize at scale.factor=10000, then select 2000 variable features with the vst method. Scale all features (features = rownames), run PCA, select the number of PCs from an ElbowPlot (10 PCs for PBMC3k), and cluster via FindNeighbors/FindClusters (resolution=0.8 for PBMC3k). Optionally compute percent.mt with PercentageFeatureSet and apply QC filtering (nFeature_RNA > 200 & < 2500 & percent.mt < 5), which the README notes is optional. The resulting clustered, scaled Seurat objects are the standardized inputs for gene_sets_prepare()/sctype_score() and for the scSorter/SCINA/scCATCH baselines.

## Related tools

- **Seurat** (Seurat v4.0 used for data processing: LogNormalize (scale.factor=10000), 2000 vst variable features, ScaleData, RunPCA, FindNeighbors/FindClusters (resolution=0.8 for PBMC3k))
- **R** (Open-source R environment in which ScType and the Seurat preprocessing pipeline are implemented) — https://github.com/IanevskiAleksandr/sc-type
- **dplyr** (R library loaded alongside Seurat for data manipulation during preprocessing)
- **ScType** (Downstream consumer of the preprocessed/clustered Seurat objects via sctype_score() and run_sctype()) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000); pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000); pbmc <- ScaleData(pbmc, features = rownames(pbmc)); pbmc <- FindNeighbors(pbmc, dims = 1:10); pbmc <- FindClusters(pbmc, resolution = 0.8)
```

## Evaluation signals

- Each dataset yields a Seurat object with 2000 variable features selected by the vst method and expression normalized by LogNormalize at scale.factor=10000
- PBMC3k clustering uses 10 PCs (chosen from ElbowPlot) and FindClusters resolution=0.8, matching the README example
- All six datasets (five GEO + pbmc3k) are processed identically so ScType, scSorter, SCINA, and scCATCH receive the same clustered inputs
- Downstream sctype_score() runs with scaled=TRUE on the scaled expression matrix, confirming scaling was retained from preprocessing
- Benchmark reproduces 72/73 correctly assigned cell types (98.6%) and >94% per-cell annotation accuracy on every dataset

## Limitations

- QC-based cell filtering (nFeature_RNA > 200 & < 2500 & percent.mt < 5) is optional and commented out in the README; unfiltered objects may carry low-quality cells
- The number of PCs is chosen manually from an ElbowPlot (10 for PBMC3k), introducing a dataset-specific decision point
- When annotating human brain clusters, markers identified from the human brain dataset itself must be excluded from the comparison to avoid bias

## Evidence

- [readme] Normalization and clustering parameters: "pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000)
pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000)"
- [discussion] GEO dataset list: "Five datasets were downloaded from Gene Express Omnibus (GEO): Human Liver (GSE124395)"
- [discussion] PBMC3k source URL: "Human PBMC dataset was downloaded from the 10x Genomics Dataset Repository (https://s3-us-west-2.amazonaws.com/10x.files/samples/cell/pbmc3k/pbmc3k_filtered_gene_bc_matrices.tar.gz)."
- [discussion] Seurat version statement: "We utilized Seurat v4.042 for data processing and normalization."
- [readme] PC and clustering choices: "pbmc <- FindNeighbors(pbmc, dims = 1:10)
pbmc <- FindClusters(pbmc, resolution = 0.8)"
- [methods] Optional QC filtering: "# pbmc <- subset(pbmc, subset = nFeature_RNA > 200 & nFeature_RNA < 2500 & percent.mt < 5)"
