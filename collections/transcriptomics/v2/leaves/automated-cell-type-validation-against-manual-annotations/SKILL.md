---
name: automated-cell-type-validation-against-manual-annotations
description: Use when a raw scRNA-seq dataset has published manual cell-type annotations
  and you need to check whether an unsupervised annotator (ScType) reproduces them
  from raw counts alone.
when_to_use_negative:
- The dataset has no published manual annotations to validate against, so there is
  no ground truth for the benchmark.
- The tissue is not represented in the ScType marker database, so gene_sets_prepare()
  cannot yield Liver-like gs_positive/gs_negative sets.
- The input is already an annotated Seurat object and the goal is de novo discovery
  rather than validating an automated annotator.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
tools:
- name: ScType
  role: Computes positive/negative marker specificity scores (sctype_score) on the
    scaled expression matrix to assign a cell type to each Seurat cluster
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat v4
  role: Preprocesses raw GSE124395 counts (LogNormalize, 2000 vst variable features,
    scaling, PCA, FindClusters at resolution 0.8, UMAP) prior to annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:9934d76c490cdfa3cddece01edf2bf219585bdb2e520c02464e01a3616905ec1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat v4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# automated-cell-type-validation-against-manual-annotations

## Summary

Benchmark the ScType fully-automated cell-type annotator against the manual annotations of a published atlas, e.g. reproducing the human liver atlas (GSE124395) where 17 Seurat clusters must map to 11 liver cell types, including the CD19/CD20/CD138 split of B-cell populations.

## When to use

Use when a raw scRNA-seq dataset has published manual cell-type annotations and you need to check whether an unsupervised annotator (ScType) reproduces them from raw counts alone.

## When NOT to use

- The dataset has no published manual annotations to validate against, so there is no ground truth for the benchmark.
- The tissue is not represented in the ScType marker database, so gene_sets_prepare() cannot yield Liver-like gs_positive/gs_negative sets.
- The input is already an annotated Seurat object and the goal is de novo discovery rather than validating an automated annotator.

## Inputs

- Raw human liver scRNA-seq count matrix from GEO (GSE124395)
- ScType marker database file ScTypeDB_full.xlsx with tissue type 'Liver'
- Seurat object (Seurat v4) with scale.data slot populated for all genes

## Outputs

- Cluster-to-cell-type annotation table with per-cluster ScType scores (sctype_liver_annotations.csv)
- Seurat object with cluster labels, scores, and UMAP embedding
- Comparison of 17 automated cluster labels against the 11 manually annotated liver cell types

## How to apply

Process the raw GSE124395 counts in Seurat v4: NormalizeData with LogNormalize (scale.factor=10000), FindVariableFeatures (vst, 2000 features), ScaleData on all genes, PCA, then FindNeighbors/FindClusters on the first 10 PCs at resolution 0.8 and RunUMAP. Build tissue-restricted marker sets with gene_sets_prepare() using ScTypeDB_full.xlsx and tissue type 'Liver', yielding gs_positive and gs_negative lists. Extract the scale.data slot and run sctype_score(scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative), then aggregate the cell-type-by-cell score matrix per Seurat cluster by row sums and take the top-scoring type per cluster; label clusters with scores < ncells/4 as 'Unknown'. Judge success by comparing the automated cluster-to-type table against the original manual annotations: the expected result is 17 clusters covering 11 liver cell types, with the two jointly-annotated B-cell clusters resolved as immature B cells (CD19+/CD20+) versus plasma cells (CD138+, CD19/CD20-negative); write the table with scores to sctype_liver_annotations.csv.

## Related tools

- **ScType** (Computes positive/negative marker specificity scores (sctype_score) on the scaled expression matrix to assign a cell type to each Seurat cluster) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat v4** (Preprocesses raw GSE124395 counts (LogNormalize, 2000 vst variable features, scaling, PCA, FindClusters at resolution 0.8, UMAP) prior to annotation)

## Examples

```
gs_list <- gene_sets_prepare("ScTypeDB_full.xlsx", "Liver"); es.max <- sctype_score(scRNAseqData = eset, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Automated assignments reproduce 17 clusters covering 11 liver cell types, matching the original manual annotations.
- The two jointly-annotated B-cell clusters separate into immature B cells (CD19+/CD20+) and plasma cells (CD138+, CD19/CD20-negative).
- No cluster scoring at or above ncells/4 is mislabeled 'Unknown' — the score < ncells/4 threshold only masks low-confidence clusters.
- The cluster-to-type table with scores is written to sctype_liver_annotations.csv.
- Agreement with the original paper's reported concordance between ScType and manual annotations is achieved using only raw counts as input.

## Limitations

- ScType's z-score standardization of gene expression may be suboptimal for single-cell transcriptomic data.
- Drop-out events in scRNA-seq data are not corrected in the current version.
- Negative markers are still relatively few compared to positive markers, which can limit annotation resolution.

## Evidence

- [abstract] Using six scRNA-seq datasets from various human and mouse tissues, we show how ScType provides unbiased and accurate cell type annotations: "Using six scRNA-seq datasets from various human and mouse tissues, we show how ScType provides unbiased and accurate cell type annotations"
- [discussion] Five datasets were downloaded from Gene Express Omnibus (GEO): Human Liver (GSE124395): "Five datasets were downloaded from Gene Express Omnibus (GEO): Human Liver (GSE124395)"
- [discussion] We utilized Seurat v4.042 for data processing and normalization.: "We utilized Seurat v4.042 for data processing and normalization."
- [discussion] ScType requires neither reference scRNA-seq datasets nor manual selection of marker genes: "ScType requires neither reference scRNA-seq datasets nor manual selection of marker genes"
- [methods] tissue_guess <- auto_detect_tissue_type(path_to_db_file = db_, seuratObject = pbmc, scaled = TRUE, assay = "RNA"): "tissue_guess <- auto_detect_tissue_type(path_to_db_file = db_, seuratObject = pbmc, scaled = TRUE, assay = "RNA")"
- [readme] es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative): "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
