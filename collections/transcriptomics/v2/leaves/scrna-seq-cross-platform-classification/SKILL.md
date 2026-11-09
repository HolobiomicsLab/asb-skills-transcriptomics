---
name: scrna-seq-cross-platform-classification
description: Use when a scRNA-seq reference dataset must label cells from a different
  platform (Smart-seq2 vs 10x) or condition (normal vs tumor), and method performance
  (accuracy, F1-score, sensitivity/specificity, unlabeling rate) must be compared.
when_to_use_negative:
- The task is manual cluster annotation with canonical markers rather than automated,
  supervised label transfer against a labeled reference
- Training and testing data come from the same platform and same condition — use a
  self-projection five-fold cross-validation design instead
- Only one method is of interest and no benchmark against alternatives is needed —
  call that method directly rather than through the evaluation workflow
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper that runs all evaluated eager, lazy and marker-based cell-type identification
    methods on train/test count matrices
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Pipeline used for preprocessing and finding cell-type specific markers (FindAllMarkers)
    for marker-based methods
- name: caret (R package)
  role: createDataPartition downsamples lung datasets to ~10,000 cells with equal
    cell-type proportions; createFolds builds five-fold cross-validation splits
- name: CELLBLAST
  role: Lazy supervised method evaluated for cross-platform prediction
  repo: https://github.com/gao-lab/Cell_BLAST
- name: CHETAH
  role: Eager supervised method evaluated in the benchmark
  repo: https://github.com/jdekanter/CHETAH
- name: SciBet
  role: Eager supervised method; among top F1-score performers in cross-platform transfer
  repo: https://github.com/zwj-tina/scibetR
- name: scLearn
  role: Eager supervised method; among top accuracy performers in cross-platform transfer
  repo: https://github.com/bm2-lab/scLearn
- name: CaSTLe
  role: Eager supervised method evaluated on the normal-lung-to-lung-cancer transfer
    task
  repo: https://github.com/yuvallb/CaSTLe
- name: ACTINN
  role: Eager supervised method evaluated in the benchmark
  repo: https://github.com/mafeiyang/ACTINN
- name: scmap-cell
  role: Lazy supervised method evaluated for cross-platform prediction
  repo: https://github.com/hemberg-lab/scmap
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:11ca16838395380fb5de2d1c6a95a3f3565738a51b871c1ca90e4e0a84543f2e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  repo_url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  tools:
  - ACTINN
  - AutomaticCellTypeIdentification
  - CELLBLAST
  - CHETAH
  - CaSTLe
  - CellFishing.jl
  - R
  - SciBet
  - Seurat
  - SingleR
  - caret (R package)
  - clustifyr
  - scHPL
  - scLearn
  - scmap-cell
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# scrna-seq-cross-platform-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking protocol for automatic cell-type identification on scRNA-seq data across platforms or biological states — e.g. training on Smart-seq2 Tabula Muris tissues and testing on 10x tissues, or training on normal lung and testing on lung cancer (GSE131907) — via the AutomaticCellTypeIdentification R package.

## When to use

Use when a scRNA-seq reference dataset must label cells from a different platform (Smart-seq2 vs 10x) or condition (normal vs tumor), and method performance (accuracy, F1-score, sensitivity/specificity, unlabeling rate) must be compared.

## When NOT to use

- The task is manual cluster annotation with canonical markers rather than automated, supervised label transfer against a labeled reference
- Training and testing data come from the same platform and same condition — use a self-projection five-fold cross-validation design instead
- Only one method is of interest and no benchmark against alternatives is needed — call that method directly rather than through the evaluation workflow

## Inputs

- Count matrix (genes as rows, cells as columns) with manually annotated cell-type labels, from GEO/ArrayExpress/GSA (e.g. GSE131907, GSE116470) or Tabula Muris
- Platform-split or condition-split training and testing datasets (Smart-seq2 vs 10x; normal lung vs lung tumor)
- Optional marker lists from PanglaoDB, CellMarker or CancerSEA for marker-based methods

