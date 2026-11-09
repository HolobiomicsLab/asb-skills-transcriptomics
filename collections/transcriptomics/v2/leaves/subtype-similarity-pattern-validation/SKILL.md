---
name: subtype-similarity-pattern-validation
description: Use when reproducing a cross-dataset cell label similarity map (e.g.,
  B cell subtypes across HCC, NSCLC, BRCA references) and checking whether recomputed
  Jaccard values reproduce a reported pattern from the scCancer2 paper.
when_to_use_negative:
- When the goal is de novo cell subtype annotation for a single dataset without any
  cross-dataset label comparison — use predCellType/runCellSubtypeClassify directly
  instead of the similarity-map validation workflow
- When the input is raw count data that has not been rough-labeled and fine-labeled
  by the scCancer2 classification pipeline — similarityCalculation requires integrated
  fine labels as input
- When comparing labels across datasets where the reference label sets are not preserved
  from published atlases (the method relies on fully preserving original labeling
  in cell atlases)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scCancer2
  role: Provides predCellType, runCellSubtypeClassify, and similarityCalculation functions
    used to generate the cross-dataset Jaccard similarity maps that are validated
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Reads the per-dataset Seurat objects (.RDS) whose RNA assay data feed predCellType
    and runCellSubtypeClassify
- name: dplyr
  role: R data manipulation used alongside scCancer in the annotation/similarity workflow
    scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:7486e7f2bd09cbcc69c60250f49b69d29e8291b1284f190e485984d94fcf944d
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# subtype-similarity-pattern-validation

## Summary

Validates cross-dataset cell-subtype label similarity patterns by recomputing Jaccard similarity matrices with scCancer2's similarityCalculation on integrated fine labels, and confirming specific label-vs-reference similarity values (e.g., HCC 'Lymphoid-B' naive vs memory B cell rows) match published figures such as Fig. 3A / Section 3.3.

## When to use

Use when reproducing a cross-dataset cell label similarity map (e.g., B cell subtypes across HCC, NSCLC, BRCA references) and checking whether recomputed Jaccard values reproduce a reported pattern from the scCancer2 paper.

## When NOT to use

- When the goal is de novo cell subtype annotation for a single dataset without any cross-dataset label comparison — use predCellType/runCellSubtypeClassify directly instead of the similarity-map validation workflow
- When the input is raw count data that has not been rough-labeled and fine-labeled by the scCancer2 classification pipeline — similarityCalculation requires integrated fine labels as input
- When comparing labels across datasets where the reference label sets are not preserved from published atlases (the method relies on fully preserving original labeling in cell atlases)

## Inputs

- Pretrained subtype template models (cellSubtypeTemplates.rds from the scCancer package rds directory)
- Marker gene files from the scCancer package txt directory
- Per-dataset Seurat objects (.RDS) for Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, and four partitions of Xue2022
- Integrated fine-label annotations (integrated-finelabels.RDS) per cell type

## Outputs

- fine-labels.RDS and similarity-matrix.RDS per dataset
- integrated-finelabels.RDS (row-bound per-dataset fine labels per cell type)
- integrated-similaritymatrix.RDS (cross-reference Jaccard similarity matrix)
- Validation report comparing the target label row (e.g., Lymphoid-B) against the published pattern from Section 3.3 / Fig. 3A

## How to apply

First reproduce the integrated fine labels: run predCellType on each dataset's RNA assay data for rough labels (merging T.cells.CD4/T.cells.CD8 into T.cells), then runCellSubtypeClassify on the five default cell types (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) with dropout.modeling = FALSE and unknown.cutoff = 0.3, saving fine-labels.RDS and similarity-matrix.RDS per dataset. Row-bind the per-dataset fine labels for each cell type into integrated-finelabels.RDS, then run similarityCalculation on the integrated fine labels to build the cross-reference Jaccard similarity matrix (integrated-similaritymatrix.RDS). The rationale is that classifier-based similarity maps quantify relationships between cell subtype labels defined in different published atlases (15 datasets, 594 samples) while preserving original labeling. To validate, extract the row of interest (e.g., HCC 'Lymphoid-B') from the recomputed matrix and confirm the pattern: low similarity to naive B cells (0.10 NSCLC, 0.17 BRCA) and high similarity to memory B cells (0.72 NSCLC, 0.67 BRCA), consistent with Section 3.3 / Fig. 3A where Lymphoid-B acts as a union label closest to memory B cells.

## Related tools

- **scCancer2** (Provides predCellType, runCellSubtypeClassify, and similarityCalculation functions used to generate the cross-dataset Jaccard similarity maps that are validated) — https://github.com/czythu/scCancer
- **Seurat** (Reads the per-dataset Seurat objects (.RDS) whose RNA assay data feed predCellType and runCellSubtypeClassify)
- **dplyr** (R data manipulation used alongside scCancer in the annotation/similarity workflow scripts)

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3); all.matrix <- similarityCalculation(all.list, annoPath)
```

## Evaluation signals

- Lymphoid-B row of the recomputed similarity matrix shows naive B cell Jaccard values of 0.10 (NSCLC) and 0.17 (BRCA)
- Lymphoid-B row shows memory B cell Jaccard values of 0.72 (NSCLC) and 0.67 (BRCA), confirming the union label is closest to memory B cells
- The same pattern reported in Section 3.3 / Fig. 3A is reproduced across both reference datasets (NSCLC and BRCA) rather than only one
- Per-dataset similarity-matrix.RDS and integrated-similaritymatrix.RDS files are produced by similarityCalculation on rbind-ed fine labels for the five default cell types (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial)
- Fine-label assignments respect unknown.cutoff = 0.3 with dropout.modeling = FALSE, and CD4/CD8 T cell labels are merged into T.cells before classification

## Limitations

- Similarity maps depend on the pretrained subtype templates and marker files shipped with the scCancer package; different reference versions may change Jaccard values
- No software version numbers are stated for R, scCancer, Seurat, or xgboost, and no repository/accession IDs are provided for the HCC datasets, complicating exact reproduction
- The validation values (0.10/0.17 naive, 0.72/0.67 memory) are specific to the Lymphoid-B label in the HCC vs NSCLC/BRCA comparison; other label pairs require their own expected patterns

## Evidence

- [abstract] Classifier-based similarity maps between cell subtypes across references were constructed: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [methods] similarityCalculation builds the cross-reference matrix: "all.matrix <- similarityCalculation(all.list, annoPath)"
- [methods] Fine-label classification parameters: "unknown.cutoff = 0.3,"
- [methods] Rough label annotation with predCellType: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [readme] Cross-dataset label similarity is a core scCancer2 capability preserving original atlas labels: [evidence span withheld — anchor status: external_doc]
