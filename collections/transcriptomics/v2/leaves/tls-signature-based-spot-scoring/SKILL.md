---
name: tls-signature-based-spot-scoring
description: Use when spatial transcriptomics spots contain detected TLS structures
  and you need to know which immune cell subtypes (e.g. plasmablasts, memory B cells,
  dendritic cells) compose them.
when_to_use_negative:
- Input is bulk RNA-seq or single-cell data without spatial spot-level coordinates
  — TLS spot scoring requires spatially resolved spots
- The goal is malignant vs non-malignant cell identification, which scCancer2 handles
  with the pretrained xgboost model (threshold 0.5), not TLS composition mapping
- Data has no plausible immune compartment (e.g. homogeneous cell lines) where marker-based
  subtype classifiers cannot be estimated
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scCancer2
  role: Spatial transcriptomics module detecting TLS spots and running cell subtype
    annotation/similarity workflows
  repo: https://github.com/czythu/scCancer
- name: scCancer
  role: Base R toolkit providing predCellType, runCellSubtypeClassify, and similarityCalculation
    functions
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: FindTransferAnchors/TransferData to transfer PBMC reference subtype annotations
    onto fine labels
- name: dplyr
  role: Data manipulation when merging cell type labels (e.g. T.cells.CD4 + T.cells.CD8
    -> T.cells)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:35c96c88a790a0adbe988642bc3bb49a743059ca2a234b1c73d46c22f30250cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# tls-signature-based-spot-scoring

## Summary

Scores and annotates tertiary lymphoid structure (TLS) spots in spatial transcriptomics data using scCancer2's spatial module, then maps cell subtype annotations (from predCellType/runCellSubtypeClassify fine labels and a Hao et al. 2021 PBMC reference) onto TLS spots to quantify immune-cell enrichment (B cells, dendritic cells, monocytes, T cells).

## When to use

Use when spatial transcriptomics spots contain detected TLS structures and you need to know which immune cell subtypes (e.g. plasmablasts, memory B cells, dendritic cells) compose them.

## When NOT to use

- Input is bulk RNA-seq or single-cell data without spatial spot-level coordinates — TLS spot scoring requires spatially resolved spots
- The goal is malignant vs non-malignant cell identification, which scCancer2 handles with the pretrained xgboost model (threshold 0.5), not TLS composition mapping
- Data has no plausible immune compartment (e.g. homogeneous cell lines) where marker-based subtype classifiers cannot be estimated

## Inputs

- Preprocessed Seurat expression objects (.RDS) per sample, with normalized RNA assay and spatial spot coordinates
- cellSubtypeTemplates.rds sub-models and marker files from scCancer2
- Hao et al. (2021) multimodal PBMC reference Seurat object
- Spatial transcriptomics samples from BRCA, CRC, and HCC (e.g. Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, Xue2022 chunks 1-4)

## Outputs

- Per-sample fine-labels.RDS with subtype annotations per cell/spot
- Per-sample similarity-matrix.RDS
- integrated-finelabels.RDS (rbind across all authors and Xue2022 chunks)
- integrated-similaritymatrix.RDS from similarityCalculation
- TLS spot composition: enriched immune cell types (B cells, dendritic cells, monocytes, T cells) and most abundant subtypes (plasmablasts, memory B cells, dendritic cells)

## How to apply

Load preprocessed Seurat objects (e.g. Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, plus the four Xue2022 chunks) from RDS files, run predCellType on the normalized RNA assay for level-1 labels, merge T.cells.CD4/T.cells.CD8 into T.cells, and intersect with the default list ('T.cells', 'Myeloid.cells', 'B.cells', 'Fibroblast', 'Endothelial'). For each sample run runCellSubtypeClassify with the cellSubtypeTemplates.rds sub-models, dropout.modeling = FALSE, unknown.cutoff = 0.3, and umap.plot = FALSE, saving fine-labels.RDS and similarity-matrix.RDS per sample. Transfer subtype annotations from the Hao et al. (2021) multimodal PBMC reference with Seurat FindTransferAnchors/TransferData to resolve subtypes such as plasmablasts, memory B cells, and dendritic cells. Finally rbind per-sample fine labels across authors and Xue2022 chunks into integrated-finelabels.RDS, run similarityCalculation to produce integrated-similaritymatrix.RDS, and map subtypes onto detected TLS spots.

## Related tools

- **scCancer2** (Spatial transcriptomics module detecting TLS spots and running cell subtype annotation/similarity workflows) — https://github.com/czythu/scCancer
- **scCancer** (Base R toolkit providing predCellType, runCellSubtypeClassify, and similarityCalculation functions) — https://github.com/czythu/scCancer
- **Seurat** (FindTransferAnchors/TransferData to transfer PBMC reference subtype annotations onto fine labels)
- **dplyr** (Data manipulation when merging cell type labels (e.g. T.cells.CD4 + T.cells.CD8 -> T.cells))

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, unknown.cutoff = 0.3, dropout.modeling = FALSE, umap.plot = FALSE)
```

## Evaluation signals

- TLS spots show the expected enriched composition — B cells, dendritic cells, monocytes, and T cells — consistent with Fig. 5E
- Most abundant subtypes within TLS spots are plasmablasts, memory B cells, and dendritic cells
- Rough level-1 labels intersect with the default list ('T.cells', 'Myeloid.cells', 'B.cells', 'Fibroblast', 'Endothelial') after merging T.cells.CD4/T.cells.CD8
- integrated-similaritymatrix.RDS is produced by similarityCalculation from the rbind of all per-sample fine labels
- Cells with subtype similarity below unknown.cutoff = 0.3 are labeled unknown rather than forced into a subtype

## Limitations

- No software version numbers stated for R, scCancer, Seurat, or xgboost, complicating exact reproduction
- No repository URL or accession IDs provided for the HCC datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021), so source data must be located independently
- unknown.cutoff = 0.3 forces subtypes below the similarity threshold to unknown, which may leave TLS spots partially unannotated

## Evidence

- [abstract] scCancer2 spatial module for TLS processing: "Thirdly, scCancer2 integrated a module to process the spatial transcriptomic data and analyze the spatial features of TME."
- [methods] runCellSubtypeClassify call with cutoffs: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,"
- [methods] unknown.cutoff parameter: "unknown.cutoff = 0.3,"
- [methods] predCellType rough labeling: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [readme] similarity calculation across datasets: [evidence span withheld — anchor status: external_doc]
