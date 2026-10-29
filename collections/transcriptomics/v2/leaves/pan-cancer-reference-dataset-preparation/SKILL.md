---
name: pan-cancer-reference-dataset-preparation
description: Use when a task requires building or curating a pan-cancer TME cell-classification
  training reference and matched external validation biopsies, or when retraining
  scATOMIC/SingleR/Seurat/scmap-cell/CHETAH/scType/SingleCellNet on identical data.
when_to_use_negative:
- The input is a normal tissue sample rather than a tumour biopsy — the default scATOMIC
  model assumes at least some cancer cells are present and the normal_tissue parameter
  would be needed instead.
- The goal is annotating a single new patient sample rather than building/training
  a reference — scATOMIC should be run per patient/biopsy, and inter-patient cancer
  cell effects can interfere with the cancer signature scoring module if samples are
  pooled.
- The dataset is already a fine-grained cell-type annotation output (not a count matrix)
  — this skill operates on raw gene-by-cell counts prior to classification.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scATOMIC
  role: Pan-cancer TME classifier trained on the assembled reference; trained models
    ship in the R package
  repo: https://github.com/abelson-lab/scATOMIC
- name: SingleR
  role: Comparator annotator trained on the same reference and applied to the same
    validation biopsies
- name: Seurat
  role: Comparator annotator trained on the same reference and applied to the same
    validation biopsies
- name: SingleCellNet
  role: Comparator annotator trained on the same reference and applied to the same
    validation biopsies
- name: scmap-cell
  role: Comparator annotator trained on the same reference and applied to the same
    validation biopsies
- name: CHETAH
  role: Comparator annotator trained on the same reference and applied to the same
    validation biopsies
- name: scType
  role: Comparator annotator trained on the same reference and applied to the same
    validation biopsies
- name: R
  role: Environment (R 4.3.2) in which the reference is prepared, annotators trained,
    and F1 scores computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:5de788a1a70d91e20c787d95b671b1ef81dfb15da9eb1780c077a01febb6e2d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - CHETAH
  - R
  - Seurat
  - SingleCellNet
  - SingleR
  - scATOMIC
  - scType
  - scmap-cell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pan-cancer-reference-dataset-preparation

## Summary

Assembles the pan-cancer tumour-microenvironment training reference (>300,000 cancer, immune, and stromal cells across 19 common cancers) and the external validation set (225 tumour biopsies, >350,000 cells) needed to train scATOMIC and benchmark it against comparator scRNA-seq annotators.

## When to use

Use when a task requires building or curating a pan-cancer TME cell-classification training reference and matched external validation biopsies, or when retraining scATOMIC/SingleR/Seurat/scmap-cell/CHETAH/scType/SingleCellNet on identical data.

## When NOT to use

- The input is a normal tissue sample rather than a tumour biopsy — the default scATOMIC model assumes at least some cancer cells are present and the normal_tissue parameter would be needed instead.
- The goal is annotating a single new patient sample rather than building/training a reference — scATOMIC should be run per patient/biopsy, and inter-patient cancer cell effects can interfere with the cancer signature scoring module if samples are pooled.
- The dataset is already a fine-grained cell-type annotation output (not a count matrix) — this skill operates on raw gene-by-cell counts prior to classification.

## Inputs

- Pan-cancer training reference scRNA-seq count matrices (>300,000 cancer/immune/stromal cells across 19 cancers, e.g. GSE176078_Wu_etal_2021_BRCA_scRNASeq.tar.gz)
- External validation set of 225 tumour biopsies with >350,000 cells (e.g. GSE161529 feature list)
- Gene-by-cell raw count matrices (dense matrix, data.frame, or Seurat object)

## Outputs

- Sparse gene-by-cell count matrices for training and validation
- Quality-filtered reference dataset (mito % ≤ 25, unique features ≥ 500)
- Per-tool combined-class F1 score table for blood, stromal, and cancer cells (comparator_f1_scores.csv)

## How to apply

