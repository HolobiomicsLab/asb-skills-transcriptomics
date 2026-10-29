---
name: cross-method-benchmark-accuracy-tabulation
description: 'Use when you need to reproduce or extend the scCancer2 Fig. 2B benchmark:
  comparing cell-type annotation accuracy of scCancer2 against baseline methods across
  multiple datasets and sequencing technologies.'
when_to_use_negative:
- The dataset lacks metadata level-1 reference labels, so prediction accuracy cannot
  be computed against a ground truth
- The goal is de novo subtype-level annotation of a new tumor microenvironment sample
  rather than cross-method accuracy benchmarking of the five major cell types
- The input data is spatial transcriptomics or bulk TCGA data rather than the six
  scRNA-seq benchmark datasets used in Fig. 2B
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCancer2
  role: 'Annotation framework being benchmarked: predCellType for rough level-1 labels
    and runCellSubtypeClassify for fine subtype labels'
  repo: http://lifeome.net/software/sccancer2/
- name: scCancer
  role: R package loaded in the analysis (library(scCancer)) providing predCellType
    and runCellSubtypeClassify
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Loads and holds the benchmark datasets; expression read from assays[['RNA']]@data
- name: xgboost
  role: Baseline method (XGBoost classifier) run on the six benchmark datasets with
    the five major cell types as prediction targets
- name: dplyr
  role: R data manipulation used alongside scCancer in the analysis
- name: scPred
  role: Baseline annotation method compared against scCancer2
- name: OCLR
  role: Baseline annotation method compared against scCancer2
- name: Scibet
  role: Baseline annotation method compared against scCancer2
- name: CHETAH
  role: Baseline annotation method compared against scCancer2
- name: scmap
  role: Baseline annotation method compared against scCancer2
- name: SingleCellNet
  role: Baseline annotation method compared against scCancer2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:aa1c6aa30b85693686fde9d04fa0d809b0a1f2134fbf1039636cc1e54bb886c7
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# cross-method-benchmark-accuracy-tabulation

## Summary

Reproduces the scCancer2 benchmark by running the scCancer2 annotation framework (predCellType + runCellSubtypeClassify) and eight baselines (scPred, OCLR, multinomial, Scibet, CHETAH, scmap, SingleCellNet, XGBoost) on six scRNA-seq datasets spanning five major cell types and three sequencing technologies, then tabulating prediction accuracy per method per dataset.

## When to use

Use when you need to reproduce or extend the scCancer2 Fig. 2B benchmark: comparing cell-type annotation accuracy of scCancer2 against baseline methods across multiple datasets and sequencing technologies.

## When NOT to use

- The dataset lacks metadata level-1 reference labels, so prediction accuracy cannot be computed against a ground truth
- The goal is de novo subtype-level annotation of a new tumor microenvironment sample rather than cross-method accuracy benchmarking of the five major cell types
- The input data is spatial transcriptomics or bulk TCGA data rather than the six scRNA-seq benchmark datasets used in Fig. 2B

## Inputs

- Per-study benchmark .RDS files (Seurat objects) identified by study label
- Level-1 reference cell-type labels from each dataset's metadata
- Package-shipped cellSubtypeTemplates.rds subtype templates and marker files from scCancer2

## Outputs

- fine-labels.RDS per dataset
- similarity-matrix.RDS per dataset
- accuracy_table.csv: one prediction-accuracy entry per method per benchmark dataset across the three sequencing technologies

## How to apply

Load each benchmark dataset as a Seurat object via readRDS using the per-study .RDS files keyed by study label (e.g., HCC datasets Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021). Run scCancer2's predCellType on the normalized expression matrix (assay slot 'RNA' @data) to get rough level-1 labels, merging 'T.cells.CD4' and 'T.cells.CD8' into 'T.cells' and intersecting with the five-type list (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial). Then run runCellSubtypeClassify per cell type with dropout.modeling = FALSE and unknown.cutoff = 0.3, using the package-shipped cellSubtypeTemplates.rds templates and marker files. Run each baseline method on the same six datasets targeting the same five major cell types, compare predictions against the metadata level-1 reference labels, and assemble per-method per-dataset accuracies into accuracy_table.csv structured as in Fig. 2B / Section 3.2.

## Related tools

- **scCancer2** (Annotation framework being benchmarked: predCellType for rough level-1 labels and runCellSubtypeClassify for fine subtype labels) — http://lifeome.net/software/sccancer2/
- **scCancer** (R package loaded in the analysis (library(scCancer)) providing predCellType and runCellSubtypeClassify) — https://github.com/czythu/scCancer
- **Seurat** (Loads and holds the benchmark datasets; expression read from assays[['RNA']]@data)
- **xgboost** (Baseline method (XGBoost classifier) run on the six benchmark datasets with the five major cell types as prediction targets)
- **dplyr** (R data manipulation used alongside scCancer in the analysis)
- **scPred** (Baseline annotation method compared against scCancer2)
- **OCLR** (Baseline annotation method compared against scCancer2)
- **Scibet** (Baseline annotation method compared against scCancer2)
- **CHETAH** (Baseline annotation method compared against scCancer2)
- **scmap** (Baseline annotation method compared against scCancer2)
- **SingleCellNet** (Baseline annotation method compared against scCancer2)

## Examples

```
t.results <- predCellType(expr@assays[["RNA"]]@data); expr$Cell.Type <- t.results$type.pred
```

## Evaluation signals

- accuracy_table.csv contains exactly one accuracy entry per method per benchmark dataset across the three sequencing technologies
- Predicted labels intersect with the five-type default list: T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial
- Per-method accuracies can be ranked and reproduce the benchmark ordering reported in Section 3.2 / Fig. 2B
- runCellSubtypeClassify was called with dropout.modeling = FALSE and unknown.cutoff = 0.3 for every cell type
- fine-labels.RDS and similarity-matrix.RDS are saved for each of the six datasets

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost in the article
- No repository URL or accession IDs are provided for the HCC benchmark datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021)
- T.cells.CD4 and T.cells.CD8 must be merged into a unified 'T.cells' category before comparison, so subtype-level (CD4 vs CD8) accuracy is not captured

## Evidence

- [results] Benchmark finding: [evidence span withheld — anchor status: not_found]
- [methods] Rough annotation step: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [methods] Fine annotation step: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,"
- [methods] Unknown cutoff parameter: "unknown.cutoff = 0.3,"
- [abstract] Framework rationale: "In this updated package scCancer2, designed for integrative tumor scRNA-seq data analysis, we developed a supervised machine learning framework to annotate TME cells"
