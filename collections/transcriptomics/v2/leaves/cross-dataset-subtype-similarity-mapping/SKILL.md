---
name: cross-dataset-subtype-similarity-mapping
description: Use when you need to compare/relate cell subtype label schemes across
  published TME scRNA-seq references, or to cross-validate a subtype annotation model
  and report fold-wise accuracy and kappa within each dataset.
when_to_use_negative:
- You need to identify malignant cells when no internal reference cells exist — scCancer2
  handles that with a separate XGBoost-style classifier trained on TCGA bulk + scRNA-seq
  data (threshold 0.5), not the subtype classifiers
- Input is bulk RNA-seq (e.g. TCGA samples) — the subtype annotation/similarity framework
  operates on single-cell RNA assay data
- Your cell types fall outside the five TME compartments (T.cells, Myeloid.cells,
  B.cells, Fibroblast, Endothelial), since the annotation is restricted to that intersection
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCancer2
  role: Provides predCellType, runCellSubtypeClassify, and similarityCalculation for
    rough labeling, multinomial subtype classification, and cross-dataset similarity
    matrices
  repo: http://lifeome.net/software/sccancer2/
- name: scCancer
  role: R package supplying the packaged marker txt files and subtype machinery used
    to train and apply classifiers
  repo: https://doi.org/10.5281/zenodo.10477296
- name: Seurat
  role: Container for reference scRNA-seq objects; RNA assay data are the classifier
    input
- name: dplyr
  role: Used alongside scCancer in the R workflow scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:848c560fbfe30b175f529c4832c66d852d1ea4bf1edb0b4d179688250d520857
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-dataset-subtype-similarity-mapping

## Summary

Train lightweight multinomial subtype classifiers for the five TME compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) on published scRNA-seq references, then use those trained classifiers to quantitatively score similarity between cell subtype labels defined in different reference datasets. Evaluation is via 5-fold cross-validated accuracy and Cohen's kappa across all 15 datasets.

## When to use

Use when you need to compare/relate cell subtype label schemes across published TME scRNA-seq references, or to cross-validate a subtype annotation model and report fold-wise accuracy and kappa within each dataset.

## When NOT to use

- You need to identify malignant cells when no internal reference cells exist — scCancer2 handles that with a separate XGBoost-style classifier trained on TCGA bulk + scRNA-seq data (threshold 0.5), not the subtype classifiers
- Input is bulk RNA-seq (e.g. TCGA samples) — the subtype annotation/similarity framework operates on single-cell RNA assay data
- Your cell types fall outside the five TME compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial), since the annotation is restricted to that intersection

## Inputs

- Published TME scRNA-seq datasets (e.g. HCC datasets Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021; Xue2022) as Seurat R objects (.RDS)
- Packaged marker gene files under system.file("txt", package = "scCancer")
- cellSubtypeTemplates.rds pretrained submodel templates (submodel.path)

## Outputs

- fine-labels.RDS: per-dataset fine cell subtype labels preserving the original atlas labels
- similarity-matrix.RDS: subtype-by-subtype cross-dataset similarity matrix
- Per-dataset mean and variance of 5-fold CV classification accuracy and Cohen's kappa

## How to apply

Load each reference dataset as a Seurat object and derive rough labels with predCellType on the RNA assay data, merging CD4/CD8 T-cell labels into a single T.cells class via gsub; restrict to the five TME compartments by intersecting predicted labels with the default list. Select high-confidence cells per compartment by marker scoring against packaged marker files under system.file("txt"), and choose discriminative features per compartment with entropy and highly-representative-gene criteria. Train multinomial subtype classifiers by weighted maximum likelihood via runCellSubtypeClassify using cellSubtypeTemplates.rds as submodel.path, with dropout.modeling = FALSE and unknown.cutoff = 0.3; cells below the unknown cutoff are left unlabeled rather than forced into a compartment. Perform 5-fold cross-validation within each of the 15 reference datasets (594 samples, 1,213,469 cells), computing accuracy and Cohen's kappa per fold, then aggregate to per-dataset mean and variance. With the trained classifiers, test them on all 15 datasets to fill a subtype-by-subtype similarity matrix (similarityCalculation), preserving each atlas's original fine labels, and save fine-labels.RDS and similarity-matrix.RDS per dataset.

## Related tools

- **scCancer2** (Provides predCellType, runCellSubtypeClassify, and similarityCalculation for rough labeling, multinomial subtype classification, and cross-dataset similarity matrices) — http://lifeome.net/software/sccancer2/
- **scCancer** (R package supplying the packaged marker txt files and subtype machinery used to train and apply classifiers) — https://doi.org/10.5281/zenodo.10477296
- **Seurat** (Container for reference scRNA-seq objects; RNA assay data are the classifier input)
- **dplyr** (Used alongside scCancer in the R workflow scripts)

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, unknown.cutoff = 0.3)
```

## Evaluation signals

- Per-fold accuracy and Cohen's kappa computed on each held-out fold of the 5-fold CV within each of the 15 reference datasets (594 samples, 1,213,469 cells)
- Fold-wise accuracy/kappa aggregate to per-dataset mean and variance summaries without NAs
- Each dataset output folder contains fine-labels.RDS and similarity-matrix.RDS
- Annotated cells restricted to the five TME compartments; labels below unknown.cutoff = 0.3 are not forced into a compartment
- Similarity map is built by testing all trained classifiers on all 15 datasets, so every subtype pair from different references is scored

## Limitations

- No software version numbers stated for R, scCancer, Seurat, or xgboost, complicating exact reproduction
- No repository URL or accession IDs provided for the HCC reference datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021), so obtaining inputs requires manual curation
- Similarity is classifier-based (test-one-reference-on-all), so it reflects classifier agreement rather than direct label-string equivalence

## Evidence

- [abstract] Similarity mapping grounded in abstract: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [methods] similarityCalculation call in methods: "all.matrix <- similarityCalculation(all.list, annoPath)"
- [methods] runCellSubtypeClassify with unknown.cutoff: "unknown.cutoff = 0.3,"
- [readme] README news on cross-dataset similarity: [evidence span withheld — anchor status: external_doc]