Assemble the pan-cancer training reference of >300,000 cancer, immune, and stromal cells spanning 19 common cancers (e.g., GSE176078 Wu et al. 2021 breast cancer scRNA-seq training data), plus an external validation set of 225 tumour biopsies containing >350,000 cells. Load the gene-by-cell count matrices and convert them to sparse matrices with as(as.matrix(input_matrix), "sparseMatrix") for speed and memory savings. Filter low-quality cells, removing those with more than 25% of reads mapping to mitochondrial genes or fewer than 500 unique features expressed. Train each annotator (scATOMIC v2.0.3, SingleR, Seurat, SingleCellNet, scmap-cell, CHETAH, scType) on this single shared reference and apply each to the same validation biopsies so benchmarking is like-for-like. Finally, map every tool's fine-grained predictions onto three combined categories — blood cell, stromal cell, and cancer cell — to enable combined-class F1 comparison.

## Related tools

- **scATOMIC** (Pan-cancer TME classifier trained on the assembled reference; trained models ship in the R package) — https://github.com/abelson-lab/scATOMIC
- **SingleR** (Comparator annotator trained on the same reference and applied to the same validation biopsies)
- **Seurat** (Comparator annotator trained on the same reference and applied to the same validation biopsies)
- **SingleCellNet** (Comparator annotator trained on the same reference and applied to the same validation biopsies)
- **scmap-cell** (Comparator annotator trained on the same reference and applied to the same validation biopsies)
- **CHETAH** (Comparator annotator trained on the same reference and applied to the same validation biopsies)
- **scType** (Comparator annotator trained on the same reference and applied to the same validation biopsies)
- **R** (Environment (R 4.3.2) in which the reference is prepared, annotators trained, and F1 scores computed)

## Examples

```
sparse_matrix <- as(as.matrix(input_matrix), "sparseMatrix")
```

## Evaluation signals

- Training reference contains >300,000 cancer, immune, and stromal cells covering 19 common cancers; validation set contains 225 biopsies with >350,000 cells.
- Count matrices are of class sparseMatrix after conversion.
- No retained cell exceeds 25% mitochondrial reads and none has fewer than 500 unique features expressed.
- All seven annotators are trained on the identical reference and evaluated on identical validation biopsies (fair like-for-like benchmark).
- Combined-class F1 scores reproduce reported values: scATOMIC 0.95/0.99/0.99 (blood/stroma/cancer); best comparator SingleR 0.92/0.97/0.76.

## Limitations

- Bile duct, bone, endometrial, esophageal, gallbladder, and gastric cancer classes are included in the reference but were not validated in external datasets and may be misclassified as other related cancers.
- Inter-patient cancer cell specific effects can interfere with scATOMIC's cancer signature scoring, so samples should be processed per biopsy, not pooled.
- Datasets larger than ~15,000 cells require a machine with greater than 16GB RAM; Windows does not natively perform parallel functions and runs slower.

## Evidence

- [abstract] We trained scATOMIC on >300,000 cancer, immune, and stromal cells defining a pan-cancer reference across 19 common cancers: "We trained scATOMIC on >300,000 cancer, immune, and stromal cells defining a pan-cancer reference across 19 common cancers and employ a hierarchical approach"
- [abstract] scATOMIC's accuracy was confirmed on 225 tumour biopsies encompassing >350,000 cancer and a variety of TME cells.: "We extensively confirm scATOMIC's accuracy on 225 tumour biopsies encompassing >350,000 cancer and a variety of TME cells."
- [methods] We strongly recommend the user filter low quality cells from their dataset. We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes: "We strongly recommend the user filter low quality cells from their dataset. We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes"
- [methods] We recommend running scATOMIC on sparse matrices to speed up performance and save memory.: "We recommend running scATOMIC on sparse matrices to speed up performance and save memory."
- [readme] scATOMIC is run directly on gene by cell count matrices. We recommend running scATOMIC on sparse matrices to speed up performance and save memory.: "scATOMIC is run directly on gene by cell count matrices. We recommend running scATOMIC on sparse matrices to speed up performance and save memory."
