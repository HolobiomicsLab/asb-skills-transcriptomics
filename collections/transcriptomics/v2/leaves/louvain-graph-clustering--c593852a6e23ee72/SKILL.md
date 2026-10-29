---
name: louvain-graph-clustering
description: Use when QC-passed, SCTransform-normalized and integrated single-cell
  expression data needs unbiased cell-cluster identification prior to marker-based
  cell-type annotation.
when_to_use_negative:
- Input is raw unfiltered cell barcodes — QC filtering (850-7800 features, <13% mitochondrial)
  must precede clustering
- Clustering individual scRNAseq cells for circadian time prediction — tauFisher cannot
  overcome single-cell dropout and requires pseudobulk/cell-type-aggregated expression
  instead
- Data lacking cell-level resolution or already containing curated cell-type labels
  — clustering would be redundant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Runs PCA on the integrated dataset and executes the Louvain algorithm to generate
    cell clusters
- name: CellRanger
  role: Aligns raw GSE223109 sequencing output with the MM10 reference to produce
    the count matrix used upstream of clustering
- name: tauFisher
  role: Downstream consumer of the annotated cell types — applied at cell-type level
    to predict circadian time from pseudobulk profiles
  repo: https://github.com/micnngo/tauFisher
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:da4bea17468758298392d35e2dc38aa371ae015c7b7d3ebf1258453c70ad4f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - CellRanger
  - Seurat
  - tauFisher
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# louvain-graph-clustering

## Summary

Louvain graph-based community detection applied to a PCA-reduced, Seurat-integrated mouse dermis scRNAseq dataset (GSE223109) to identify unsupervised cell clusters, which are then annotated as fibroblasts, immune, muscle, and endothelial cells using canonical marker genes.

## When to use

Use when QC-passed, SCTransform-normalized and integrated single-cell expression data needs unbiased cell-cluster identification prior to marker-based cell-type annotation.

## When NOT to use

- Input is raw unfiltered cell barcodes — QC filtering (850-7800 features, <13% mitochondrial) must precede clustering
- Clustering individual scRNAseq cells for circadian time prediction — tauFisher cannot overcome single-cell dropout and requires pseudobulk/cell-type-aggregated expression instead
- Data lacking cell-level resolution or already containing curated cell-type labels — clustering would be redundant

## Inputs

- Seurat-integrated scRNAseq expression matrix (3250 SelectIntegrationFeatures, SCTransform-normalized per sample)
- QC-passed cell barcodes (850-7800 features, <13% mitochondrial genes) from CellRanger 3.1.0 / MM10 output

## Outputs

- Cluster assignment per cell (Louvain cluster labels)
- Marker-annotated cell types per cluster (fibroblast, immune, muscle, endothelial)
- cell_counts.csv tabulating cell numbers per annotated type

## How to apply

First run principal component analysis on the Seurat-integrated dataset built from 3250 integration features selected via SelectIntegrationFeatures across SCTransform-normalized samples. Then apply the Louvain algorithm to the neighbor graph computed in PCA space to generate cell clusters; in the GSE223109 mouse dermis time series this was done in Seurat after CellRanger 3.1.0 (MM10) alignment and QC filtering of cells with 850-7800 detected features and <13% mitochondrial genes. Annotate the resulting clusters by marker gene expression (Col1a1 fibroblasts, Acta1 muscle, Fabp4 endothelial, Cd52/Cd74 immune) rather than treating clusters as final cell types. The rationale is that Louvain modularity optimization on the k-nearest-neighbor graph groups transcriptionally similar cells without prior labels, enabling downstream tabulation of cells per annotated type (16,866 total QC-passing cells: fibroblasts 12,649, immune 3,353, muscle 722, endothelial 142).

## Related tools

- **Seurat** (Runs PCA on the integrated dataset and executes the Louvain algorithm to generate cell clusters)
- **CellRanger** (Aligns raw GSE223109 sequencing output with the MM10 reference to produce the count matrix used upstream of clustering)
- **tauFisher** (Downstream consumer of the annotated cell types — applied at cell-type level to predict circadian time from pseudobulk profiles) — https://github.com/micnngo/tauFisher

## Examples

```
seurat_obj <- FindNeighbors(seurat_obj, dims = 1:30); seurat_obj <- FindClusters(seurat_obj, resolution = 0.5)
```

## Evaluation signals

- Total QC-passing cells should equal 16,866 after clustering and annotation
- Cell-type counts should match fibroblasts 12,649, immune 3,353, muscle 722, endothelial 142
- Marker genes should show cluster-restricted expression: Col1a1 in fibroblasts, Acta1 in muscle, Fabp4 in endothelial, Cd52/Cd74 in immune cells
- Integration used exactly 3250 features selected via SelectIntegrationFeatures before PCA
- Resulting cell-count table saved as cell_counts.csv

## Limitations

- Cluster identities depend entirely on marker-gene annotation — Louvain clusters are not intrinsically cell types and can over- or under-segment rare populations (e.g., only 142 endothelial cells detected)
- Phase heterogeneity conclusions at cell-type level rest on computational analysis without experimental validation
- Single-cell-level circadian prediction is not feasible due to high scRNAseq dropout; aggregation into pseudobulk is required

## Evidence

- [discussion] PCA + Louvain workflow: "Principal component analysis was then done on the integrated dataset and the Louvain algorithm was used to generate the clusters"
- [discussion] QC filter: "Cells with 850-7800 features and less than 13% of mitochondrial genes were kept."
- [discussion] Integration features: "The SCTransform function was performed on each sample and 3250 integration features were selected using SelectIntegrationFeatures"
- [discussion] CellRanger MM10 alignment: "We used CellRanger version 3.1.0 with MM10 reference to process the raw sequencing output."
- [readme] README installation: [evidence span withheld — anchor status: external_doc]
