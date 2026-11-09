---
name: supervised-multinomial-classifier-training
description: Use when annotating single-cell TME data at fine subtype resolution across
  five compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) and
  you need a supervised classifier trained on published reference datasets with cross-validated
  accuracy/kappa evaluation.
when_to_use_negative:
- The goal is malignant vs non-malignant cell identification without a confident internal
  reference — that task uses the pretrained xgboost model (threshold 0.5) trained
  on pan-cancer TCGA bulk + scRNA-seq data, not the multinomial subtype classifiers
- Input is spatial transcriptomic data requiring the dedicated spatial module rather
  than scRNA-seq subtype annotation
- Only rough compartment-level labels are needed and high-confidence subtype resolution
  with unknown-cutoff handling is unnecessary
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCancer2
  role: Provides runCellSubtypeClassify, predCellType, cellSubtypeTemplates.rds, and
    packaged marker files for training and applying the multinomial subtype classifiers
  repo: http://lifeome.net/software/sccancer2/
- name: scCancer
  role: Underlying R toolkit loaded in the annotation workflow (library(scCancer));
    its zenodo DOI 10.5281/zenodo.10477297 versions the code
  repo: https://doi.org/10.5281/zenodo.10477296
- name: Seurat
  role: Data container for reference scRNA-seq datasets; predCellType is run on expr@assays[["RNA"]]@data
    of the Seurat object
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:f98d78d072499598e24bd2d08b7544df6e90d46f80b81ec0c9946ceda591d21b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# supervised-multinomial-classifier-training

## Summary

Train multinomial cell subtype classifiers on tumor microenvironment (TME) scRNA-seq reference datasets using scCancer2's weighted maximum likelihood framework, then quantify performance via 5-fold cross-validated accuracy and Cohen's kappa. The trained classifiers also underpin cross-dataset cell subtype similarity maps.

## When to use

Use when annotating single-cell TME data at fine subtype resolution across five compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) and you need a supervised classifier trained on published reference datasets with cross-validated accuracy/kappa evaluation.

## When NOT to use

- The goal is malignant vs non-malignant cell identification without a confident internal reference — that task uses the pretrained xgboost model (threshold 0.5) trained on pan-cancer TCGA bulk + scRNA-seq data, not the multinomial subtype classifiers
- Input is spatial transcriptomic data requiring the dedicated spatial module rather than scRNA-seq subtype annotation
- Only rough compartment-level labels are needed and high-confidence subtype resolution with unknown-cutoff handling is unnecessary

## Inputs

- Reference scRNA-seq datasets as Seurat objects (RNA assay log-normalized expression, e.g. author-named .RDS files such as Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, Xue2022)
- Packaged marker gene files under system.file("txt", package = "scCancer") (e.g. genes-scRNA-tcga-sorted.txt)
- cellSubtypeTemplates.rds multinomial submodel templates supplied as submodel.path

## Outputs

- Per-cell fine subtype labels across the five TME compartments, saved as fine-labels.RDS
- Fold-wise and per-dataset mean/variance of classification accuracy and Cohen's kappa from 5-fold cross-validation
- Classifier-based subtype similarity matrix per dataset, saved as similarity-matrix.RDS (via similarityCalculation)

## How to apply

Load each reference dataset as a Seurat object and derive rough labels with predCellType on the RNA assay data, merging CD4/CD8 T-cell labels into a single T.cells class; restrict labels to the five TME compartments by intersecting with the default list. Select high-confidence cells per compartment by marker scoring against the packaged marker files under system.file("txt"), and select discriminative features using entropy and highly-representative-gene criteria. Train the multinomial subtype classifiers by weighted maximum likelihood via runCellSubtypeClassify using cellSubtypeTemplates.rds as submodel.path, with dropout.modeling = FALSE and unknown.cutoff = 0.3 (cells below the cutoff become 'unknown'). Evaluate by 5-fold cross-validation within each of the 15 reference datasets (594 samples, 1,213,469 cells), computing classification accuracy and Cohen's kappa on each held-out fold, then aggregate into per-dataset mean and variance summaries. Persist each dataset's fine labels to fine-labels.RDS and its subtype similarity matrix to similarity-matrix.RDS in the per-dataset output folder.

## Related tools

- **scCancer2** (Provides runCellSubtypeClassify, predCellType, cellSubtypeTemplates.rds, and packaged marker files for training and applying the multinomial subtype classifiers) — http://lifeome.net/software/sccancer2/
- **scCancer** (Underlying R toolkit loaded in the annotation workflow (library(scCancer)); its zenodo DOI 10.5281/zenodo.10477297 versions the code) — https://doi.org/10.5281/zenodo.10477296
- **Seurat** (Data container for reference scRNA-seq datasets; predCellType is run on expr@assays[["RNA"]]@data of the Seurat object)

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3)
```

## Evaluation signals

- 5-fold cross-validation within each of the 15 reference datasets yields held-out fold classification accuracy and Cohen's kappa; the paper's evaluation aggregates these into per-dataset mean and variance (594 samples, 1,213,469 cells total)
- Cells scoring below unknown.cutoff = 0.3 are labeled 'unknown' rather than forced into one of the five compartments — the unknown label proportion should be non-trivial but not dominant
- Fine labels are restricted to the five TME compartments (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial); labels outside this set indicate the rough-label intersection step failed (CD4/CD8 must be merged to T.cells)
- Each processed dataset must produce fine-labels.RDS and similarity-matrix.RDS in its per-dataset output folder; similarity matrices from all 15 datasets compile into the cross-dataset similarity maps

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost in the methods, complicating exact reproduction
- No repository URL or accession IDs are provided for the HCC scRNA-seq datasets (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021), so reference data must be sourced independently
- The subtype vocabulary is limited to five TME compartments with original atlas labelings preserved; subtypes outside this scheme cannot be trained directly
- Feature and cell selection depend on the packaged marker files under system.file("txt"), so marker coverage constrains which subtypes can be learned reliably

## Evidence

- [abstract] In this updated package scCancer2, designed for integrative tumor scRNA-seq data analysis, we developed a supervised machine learning framework to annotate TME cells: "In this updated package scCancer2, designed for integrative tumor scRNA-seq data analysis, we developed a supervised machine learning framework to annotate TME cells"
- [methods] t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path"
- [methods] t.results <- predCellType(expr@assays[["RNA"]]@data): "t.results <- predCellType(expr@assays[["RNA"]]@data)"
- [results] Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets.: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [readme] Cell subtype annotation and cross-dataset label similarity: Our analysis mainly focused on cell subtype annotation by training multiple lightweight machine-learning models on scRNA-seq data.: [evidence span withheld — anchor status: external_doc]
