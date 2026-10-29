---
name: retina-bipolar-and-amacrine-cell-subtype-annotation
description: Use when annotating clusters in mouse retina scRNA-seq data with ScType
  and you need to separate closely related subtypes (amacrine subpopulations; rod
  vs cone bipolar cells) beyond the original manual labels.
when_to_use_negative:
- When the tissue is not retina/eye and a different ScTypeDB tissue section should
  be used for gene_sets_prepare
- When a supervised classifier trained on annotated reference retinal data is required
  — ScType uses no reference dataset and no manually selected markers
- When clusters cannot be reliably resolved upstream (low-quality clustering, insufficient
  PCs), since ScType assigns labels per Seurat cluster and will return 'Unknown' for
  weak-score clusters
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ScType
  role: Fully-automated cell-type identification via positive and negative marker
    gene scoring (sctype_score, gene_sets_prepare)
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: 'Seurat v4.0 used for data processing and normalization: CreateSeuratObject,
    NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters,
    RunUMAP'
- name: R
  role: Open-source R implementation hosting the ScType package and the annotation
    workflow
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:63e5fb987d41017d6db84cd52f997e33fc350f6540ee4473b91e95c9b2bc31bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# retina-bipolar-and-amacrine-cell-subtype-annotation

## Summary

Applies ScType's fully-automated marker-based scoring to mouse retina scRNA-seq (GSE63473) to resolve closely related subtypes — three amacrine populations (GABAergic, glycinergic, starburst) and rod (PRKCA+/CAR8+) vs cone (SCGN+) bipolar cells — surpassing the original study's merged bipolar annotation.

## When to use

Use when annotating clusters in mouse retina scRNA-seq data with ScType and you need to separate closely related subtypes (amacrine subpopulations; rod vs cone bipolar cells) beyond the original manual labels.

## When NOT to use

- When the tissue is not retina/eye and a different ScTypeDB tissue section should be used for gene_sets_prepare
- When a supervised classifier trained on annotated reference retinal data is required — ScType uses no reference dataset and no manually selected markers
- When clusters cannot be reliably resolved upstream (low-quality clustering, insufficient PCs), since ScType assigns labels per Seurat cluster and will return 'Unknown' for weak-score clusters

## Inputs

- Mouse retina scRNA-seq raw count matrix from GEO accession GSE63473
- ScTypeDB_full.xlsx marker database (ScType marker database combining CellMarker and PanglaoDB)
- gene_sets_prepare.R and sctype_score_.R functions from github.com/IanevskiAleksandr/sc-type

## Outputs

- Seurat object with per-cluster cell-type labels in the sctype_classification metadata column
- UMAP embedding overlaid with ScType annotations (DimPlot)
- Per-cluster annotation table (cluster, cell type, score, ncells) exported as gse63473_sctype_annotations.csv

## How to apply

Download GSE63473 from GEO, build a Seurat object (CreateSeuratObject with min.cells = 3, min.features = 200), normalize with LogNormalize (scale.factor = 10000), select 2000 variable features (vst), scale all genes, run PCA and cluster with FindNeighbors/FindClusters at resolution = 0.8, then embed with RunUMAP on dims 1:10 after selecting ~10 PCs from the Elbow plot. Source gene_sets_prepare.R and sctype_score_.R from the sc-type GitHub repo and prepare tissue-matched positive and negative marker gene sets from ScTypeDB_full.xlsx restricted to the appropriate (eye/retina) tissue type. Extract the scaled expression matrix from the Seurat RNA assay scale.data slot (handling Seurat v4 vs v5 differences) and compute per-cell scores for all candidate cell types with sctype_score, supplying gs_list$gs_positive and gs_list$gs_negative. Aggregate es.max scores per Seurat cluster by summation, take the top-scoring cell type per cluster with dplyr top_n(n = 1, wt = scores), and set clusters whose top score is below ncells/4 to 'Unknown' before writing the winning label into the sctype_classification metadata column. The negative-marker term is essential: ScType is the only unsupervised method using marker specificity across both cell clusters and cell types with positive and negative markers, which drives its ability to split rod vs cone bipolar clusters.

## Related tools

- **ScType** (Fully-automated cell-type identification via positive and negative marker gene scoring (sctype_score, gene_sets_prepare)) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Seurat v4.0 used for data processing and normalization: CreateSeuratObject, NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters, RunUMAP)
- **R** (Open-source R implementation hosting the ScType package and the annotation workflow) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R"); source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/sctype_score_.R"); gs_list <- gene_sets_prepare("ScTypeDB_full.xlsx", "Eye"); es.max <- sctype_score(scRNAseqData = retina_scaled, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Amacrine cell clusters resolve into three distinct populations annotated as GABAergic, glycinergic and starburst
- Rod bipolar clusters are annotated separately from cone bipolar clusters, with rod bipolar cells expressing PRKCA and CAR8 and cone bipolar cells expressing SCGN
- All clusters whose top aggregated ScType score falls below ncells/4 are labelled 'Unknown'
- The exported gse63473_sctype_annotations.csv contains cluster, cell type, score and ncells for every Seurat cluster
- ScType's unsupervised labels match or refine the original manual retinal annotations (it enhances resolution where the original study merged bipolar cells into a single group)

## Limitations

- Clusters whose top aggregated score falls below ncells/4 are set to 'Unknown', so low-information clusters may remain unannotated
- ScType uses z-score transformation for combining multiple markers, which may be suboptimal for single-cell transcriptomic data
- Drop-out events in scRNA-seq data are not corrected in the current version, which can weaken negative-marker evidence
- The number of negative markers is still relatively low compared to positive markers, limiting annotation resolution

## Evidence

- [results] Rod vs cone bipolar separation: [evidence span withheld — anchor status: not_found]
- [discussion] Annotation algorithm uniqueness: "ScType is currently the only unsupervised method that makes use of the marker gene specificity across both cell clusters and cell types"
- [discussion] Z-score standardization step: "ScType first standardizes each gene expression profile into z-score across all cells."
- [readme] Function loading: "source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R"); source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/sctype_score_.R")"
- [readme] Scoring call: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [discussion] Processing and normalization tooling: "We utilized Seurat v4.042 for data processing and normalization."