## Outputs

- Per-method prediction table (predicted cell type or unassigned per cell)
- Per-method metric table (accuracy, F1-score, unlabeling rate, sensitivity, specificity, running time)
- Method ranking / grouped comparison plot (e.g. reproducing Fig. 2C or Fig. 3B)

## How to apply

Download count matrices with manual labels (e.g. Tabula Muris from https://tabula-muris.ds.czbiohub.org/ or GSE131907 lung data), then preprocess by filtering genes not expressed in any cells, cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed-gene count, cells with >20% mitochondrial gene expression, and cells without cell-type labels. Split by platform or condition (Smart-seq2 limb muscle/liver/thymus/tongue as training, 10x tissues as testing; or normal lung vs tumor downsampled to ~10,000 cells with equal cell-type proportions using caret's createDataPartition). Run each method through the AutomaticCellTypeIdentification package's eagersupervised/lazysupervised/markersupervised interfaces on the held-out data. Compare predictions to manual labels computing accuracy, F1-score, unlabeling rate, and for tumor detection sensitivity/specificity with 'unknown'/'unassigned' predictions treated as malignant. Rank methods and validate against reported top performers (e.g. clustifyr, scLearn, scHPL for cross-platform accuracy; scHPL, SciBet, SingleR for F1-score; clustifyr near-100% specificity and sensitivity for tumor detection).

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper that runs all evaluated eager, lazy and marker-based cell-type identification methods on train/test count matrices) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Pipeline used for preprocessing and finding cell-type specific markers (FindAllMarkers) for marker-based methods)
- **caret (R package)** (createDataPartition downsamples lung datasets to ~10,000 cells with equal cell-type proportions; createFolds builds five-fold cross-validation splits)
- **CELLBLAST** (Lazy supervised method evaluated for cross-platform prediction) — https://github.com/gao-lab/Cell_BLAST
- **CHETAH** (Eager supervised method evaluated in the benchmark) — https://github.com/jdekanter/CHETAH
- **SciBet** (Eager supervised method; among top F1-score performers in cross-platform transfer) — https://github.com/zwj-tina/scibetR
- **scLearn** (Eager supervised method; among top accuracy performers in cross-platform transfer) — https://github.com/bm2-lab/scLearn
- **CaSTLe** (Eager supervised method evaluated on the normal-lung-to-lung-cancer transfer task) — https://github.com/yuvallb/CaSTLe
- **ACTINN** (Eager supervised method evaluated in the benchmark) — https://github.com/mafeiyang/ACTINN
- **scmap-cell** (Lazy supervised method evaluated for cross-platform prediction) — https://github.com/hemberg-lab/scmap

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- Every evaluated method yields sensitivity and specificity values in [0, 1] on the tumor-detection task, ordering reproduces Fig. 3B with clustifyr near 100% for both
- Cross-platform ranking reproduces clustifyr, scLearn and scHPL as top accuracy performers and scHPL, SciBet and SingleR as top F1-score performers
- Filtered datasets contain no genes with zero expression, no cells with <200 expressed genes, no cells with >2x SD expressed genes, and no cells with >20% mitochondrial reads
- Test predictions compare directly against manual annotations; per-method accuracy/F1/unlabeling rate are computable for every method-tissue pair
- Per-method metric table exports cleanly as cross_platform_performance.csv with one row per method per test tissue

## Limitations

- Cross-platform performance depends on the training split: Smart-seq2 training used limb muscle, liver, thymus and tongue (tissues with more cell types), so results may not generalize to other splits
- Methods requiring markers need markers downloaded from PanglaoDB, CellMarker or CancerSEA and cell-type specific markers found via the Seurat pipeline
- Some methods are python-based and require the conda environment to be loaded at the beginning in R
- Lung benchmark uses downsampled ~10,000-cell datasets with equal cell-type proportions, so class-imbalance behavior is not captured

## Evidence

- [readme] Method interfaces: [evidence span withheld — anchor status: not_found]
