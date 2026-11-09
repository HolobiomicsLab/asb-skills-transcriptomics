---
name: cell-subtype-supervised-classification
description: Use when a Seurat scRNA-seq object from tumor tissue needs fine-grained
  cell subtype labels within the five TME compartments (T.cells, Myeloid.cells, B.cells,
  Fibroblast, Endothelial), or when cross-dataset subtype label similarity must be
  quantified.
when_to_use_negative:
- The goal is de novo clustering/discovery of novel cell states rather than assigning
  cells to the five predefined TME compartments with known subtypes
- The data are spatial transcriptomics spots or bulk RNA-seq samples — use scCancer2's
  spatial module or the xgboost malignant-cell classifier instead
- No compatible reference templates or marker files exist for the tissue/cancer type,
  so the supervised classifiers cannot be trusted without retraining
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scCancer2
  role: Provides predCellType, runCellSubtypeClassify, and similarityCalculation for
    rough labeling, fine subtype classification, and cross-dataset similarity mapping
  repo: https://doi.org/10.5281/zenodo.10477296
- name: scCancer
  role: R package supplying pretrained subtype templates (cellSubtypeTemplates.rds)
    and marker files in its rds and txt directories
  repo: https://doi.org/10.5281/zenodo.10477296
- name: Seurat
  role: Data structure for loading and manipulating the scRNA-seq objects before and
    after annotation
- name: dplyr
  role: Data manipulation during label merging and preprocessing
- name: R
  role: Execution environment for the scCancer2 workflow
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:12b4fa7219ba227e8fad65872c7b1dad4e3bca1b82333110a2fdaa41fcaacf47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# cell-subtype-supervised-classification

## Summary

A supervised machine-learning framework in scCancer2 that annotates tumor microenvironment (TME) cells to fine subtypes by training multinomial classifiers per compartment across 15 scRNA-seq datasets (594 samples, 1,213,469 cells), and supports cross-dataset label similarity maps.

## When to use

Use when a Seurat scRNA-seq object from tumor tissue needs fine-grained cell subtype labels within the five TME compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial), or when cross-dataset subtype label similarity must be quantified.

## When NOT to use

- The goal is de novo clustering/discovery of novel cell states rather than assigning cells to the five predefined TME compartments with known subtypes
- The data are spatial transcriptomics spots or bulk RNA-seq samples — use scCancer2's spatial module or the xgboost malignant-cell classifier instead
- No compatible reference templates or marker files exist for the tissue/cancer type, so the supervised classifiers cannot be trusted without retraining

## Inputs

- Seurat single-cell RNA-seq objects (e.g. Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, and four partitions of Xue2022)
- Pretrained subtype template models (cellSubtypeTemplates.rds from the scCancer package rds directory)
- Marker gene files packaged under system.file('txt', package='scCancer')

## Outputs

- Per-dataset fine subtype labels (fine-labels.RDS)
- Per-dataset subtype similarity matrices (similarity-matrix.RDS)
- Integrated fine labels across datasets (integrated-finelabels.RDS)
- Cross-reference Jaccard similarity matrix (integrated-similaritymatrix.RDS)
- 5-fold cross-validation classification accuracy and Cohen's kappa statistics

## How to apply

First derive rough labels by running predCellType on the RNA assay data (expr@assays[['RNA']]@data) and merge T.cells.CD4/T.cells.CD8 into a single T.cells class via gsub. Restrict annotation to the five default TME compartments, then train multinomial subtype classifiers by weighted maximum likelihood with runCellSubtypeClassify, using cellSubtypeTemplates.rds (from the scCancer package rds directory) as submodel.path, packaged marker files under system.file('txt') as markers.path, dropout.modeling = FALSE, and unknown.cutoff = 0.3. Select high-confidence cells per compartment by marker scoring and discriminative features per compartment using entropy and highly-representative-gene criteria. Save per-dataset outputs as fine-labels.RDS and similarity-matrix.RDS; merge fine labels across datasets by rbind into integrated-finelabels.RDS and run similarityCalculation to build the cross-reference Jaccard similarity matrix (integrated-similaritymatrix.RDS). Performance is validated via 5-fold cross-validation computing classification accuracy and Cohen's kappa per held-out fold.

## Related tools

- **scCancer2** (Provides predCellType, runCellSubtypeClassify, and similarityCalculation for rough labeling, fine subtype classification, and cross-dataset similarity mapping) — https://doi.org/10.5281/zenodo.10477296
- **scCancer** (R package supplying pretrained subtype templates (cellSubtypeTemplates.rds) and marker files in its rds and txt directories) — https://doi.org/10.5281/zenodo.10477296
- **Seurat** (Data structure for loading and manipulating the scRNA-seq objects before and after annotation)
- **dplyr** (Data manipulation during label merging and preprocessing)
- **R** (Execution environment for the scCancer2 workflow)

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3)
```

## Evaluation signals

- 5-fold cross-validation within each reference dataset yields per-fold classification accuracy and Cohen's kappa; aggregate per-dataset mean and variance summaries should be consistent with the paper's evaluation on 15 datasets (594 samples, 1,213,469 cells)
- The Lymphoid-B row of the integrated Jaccard similarity matrix must show low similarity to naive B cells (0.10 NSCLC, 0.17 BRCA) and high similarity to memory B cells (0.72 NSCLC, 0.67 BRCA), matching Section 3.3 / Fig. 3A
- Predicted rough labels intersected with the default compartment list must contain only T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial
- Output files fine-labels.RDS and similarity-matrix.RDS must exist in each per-dataset output folder, and integrated-finelabels.RDS / integrated-similaritymatrix.RDS must be produced after rbind and similarityCalculation

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost, so exact reproducibility may require environment probing
- No repository URL or accession IDs are provided for the HCC datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021), so inputs must be sourced separately
- The unknown.cutoff = 0.3 parameter discards low-confidence assignments; cells below classifier confidence remain unlabeled rather than force-assigned

## Evidence

- [methods] Rough and fine label annotation: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [methods] Fine subtype classification call: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path"
- [methods] unknown cutoff threshold: "unknown.cutoff = 0.3,"
- [methods] similarity calculation: "all.matrix <- similarityCalculation(all.list, annoPath)"
- [abstract] classifier-based similarity maps across references: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [readme] README on scCancer2 subtype annotation scope: [evidence span withheld — anchor status: external_doc]
