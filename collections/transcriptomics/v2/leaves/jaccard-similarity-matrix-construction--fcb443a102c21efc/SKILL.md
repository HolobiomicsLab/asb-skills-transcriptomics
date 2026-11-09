---
name: jaccard-similarity-matrix-construction
description: Use when comparing label vocabularies across published single-cell references
  (e.g. is HCC 'Lymphoid-B' closer to naive or memory B cells?) or harmonizing subtype
  annotations from multiple atlases.
when_to_use_negative:
- When only a single dataset is available — the similarity matrix requires cross-reference
  testing across multiple labeled datasets.
- When the goal is malignant cell identification or spatial transcriptome analysis,
  which are separate scCancer2 modules, not label similarity mapping.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scCancer2
  role: Provides predCellType, runCellSubtypeClassify, and similarityCalculation used
    to produce the cross-dataset Jaccard similarity matrix
  repo: https://doi.org/10.5281/zenodo.10477296
- name: scCancer
  role: R package supplying the pretrained subtype templates (cellSubtypeTemplates.rds)
    and marker files (txt directory) consumed by the classifiers
  repo: czythu/scCancer
- name: Seurat
  role: Reads and holds each dataset's Seurat object; rough labels are predicted on
    expr@assays[['RNA']]@data
- name: dplyr
  role: Used alongside scCancer in the R analysis scripts for label handling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:2e0221806745d40163db3e4e40d112dcf1b669347df0d995083c207d86505a5a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# jaccard-similarity-matrix-construction

## Summary

Quantitatively maps the similarity between cell subtype labels defined in different scRNA-seq references by training subtype classifiers, testing them across datasets, and computing a Jaccard similarity matrix (e.g. scCancer2's similarityCalculation over integrated fine labels).

## When to use

Use when comparing label vocabularies across published single-cell references (e.g. is HCC 'Lymphoid-B' closer to naive or memory B cells?) or harmonizing subtype annotations from multiple atlases.

## When NOT to use

- When only a single dataset is available — the similarity matrix requires cross-reference testing across multiple labeled datasets.
- When the goal is malignant cell identification or spatial transcriptome analysis, which are separate scCancer2 modules, not label similarity mapping.

## Inputs

- Pretrained subtype template models (cellSubtypeTemplates.rds from the scCancer package rds directory)
- Marker gene files from the scCancer package txt directory
- Per-dataset Seurat objects (e.g. Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, and four partitions of Xue2022)
- Integrated fine-label RDS files per cell type

## Outputs

- Per-dataset fine-labels.RDS and similarity-matrix.RDS
- integrated-finelabels.RDS (row-bound fine labels per cell type)
- integrated-similaritymatrix.RDS (cross-reference Jaccard similarity matrix)

## How to apply

First annotate each dataset with the pretrained subtype template models (cellSubtypeTemplates.rds) and marker files via predCellType on the RNA assay data to get rough labels, merging T.cells.CD4/T.cells.CD8 into T.cells, then runCellSubtypeClassify on the five default cell types (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) with dropout.modeling = FALSE and unknown.cutoff = 0.3. Save fine-labels.RDS and similarity-matrix.RDS per dataset, then row-bind (rbind) the per-dataset fine labels for each cell type into integrated-finelabels.RDS. Feed the integrated labels into similarityCalculation, which trains classifiers on each reference's labels and tests them on all other datasets to build a cross-reference Jaccard similarity matrix saved as integrated-similaritymatrix.RDS. Interpret the matrix rows to determine which labels are unions or near-equivalents — e.g. HCC 'Lymphoid-B' has low Jaccard similarity to naive B cells (0.10 NSCLC, 0.17 BRCA) but high to memory B cells (0.72 NSCLC, 0.67 BRCA), revealing it as a union label closest to memory B cells.

## Related tools

- **scCancer2** (Provides predCellType, runCellSubtypeClassify, and similarityCalculation used to produce the cross-dataset Jaccard similarity matrix) — https://doi.org/10.5281/zenodo.10477296
- **scCancer** (R package supplying the pretrained subtype templates (cellSubtypeTemplates.rds) and marker files (txt directory) consumed by the classifiers) — czythu/scCancer
- **Seurat** (Reads and holds each dataset's Seurat object; rough labels are predicted on expr@assays[['RNA']]@data)
- **dplyr** (Used alongside scCancer in the R analysis scripts for label handling)

## Examples

```
all.matrix <- similarityCalculation(all.list, annoPath)
```

## Evaluation signals

- The Lymphoid-B row of the integrated similarity matrix matches the reported pattern: naive B cells 0.10 (NSCLC) and 0.17 (BRCA); memory B cells 0.72 (NSCLC) and 0.67 (BRCA).
- Each dataset produces fine-labels.RDS and similarity-matrix.RDS after runCellSubtypeClassify with unknown.cutoff = 0.3.
- integrated-finelabels.RDS contains correctly row-bound (rbind) fine labels per cell type across all datasets.
- Similarity values fall in [0, 1] (Jaccard) and union labels show high similarity to multiple constituent subtypes.

## Limitations

- No software version numbers were stated for R, scCancer, Seurat, or xgboost in the methods, so exact reproducibility may vary.
- No repository URL or accession IDs are provided for the HCC source datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, Xue2022), requiring manual acquisition.
- The unknown.cutoff = 0.3 threshold drops cells below the confidence cutoff, which can leave cells unlabeled and affect similarity estimates.

## Evidence

- [methods] all.matrix <- similarityCalculation(all.list, annoPath): "all.matrix <- similarityCalculation(all.list, annoPath)"
- [abstract] Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets.: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [methods] unknown.cutoff = 0.3,: "unknown.cutoff = 0.3,"
- [methods] t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path"
