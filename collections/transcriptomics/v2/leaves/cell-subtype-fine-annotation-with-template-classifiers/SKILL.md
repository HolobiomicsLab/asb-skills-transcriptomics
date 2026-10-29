---
name: cell-subtype-fine-annotation-with-template-classifiers
description: Use when a normalized single-cell tumor scRNA-seq expression matrix needs
  cell subtype labels beyond the five major TME types, using pretrained subtype templates
  rather than reference-free clustering.
when_to_use_negative:
- Input is raw (unnormalized) count data — predCellType operates on the LogNormalize-normalized
  RNA @data matrix, not raw counts
- Non-tumor scRNA-seq data or cell types outside the TME major types covered by the
  package-shipped templates (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial)
- Spatial transcriptomic spots requiring the scCancer2 spatial module rather than
  single-cell subtype templates
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scCancer2
  role: Main package providing predCellType for rough TME label assignment, runCellSubtypeClassify
    for template-based fine subtype classification, and similarityCalculation for
    cross-reference label similarity maps
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Holds input data as Seurat objects; normalization via NormalizeData (LogNormalize,
    scale.factor 10000) and data access via the RNA assay @data slot
- name: R
  role: Execution environment; used with dplyr for label merging (e.g., gsub of T.cells.CD4/T.cells.CD8
    into T.cells)
- name: xgboost
  role: Underlying classifier engine for the supervised machine-learning framework
    (also used for malignant cell identification with a 0.5 prediction threshold)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:fd4a81ca8d9f6bfd971ee3a5df608ca0efa6df7df6a4a36affa834c1930b2046
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - CHETAH
  - OCLR
  - R
  - Scibet
  - Seurat
  - SingleCellNet
  - dplyr
  - scCancer
  - scCancer2
  - scPred
  - scmap
  - xgboost
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# cell-subtype-fine-annotation-with-template-classifiers

## Summary

Fine-grained tumor microenvironment cell subtype annotation using scCancer2's two-stage supervised framework: a rough label from predCellType (e.g., five major cell types: T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) followed by runCellSubtypeClassify, which trains lightweight machine-learning classifiers on package-shipped cellSubtypeTemplates.rds subtype templates and marker files per major type.

## When to use

Use when a normalized single-cell tumor scRNA-seq expression matrix needs cell subtype labels beyond the five major TME types, using pretrained subtype templates rather than reference-free clustering.

## When NOT to use

- Input is raw (unnormalized) count data — predCellType operates on the LogNormalize-normalized RNA @data matrix, not raw counts
- Non-tumor scRNA-seq data or cell types outside the TME major types covered by the package-shipped templates (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial)
- Spatial transcriptomic spots requiring the scCancer2 spatial module rather than single-cell subtype templates

## Inputs

- Normalized scRNA-seq expression matrix in a Seurat object (RNA assay @data slot)
- Per-dataset .RDS Seurat object files (e.g., per-study author-labeled files such as Ma2022.RDS, Zhang2019.RDS)
- Package-shipped cellSubtypeTemplates.rds subtype template files and marker files

## Outputs

- fine-labels.RDS: per-cell fine subtype label assignments per dataset
- similarity-matrix.RDS: classifier-based similarity matrix between cell subtypes across references
- Per-cell rough labels stored in the Seurat metadata column Cell.Type

## How to apply

First load each dataset as a Seurat object and run scCancer2's predCellType on the normalized RNA assay (@assays[['RNA']]@data) to assign rough level-1 labels; merge 'T.cells.CD4' and 'T.cells.CD8' into a unified 'T.cells' with gsub and intersect predictions with the five-type default list (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial). Then run runCellSubtypeClassify per major cell type with dropout.modeling = FALSE and unknown.cutoff = 0.3, pointing submodel.path and markers.path at the package-shipped cellSubtypeTemplates.rds templates and marker files. Rationale: the templates encode subtype definitions learned from 15 published scRNA-seq cell atlases (594 samples) with original labelings preserved, so annotation is classifier-based rather than de novo. Save per-dataset outputs (fine-labels.RDS and similarity-matrix.RDS) and, optionally, use similarityCalculation on all label lists to build quantitative cross-reference similarity maps between subtype labels.

## Related tools

- **scCancer2** (Main package providing predCellType for rough TME label assignment, runCellSubtypeClassify for template-based fine subtype classification, and similarityCalculation for cross-reference label similarity maps) — https://github.com/czythu/scCancer
- **Seurat** (Holds input data as Seurat objects; normalization via NormalizeData (LogNormalize, scale.factor 10000) and data access via the RNA assay @data slot)
- **R** (Execution environment; used with dplyr for label merging (e.g., gsub of T.cells.CD4/T.cells.CD8 into T.cells))
- **xgboost** (Underlying classifier engine for the supervised machine-learning framework (also used for malignant cell identification with a 0.5 prediction threshold))

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3)
```

## Evaluation signals

- Predicted level-1 types intersect with the five-type default list (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) — labels outside this list indicate misconfigured label merging
- Both fine-labels.RDS and similarity-matrix.RDS are written per dataset as expected outputs of runCellSubtypeClassify
- unknown.cutoff = 0.3 is used so low-similarity cells are labeled 'unknown' rather than forced into a subtype
- In benchmark use, predicted labels compared against level-1 metadata reference labels yield per-dataset accuracies consistent with the reported Fig. 2B ordering
- Cross-dataset similarityCalculation produces classifier-based similarity maps reproducible across all 15 reference datasets

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost, and no repository URL/accession IDs are provided for the HCC scRNA-seq benchmark datasets
- Fine annotation depends on the shipped cellSubtypeTemplates.rds templates; subtypes absent from the 15 curated references cannot be assigned (and fall into 'unknown' under the 0.3 cutoff)
- Dropout modeling is disabled in the reported workflow (dropout.modeling = FALSE), which may reduce sensitivity in sparse datasets

## Evidence

- [methods] predCellType on normalized data is the rough-label stage: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [methods] runCellSubtypeClassify with unknown.cutoff 0.3 performs template-based fine annotation: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path
"
- [methods] unknown.cutoff parameter value: "unknown.cutoff = 0.3,"
- [readme] README describes the annotation-by-classifiers approach underlying this skill: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract frames the supervised ML framework for TME cell annotation in scCancer2: "In this updated package scCancer2, designed for integrative tumor scRNA-seq data analysis, we developed a supervised machine learning framework to annotate TME cells"
