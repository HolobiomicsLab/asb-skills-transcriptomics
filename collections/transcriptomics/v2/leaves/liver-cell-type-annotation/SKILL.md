---
name: liver-cell-type-annotation
description: Use when a processed Seurat object of human liver scRNA-seq data (e.g.
  transplanted liver samples, cold perfusion/portal reperfusion stages) has PCA/UMAP
  clusters but no cell-type labels and needs annotation compatible with scCrossTalk's
  sc_celltype input.
when_to_use_negative:
- Input data are not human liver scRNA-seq counts — the reference populations (MAIT
  cells, S100A12+ neutrophils, etc.) are liver-transplant-specific and will not map
  to other tissues
- Cells are already labeled with cell types — re-annotation only adds noise before
  scCrossTalk communication inference
- Only raw unnormalized counts are available without any clustering/dimensional reduction
  performed — cluster-level annotation requires the Seurat UMAP/graph step first
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Builds per-sample objects, LogNormalize normalization (scale factor 10,000),
    CCA-based integration, PCA/UMAP and clustering whose clusters receive the cell-type
    labels
- name: scCrossTalk
  role: Consumer of the annotated cell-type vector (sc_celltype argument of create_scCrossTalk)
    for downstream ligand-repair cell-cell communication inference
  repo: https://github.com/multitalk/scCrossTalk
- name: devtools
  role: Installs scCrossTalk via devtools::install_github('multitalk/scCrossTalk')
    or devtools::install_local()
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Execution environment for the Seurat annotation pipeline and scCrossTalk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:c15ba8cbd814efc8f9d1e3b3295d630909a809baa4d7856afca4511421cd586c
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# liver-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate clusters from an integrated human liver transplant single-cell dataset (12 samples, 75,231 cells) into hepatic parenchymal and non-parenchymal populations (hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) so the labels can feed downstream scCrossTalk cell-cell communication inference.

## When to use

Use when a processed Seurat object of human liver scRNA-seq data (e.g. transplanted liver samples, cold perfusion/portal reperfusion stages) has PCA/UMAP clusters but no cell-type labels and needs annotation compatible with scCrossTalk's sc_celltype input.

## When NOT to use

- Input data are not human liver scRNA-seq counts — the reference populations (MAIT cells, S100A12+ neutrophils, etc.) are liver-transplant-specific and will not map to other tissues
- Cells are already labeled with cell types — re-annotation only adds noise before scCrossTalk communication inference
- Only raw unnormalized counts are available without any clustering/dimensional reduction performed — cluster-level annotation requires the Seurat UMAP/graph step first

## Inputs

- Raw Cell Ranger/GRCh38 count matrices from 12 human liver transplant samples (cold preservation and two-hour portal perfusion stages)
- Integrated Seurat object with PCA reduction, UMAP embedding, and Louvain/Leiden cluster assignments
- Marker gene expression matrix for cluster label assignment

## Outputs

- Cell-type annotation vector per cell (hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) in scCrossTalk celltype format
- Annotated Seurat object saved as liver_seurat_object.rds (75,231 cells total)

## How to apply

After building Seurat objects per sample from Cell Ranger/GRCh38 counts and normalizing with LogNormalize (scale factor 10,000), integrate the samples via Seurat's CCA-based workflow (FindIntegrationAnchors on PCA-reduced features) to remove donor/time-point batch effects, then run PCA, UMAP embedding, and Louvain/Leiden clustering on the k-nearest-neighbor graph. Assign biological labels to each cluster using canonical marker expression consistent with the paper's populations: hepatocytes (parenchymal), endothelial cells, mucosal-associated invariant T (MAIT) cells, GZMB+ GZMK+ natural killer cells, and S100A12+ neutrophils (non-parenchymal). Format the labels as a character vector keyed to cells so they can be passed as the sc_celltype argument to create_scCrossTalk(sc_data, sc_celltype, species). Validate by confirming the integrated object retains the reported 75,231 cells across 12 samples from 7 donors (three non-EAD, four EAD) before saving as liver_seurat_object.rds.

## Related tools

- **Seurat** (Builds per-sample objects, LogNormalize normalization (scale factor 10,000), CCA-based integration, PCA/UMAP and clustering whose clusters receive the cell-type labels)
- **scCrossTalk** (Consumer of the annotated cell-type vector (sc_celltype argument of create_scCrossTalk) for downstream ligand-repair cell-cell communication inference) — https://github.com/multitalk/scCrossTalk
- **devtools** (Installs scCrossTalk via devtools::install_github('multitalk/scCrossTalk') or devtools::install_local()) — https://github.com/multitalk/scCrossTalk
- **R** (Execution environment for the Seurat annotation pipeline and scCrossTalk)

## Examples

```
obj <- create_scCrossTalk(sc_data, sc_celltype, species)
```

## Evaluation signals

- Integrated Seurat object contains exactly 75,231 cells from 12 samples / 7 donors (three non-EAD, four EAD) before annotation
- Every cluster maps to one of the paper's reported populations (hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) with no unassigned clusters
- Cell-type vector length equals the number of cells and names match the Seurat object cell names (required by create_scCrossTalk)
- create_scCrossTalk(sc_data, sc_celltype, species) accepts the annotation and returns an object of class scCrossTalk without celltype errors
- Annotated object saves/loads cleanly as liver_seurat_object.rds

## Limitations

- Annotation vocabulary must match scCrossTalk's celltype input format; inconsistent labels will break downstream LR-pair inference
- The reference populations come from liver transplant samples at cold preservation and portal perfusion time points; the label set may be incomplete for other liver contexts
- Batch effects across donors and time points must be removed first (CCA integration) or cluster identities will be confounded by donor variation

## Evidence

- [methods] workflow task_001 pipeline: [evidence span withheld — anchor status: not_found]
