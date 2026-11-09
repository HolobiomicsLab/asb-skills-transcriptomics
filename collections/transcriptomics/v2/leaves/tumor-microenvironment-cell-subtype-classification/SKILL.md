---
name: tumor-microenvironment-cell-subtype-classification
description: Use when a processed tumor scRNA-seq Seurat object (LogNormalize, 5000
  vst features) needs major-type and fine subtype labels, or when subtype labels from
  different published references must be compared for similarity.
when_to_use_negative:
- Input is raw droplet counts with ambient-RNA contamination — preprocess and correct
  first (e.g. SoupX, min.genes = 200 QC) before annotation
- The goal is malignant vs non-malignant cell identification — use scCancer2's separate
  xgboost classifier trained on TCGA bulk + scRNA-seq data instead
- Data are spatial transcriptomics spots rather than single-cell profiles — use scCancer2's
  spatial transcriptome module
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scCancer2
  role: Provides predCellType, runCellSubtypeClassify, and similarityCalculation for
    two-stage TME annotation and cross-dataset label similarity
  repo: https://doi.org/10.5281/zenodo.10477296
- name: scCancer
  role: R toolkit loaded alongside dplyr in the annotation workflow
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Object handling, normalization, and FindTransferAnchors/TransferData for subtype
    label transfer from the PBMC reference
- name: dplyr
  role: Data manipulation when merging/intersecting level-1 cell type labels
- name: xgboost
  role: Backend for the lightweight machine-learning sub-models and the malignant
    cell classifier
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:2779dc7754edd9910fca1dc6bbf3df0a75c0d473d98ecbeafb8af354100d30cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  - xgboost
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8208955223880597
n_claims: 67
---

# tumor-microenvironment-cell-subtype-classification

## Summary

Two-stage TME cell annotation from scCancer2: a rough level-1 label via predCellType, followed by fine subtype assignment with runCellSubtypeClassify using pre-trained lightweight machine-learning sub-models, plus cross-reference label similarity quantification via similarityCalculation.

## When to use

Use when a processed tumor scRNA-seq Seurat object (LogNormalize, 5000 vst features) needs major-type and fine subtype labels, or when subtype labels from different published references must be compared for similarity.

## When NOT to use

- Input is raw droplet counts with ambient-RNA contamination — preprocess and correct first (e.g. SoupX, min.genes = 200 QC) before annotation
- The goal is malignant vs non-malignant cell identification — use scCancer2's separate xgboost classifier trained on TCGA bulk + scRNA-seq data instead
- Data are spatial transcriptomics spots rather than single-cell profiles — use scCancer2's spatial transcriptome module

## Inputs

- Preprocessed Seurat object (.RDS) with normalized RNA assay (LogNormalize, scale.factor 10000)
- cellSubtypeTemplates.rds pre-trained sub-models and markers files shipped with scCancer2
- External reference Seurat object (e.g. Hao et al. 2021 PBMC multi-modal reference) for label transfer
- Per-sample fine-labels.RDS files for cross-dataset integration

## Outputs

- Per-cell level-1 TME labels (e.g. T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial)
- Per-sample fine-labels.RDS with cell subtype assignments and unknown labels (unknown.cutoff = 0.3)
- similarity-matrix.RDS and integrated-similaritymatrix.RDS quantifying label similarity across references
- Integrated-finelabels.RDS combining all samples

## How to apply

Start from a normalized Seurat RNA assay (LogNormalize, scale factor 10000; variable features = min(5000, nrow); ScaleData regressing nCount_RNA and mito.percent). Run predCellType on the normalized RNA data to get level-1 labels (e.g. T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial), merging T.cells.CD4 and T.cells.CD8 into T.cells. Then run runCellSubtypeClassify with cellSubtypeTemplates.rds sub-models and marker files, dropout.modeling = FALSE, unknown.cutoff = 0.3, umap.plot = FALSE, saving per-sample fine-labels.RDS and similarity-matrix.RDS. Optionally transfer fine subtypes from an external reference such as the Hao et al. (2021) PBMC multi-modal reference using Seurat FindTransferAnchors/TransferData. To compare subtype labels across datasets, rbind per-sample fine labels into integrated-finelabels.RDS and run similarityCalculation to produce an integrated similarity matrix grounded in the trained classifiers across 15 scRNA-seq datasets (594 samples).

## Related tools

- **scCancer2** (Provides predCellType, runCellSubtypeClassify, and similarityCalculation for two-stage TME annotation and cross-dataset label similarity) — https://doi.org/10.5281/zenodo.10477296
- **scCancer** (R toolkit loaded alongside dplyr in the annotation workflow) — https://github.com/czythu/scCancer
- **Seurat** (Object handling, normalization, and FindTransferAnchors/TransferData for subtype label transfer from the PBMC reference)
- **dplyr** (Data manipulation when merging/intersecting level-1 cell type labels)
- **xgboost** (Backend for the lightweight machine-learning sub-models and the malignant cell classifier)

## Examples

```
t.results <- predCellType(expr@assays[["RNA"]]@data); t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3, umap.plot = FALSE)
```

## Evaluation signals

- Per-sample fine-labels.RDS and similarity-matrix.RDS exist and load as valid R objects after runCellSubtypeClassify
- Rough labels are restricted to the expected level-1 set ('T.cells', 'Myeloid.cells', 'B.cells', 'Fibroblast', 'Endothelial') after merging T.cells.CD4/CD8
- Cells with subtype classifier similarity below unknown.cutoff = 0.3 are labeled unknown rather than forced into a subtype
- Integrated similarity matrix reproduces expected cross-reference relationships (e.g. plasmablast, memory B cell, dendritic cell subtypes consistent with Hao et al. 2021 PBMC reference and Fig. 5E)
- Downstream compositions are biologically coherent — e.g. TLS spots enriched for B cells, dendritic cells, monocytes, and T cells

## Limitations

- No software version numbers stated for R, scCancer, Seurat, or xgboost, so exact reproducibility may vary
- No repository URL or accession IDs provided for the HCC scRNA-seq datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, Xue2022)
- dropout.modeling was disabled (dropout.modeling = FALSE) and umap.plot = FALSE in the reproduced workflow, so dropout-aware subtype scoring is not exercised
- Fine subtype labels depend on the reference definitions; the method quantifies cross-reference label similarity precisely because subtypes defined in different published datasets do not map one-to-one

## Evidence

- [methods] Rough label annotation with predCellType: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [methods] Fine label annotation with runCellSubtypeClassify: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path"
- [abstract] Similarity maps across references: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [abstract] Supervised TME annotation scope: "In this updated package scCancer2, designed for integrative tumor scRNA-seq data analysis, we developed a supervised machine learning framework to annotate TME cells"
- [methods] Normalization parameters: "object <- NormalizeData(object = object,
                        normalization.method = "LogNormalize",
                        scale.factor = 10000,"
- [readme] Package purpose: [evidence span withheld — anchor status: external_doc]
