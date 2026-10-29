---
name: louvain-graph-based-cell-clustering
description: Use when a Seurat object has been normalized (LogNormalize, scale factor
  10,000) and PCA-reduced, and cell populations must be identified and later assigned
  labels (hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+
  neutrophils).
when_to_use_negative:
- Input is already a cell-type-annotated Seurat object (e.g., ready for scCrossTalk's
  create_scCrossTalk) rather than an unclustered count matrix
- Data are not single-cell per-cell expression profiles, e.g., bulk RNA-seq or LR-pair
  enrichment scores from scCrossTalk output
- Counts have not been normalized and batch-integrated, so Louvain clusters would
  reflect donor/time-point batch effects rather than biology
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R framework providing LogNormalize normalization, CCA integration (FindIntegrationAnchors),
    PCA, UMAP, and Louvain/Leiden graph-based clustering; compatible with Seurat 5
- name: scCrossTalk
  role: 'Downstream consumer of the clustered, annotated object: its create_scCrossTalk
    function requires the cell types produced by this clustering step'
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Execution environment; devtools is used to install scCrossTalk from GitHub
    or a local ZIP
- name: devtools
  role: Installs scCrossTalk via devtools::install_github('multitalk/scCrossTalk')
    or devtools::install_local
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:4873f0db7c52ed8ad991164c88ed0efd8b44a7c051a0f8861c1a621dc51f3f16
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - devtools
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# louvain-graph-based-cell-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Graph-based clustering of single-cell RNA-seq data via Louvain/Leiden community detection on a k-nearest-neighbor graph, applied after PCA on top variable features and UMAP embedding in Seurat. Used to recover hepatic cell populations from the 12 liver transplant samples that constitute the paper's 75,231-cell landscape.

## When to use

Use when a Seurat object has been normalized (LogNormalize, scale factor 10,000) and PCA-reduced, and cell populations must be identified and later assigned labels (hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils).

## When NOT to use

- Input is already a cell-type-annotated Seurat object (e.g., ready for scCrossTalk's create_scCrossTalk) rather than an unclustered count matrix
- Data are not single-cell per-cell expression profiles, e.g., bulk RNA-seq or LR-pair enrichment scores from scCrossTalk output
- Counts have not been normalized and batch-integrated, so Louvain clusters would reflect donor/time-point batch effects rather than biology

## Inputs

- Raw count matrix per sample from Cell Ranger with GRCh38 reference (12 human liver samples from 7 donors, cold preservation and two-hour portal perfusion stages)
- Seurat object after LogNormalize normalization and CCA-based integration
- PCA reduction computed on top variable features

## Outputs

- Cluster assignment (Louvain/Leiden communities) per cell on the k-NN graph
- UMAP embedding for visualizing cluster structure
- Annotated Seurat object with cell type labels (hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils), saved as liver_seurat_object.rds

## How to apply

Build a Seurat object per sample from the Cell Ranger/GRCh38 raw count matrix and normalize with LogNormalize (scale factor 10,000). Integrate the 12 samples from three non-EAD and four EAD patients using Seurat's CCA-based integration (FindIntegrationAnchors on PCA-reduced features) to remove donor/time-point batch effects. Run PCA on the top variable features, embed cells with UMAP, construct the k-nearest-neighbor graph, and cluster with Louvain/Leiden community detection. Assign cell type labels to the resulting clusters, consistent with the scCrossTalk celltype input format, covering hepatic parenchymal and non-parenchymal populations. Validate that the integrated object contains the reported total of 75,231 cells and save the annotated object as liver_seurat_object.rds; the annotated cell types then feed scCrossTalk's create_scCrossTalk(sc_data, sc_celltype, species) for cell-cell communication inference.

## Related tools

- **Seurat** (R framework providing LogNormalize normalization, CCA integration (FindIntegrationAnchors), PCA, UMAP, and Louvain/Leiden graph-based clustering; compatible with Seurat 5)
- **scCrossTalk** (Downstream consumer of the clustered, annotated object: its create_scCrossTalk function requires the cell types produced by this clustering step) — https://github.com/multitalk/scCrossTalk
- **R** (Execution environment; devtools is used to install scCrossTalk from GitHub or a local ZIP)
- **devtools** (Installs scCrossTalk via devtools::install_github('multitalk/scCrossTalk') or devtools::install_local)

## Examples

```
library(Seurat); obj <- NormalizeData(obj, normalization.method = "LogNormalize", scale.factor = 10000); obj <- RunPCA(obj); obj <- RunUMAP(obj, dims = 1:30); obj <- FindClusters(obj, algorithm = 1)
```

## Evaluation signals

- The integrated Seurat object contains exactly the reported total of 75,231 hepatic parenchymal and non-parenchymal cells
- Every cluster receives a cell type label consistent with the scCrossTalk celltype input format (e.g., hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils)
- Cell populations recover both parenchymal and non-parenchymal compartments from the 12 samples spanning three non-EAD and four EAD patients and two collection stages (cold preservation, two-hour portal perfusion)
- The annotated object is saved reproducibly as liver_seurat_object.rds and can be loaded into create_scCrossTalk(sc_data, sc_celltype, species)
- UMAP embedding shows clusters separable without donor/time-point batch structure dominating, indicating CCA integration preceded clustering

## Limitations

- Downstream scCrossTalk is reported as compatible with Seurat 5; clustering output from older Seurat versions may need conversion
- No changelog was found for the source repository, so version-specific behavior of the pipeline is not documented
- Batch effects across 7 donors and two perfusion time points must be removed via CCA integration before clustering, otherwise clusters confound donor identity with cell type

## Evidence

- [other] Perform PCA on the top variable features, embed cells with UMAP, and cluster cells (Louvain/Leiden on the k-nearest-neighbor graph).: [evidence span withheld — anchor status: not_found]
