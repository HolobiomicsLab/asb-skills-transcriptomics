---
name: cross-dataset-cell-label-integration
description: Use when per-dataset cell subtype fine labels have been produced (fine-labels.RDS
  per dataset) and you need to reconcile or compare subtype label definitions across
  published references by constructing a cross-dataset Jaccard similarity matrix.
when_to_use_negative:
- You only need rough cell-type labels for a single dataset — predCellType alone suffices
  and the cross-dataset label integration is unnecessary.
- Your data are spatial transcriptomics or bulk TCGA expression — the label similarity
  maps were constructed by testing the trained subtype classifiers on the 15 scRNA-seq
  datasets, not on spatial or bulk inputs.
- You want de novo clustering-based cell identification rather than preserving original
  published label definitions across references.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCancer
  role: Core R package providing predCellType, runCellSubtypeClassify, and similarityCalculation
    used to produce rough labels, fine subtype labels, and cross-dataset Jaccard similarity
    matrices
  repo: czythu/scCancer
- name: scCancer2
  role: Updated package version that introduces supervised machine learning subtype
    annotation and quantitative cross-dataset label similarity maps
- name: Seurat
  role: Seurat objects store the per-dataset scRNA-seq expression (RNA assay @data)
    that predCellType consumes
- name: dplyr
  role: Loaded alongside scCancer in the R analysis scripts for label/data manipulation
- name: R
  role: Execution environment for the integration and similarity-calculation scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:d72ccd65e0ea8c359218ef36180d8f9402ce718261df4a518d4ff446192d97ad
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-dataset-cell-label-integration

## Summary

A scCancer2-based workflow that integrates fine-grained cell subtype labels across published scRNA-seq datasets (HCC cohorts Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021 and four Xue2022 partitions) and quantitatively builds Jaccard similarity maps between subtype labels defined in different references (e.g., NSCLC and BRCA vs. HCC 'Lymphoid-B').

## When to use

Use when per-dataset cell subtype fine labels have been produced (fine-labels.RDS per dataset) and you need to reconcile or compare subtype label definitions across published references by constructing a cross-dataset Jaccard similarity matrix.

## When NOT to use

- You only need rough cell-type labels for a single dataset — predCellType alone suffices and the cross-dataset label integration is unnecessary.
- Your data are spatial transcriptomics or bulk TCGA expression — the label similarity maps were constructed by testing the trained subtype classifiers on the 15 scRNA-seq datasets, not on spatial or bulk inputs.
- You want de novo clustering-based cell identification rather than preserving original published label definitions across references.

## Inputs

- Seurat objects (.RDS) for each dataset (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, Xue2022 partitions)
- Pretrained subtype template models (cellSubtypeTemplates.rds from scCancer rds directory)
- Marker gene files from the scCancer package txt directory
- Per-dataset fine-label files (fine-labels.RDS) for row-binding

## Outputs

- Per-dataset fine-label annotations (fine-labels.RDS) and per-dataset similarity matrices (similarity-matrix.RDS)
- Integrated fine labels across datasets (integrated-finelabels.RDS)
- Cross-reference Jaccard similarity matrix between cell subtype labels (integrated-similaritymatrix.RDS)

## How to apply

Load the pretrained subtype template models (cellSubtypeTemplates.rds from the scCancer package rds directory) and marker files from the package txt directory. For each dataset (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, and the four Xue2022 partitions), read the Seurat object, run predCellType on the RNA assay data for rough labels, and merge T.cells.CD4/T.cells.CD8 into a single T.cells category. Run runCellSubtypeClassify on the five default cell types (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) with dropout.modeling = FALSE and unknown.cutoff = 0.3, saving fine-labels.RDS and similarity-matrix.RDS per dataset. Merge per-dataset fine labels for each cell type by row-binding (rbind) into integrated-finelabels.RDS, then run similarityCalculation on the integrated fine labels to build the cross-reference Jaccard similarity matrix saved as integrated-similaritymatrix.RDS. The rationale is to fully preserve original atlas labeling while quantitatively relating subtype labels across references using trained classifiers.

## Related tools

- **scCancer** (Core R package providing predCellType, runCellSubtypeClassify, and similarityCalculation used to produce rough labels, fine subtype labels, and cross-dataset Jaccard similarity matrices) — czythu/scCancer
- **scCancer2** (Updated package version that introduces supervised machine learning subtype annotation and quantitative cross-dataset label similarity maps)
- **Seurat** (Seurat objects store the per-dataset scRNA-seq expression (RNA assay @data) that predCellType consumes)
- **dplyr** (Loaded alongside scCancer in the R analysis scripts for label/data manipulation)
- **R** (Execution environment for the integration and similarity-calculation scripts)

## Examples

```
all.matrix <- similarityCalculation(all.list, annoPath)
```

## Evaluation signals

- The Lymphoid-B row of the integrated Jaccard similarity matrix matches the reported pattern: low similarity to naive B cells (0.10 with NSCLC, 0.17 with BRCA) and high similarity to memory B cells (0.72 with NSCLC, 0.67 with BRCA).
- Each dataset yields a fine-labels.RDS and similarity-matrix.RDS after runCellSubtypeClassify with dropout.modeling = FALSE and unknown.cutoff = 0.3.
- Rough labels from predCellType show T.cells.CD4 and T.cells.CD8 merged into a single T.cells category before fine subtype classification.
- The integrated-finelabels.RDS is the row-bound (rbind) combination of per-dataset fine labels for each of the five cell types (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial).
- The final integrated-similaritymatrix.RDS contains Jaccard similarity values computed by similarityCalculation over all 15 datasets.

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost, so exact reproducibility may depend on package versions.
- No repository URL or accession IDs are provided for the HCC datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021) or Xue2022, requiring external retrieval.
- A subtype label falls back to 'unknown' when classifier confidence is below unknown.cutoff = 0.3, which can reduce coverage of the integrated label space.

## Evidence

- [abstract] Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets.: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [methods] predCellType rough labeling: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [methods] similarityCalculation builds the cross-dataset matrix: "all.matrix <- similarityCalculation(all.list, annoPath)"
- [methods] unknown.cutoff parameter: "unknown.cutoff = 0.3,"
- [readme] README: cross-dataset label similarity rationale: [evidence span withheld — anchor status: external_doc]
