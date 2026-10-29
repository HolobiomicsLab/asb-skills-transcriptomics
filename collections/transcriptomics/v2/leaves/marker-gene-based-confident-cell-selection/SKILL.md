---
name: marker-gene-based-confident-cell-selection
description: Use when building a labeled reference scRNA-seq training set for supervised
  TME cell subtype annotation and you need to filter cells down to those confidently
  belonging to T.cells, Myeloid.cells, B.cells, Fibroblast, or Endothelial compartments.
when_to_use_negative:
- Input is spatial transcriptomics data or the goal is malignant vs non-malignant
  cell identification — scCancer2 handles those with a pretrained xgboost model (threshold
  0.5) instead of marker-based confident selection
- The data already carries curated fine-grained subtype labels — re-selecting confident
  cells by marker scoring would discard existing annotation
- The tissue is not a tumor microenvironment — the default compartment list only covers
  T.cells, Myeloid.cells, B.cells, Fibroblast, and Endothelial
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scCancer2
  role: Supplies predCellType for rough labels, the packaged marker files under system.file("txt"),
    and runCellSubtypeClassify for multinomial subtype classification
  repo: http://lifeome.net/software/sccancer2/
- name: scCancer
  role: R package whose packaged txt marker files and cellSubtypeTemplates.rds back
    the marker-based confident cell selection
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Provides the Seurat object whose RNA assay data is passed to predCellType
    and runCellSubtypeClassify
- name: dplyr
  role: Used with gsub to merge CD4/CD8 T-cell labels into a single T.cells class
    and intersect labels with the five default compartments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:671b48459ae0a1a62fcda8e0d9747d239fef2ce477c04a9dd9b1093e5a264701
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# marker-gene-based-confident-cell-selection

## Summary

Selects high-confidence cells per TME compartment by scoring single-cell expression against curated marker gene files (packaged under system.file("txt", package = "scCancer")), yielding trustworthy training labels for the multinomial subtype classifiers in scCancer2.

## When to use

Use when building a labeled reference scRNA-seq training set for supervised TME cell subtype annotation and you need to filter cells down to those confidently belonging to T.cells, Myeloid.cells, B.cells, Fibroblast, or Endothelial compartments.

## When NOT to use

- Input is spatial transcriptomics data or the goal is malignant vs non-malignant cell identification — scCancer2 handles those with a pretrained xgboost model (threshold 0.5) instead of marker-based confident selection
- The data already carries curated fine-grained subtype labels — re-selecting confident cells by marker scoring would discard existing annotation
- The tissue is not a tumor microenvironment — the default compartment list only covers T.cells, Myeloid.cells, B.cells, Fibroblast, and Endothelial

## Inputs

- Seurat object with normalized RNA assay (LogNormalize, scale factor 10000)
- Rough cell-type predictions from predCellType
- Packaged marker gene files under system.file("txt", package = "scCancer")
- Packaged subtype submodel templates (cellSubtypeTemplates.rds)

## Outputs

- High-confidence cells per TME compartment for classifier training
- Fine-grained cell subtype labels saved to fine-labels.RDS
- Subtype similarity matrix saved to similarity-matrix.RDS
- Per-dataset cross-validated accuracy and kappa statistics

## How to apply

First derive rough labels with predCellType on the RNA assay data of the Seurat object, merging CD4/CD8 T-cell labels into a single T.cells class via gsub. Restrict annotation to the five TME compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) by intersecting predicted labels with the default compartment list. Then score cells per compartment against the marker files packaged under system.file("txt") and retain only high-confidence cells, using entropy and highly-representative-gene criteria to select discriminative features per compartment. These curated cells and features become the training input for runCellSubtypeClassify with cellSubtypeTemplates.rds as submodel.path, dropout.modeling = FALSE and unknown.cutoff = 0.3, and classifier quality is validated by 5-fold cross-validated accuracy and kappa.

## Related tools

- **scCancer2** (Supplies predCellType for rough labels, the packaged marker files under system.file("txt"), and runCellSubtypeClassify for multinomial subtype classification) — http://lifeome.net/software/sccancer2/
- **scCancer** (R package whose packaged txt marker files and cellSubtypeTemplates.rds back the marker-based confident cell selection) — https://github.com/czythu/scCancer
- **Seurat** (Provides the Seurat object whose RNA assay data is passed to predCellType and runCellSubtypeClassify)
- **dplyr** (Used with gsub to merge CD4/CD8 T-cell labels into a single T.cells class and intersect labels with the five default compartments)

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3)
```

## Evaluation signals

- 5-fold cross-validation within each of the 15 reference datasets (594 samples, 1,213,469 cells) yields mean accuracy and Cohen's kappa above the aggregated per-dataset variance summaries reported for the package
- Every retained training cell maps to one of the five TME compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) after intersecting predCellType output with the default list
- Cells assigned 'unknown' are controlled by unknown.cutoff = 0.3 in runCellSubtypeClassify; classify with dropout.modeling = FALSE
- Per-dataset outputs contain a fine-labels.RDS and a similarity-matrix.RDS in the per-dataset annotation folder
- Discriminative features selected per compartment satisfy entropy and highly-representative-gene criteria

## Limitations

- CD4 and CD8 T-cell labels are merged into a single T.cells class, so the five-compartment model cannot resolve those subtypes without the fine subtype classifiers
- No repository URL or accession IDs were provided for the HCC reference datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, Xue2022), complicating reuse of the reference labels
- No software version numbers stated for R, scCancer, Seurat, or xgboost, so exact reproducibility of the marker-scoring behavior is not guaranteed

## Evidence

- [methods] marker files resolved from packaged txt directory: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] rough labeling with predCellType on RNA assay data: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [methods] fine subtype classification call consuming markers.path and submodel.path: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path"
- [abstract] cross-dataset classifier evaluation grounding the cross-validated accuracy/kappa signals: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [readme] package scope statement from repository README: [evidence span withheld — anchor status: external_doc]
