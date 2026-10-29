---
name: cross-platform-cell-type-classification
description: 'Use when evaluating which automatic cell-type identification method
  transfers across scRNA-seq platforms: labeled training data from one platform, held-out
  or cross-platform test data from others, ranked by accuracy, F1 and unlabeling rate.'
when_to_use_negative:
- Training and testing cells come from the same platform/dataset — the self-projection
  evaluation (five-fold cross-validation within one dataset) is the appropriate design
  instead
- Input is raw sequencing reads rather than a preprocessed count matrix with gene
  rows and cell columns; the package expects a count matrix
- No manually annotated labels or marker sets are available for any platform, so neither
  supervised nor marker-based interfaces can be trained
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper exposing eagersupervised, lazysupervised and markersupervised interfaces
    to run all evaluated methods under one calling convention
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Normalizes the count matrix (NormalizeData) and derives cell-type-specific
    marker sets via FindAllMarkers for marker-learning methods
- name: caret
  role: Provides createFolds for the five-fold cross-validation splits used as cross-platform
    training/testing sets
- name: CELLBLAST
  role: Lazy-learning method evaluated via lazysupervised for cross-platform prediction
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scmap-cell
  role: Lazy-learning method that outperformed CELLBLAST in the cross-platform PBMC
    comparison
  repo: https://github.com/hemberg-lab/scmap
- name: scLearn
  role: Eager-learning method among the top three in mean cross-platform accuracy
  repo: https://github.com/bm2-lab/scLearn
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:d722eec0b6d4786e2e85bf788f5989586c40b9f03267e3c2d6474dd6bb59a9fb
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
  - CaSTLe
  - CellFishing.jl
  - Garnett
  - R
  - SciBet
  - Seurat
  - caret
  - scID
  - scLearn
  - scmap-cell
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-platform-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark automatic cell-type identification methods in cross-platform prediction on scRNA-seq data (e.g. PBMC from seven platforms), training on one platform and testing on others, and rank eager, lazy and marker-learning methods by accuracy, F1-score and unlabeling rate.

## When to use

Use when evaluating which automatic cell-type identification method transfers across scRNA-seq platforms: labeled training data from one platform, held-out or cross-platform test data from others, ranked by accuracy, F1 and unlabeling rate.

## When NOT to use

- Training and testing cells come from the same platform/dataset — the self-projection evaluation (five-fold cross-validation within one dataset) is the appropriate design instead
- Input is raw sequencing reads rather than a preprocessed count matrix with gene rows and cell columns; the package expects a count matrix
- No manually annotated labels or marker sets are available for any platform, so neither supervised nor marker-based interfaces can be trained

## Inputs

- PBMC count matrix with manually annotated cell-type labels (SCP424, Single Cell Portal), genes as rows and cells as columns
- Cross-platform train/test split assignments (five caret createFolds folds, four folds training / one fold testing)
- Cell-type marker lists (Seurat FindAllMarkers top 5 and 15 genes per type plus canonical PBMC markers) for marker-learning methods

## Outputs

- pbmc_cross-platform_results.csv metric table: accuracy, mean per-cell-type F1-score and unlabeling rate per method per platform pair
- Method rankings separating eager-learning top performers (clustifyr, scLearn, scPred by accuracy; scPred, Seurat, SingleCellNet by F1) from lazy and marker-learning methods

## How to apply

Download the PBMC count matrix with manually annotated labels (SCP424 from the Single Cell Portal) and preprocess it: filter genes not expressed in any cells, remove cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean, cells with more than 20% mitochondrial gene expression, and cells lacking cell-type labels. Build marker sets with the Seurat pipeline (NormalizeData, then FindAllMarkers with manual labels as 'active.ident', only.pos = TRUE, taking the top 5 and 15 genes per cell type, supplemented by canonical PBMC markers). Split each platform dataset into five folds with caret's createFolds and run five rounds of cross-validation (four folds training, one fold testing). Invoke the methods through the AutomaticCellTypeIdentification package's eagersupervised, lazysupervised and markersupervised interfaces with a consistent train/test/label_train calling convention. Compute accuracy excluding unlabeled cells, mean per-cell-type F1-score, and unlabeling rate for every method and platform pair, then aggregate into a CSV table (pbmc_cross_platform_results.csv) and rank methods within each learning category.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper exposing eagersupervised, lazysupervised and markersupervised interfaces to run all evaluated methods under one calling convention) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Normalizes the count matrix (NormalizeData) and derives cell-type-specific marker sets via FindAllMarkers for marker-learning methods)
- **caret** (Provides createFolds for the five-fold cross-validation splits used as cross-platform training/testing sets)
- **CELLBLAST** (Lazy-learning method evaluated via lazysupervised for cross-platform prediction) — https://github.com/gao-lab/Cell_BLAST
- **scmap-cell** (Lazy-learning method that outperformed CELLBLAST in the cross-platform PBMC comparison) — https://github.com/hemberg-lab/scmap
- **scLearn** (Eager-learning method among the top three in mean cross-platform accuracy) — https://github.com/bm2-lab/scLearn

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- Results table contains one entry per method per platform pair with accuracy, F1-score and unlabeling rate all within the 0–1 range
- Accuracy is computed excluding unlabeled cells; the unlabeling rate is reported separately rather than penalizing accuracy
- Replication of the paper's ranking: top three eager methods by mean accuracy are clustifyr, scLearn and scPred; by mean F1-score they are scPred, Seurat and SingleCellNet
- Among lazy learning methods, CellFishing.jl and scmap-cell should outperform CELLBLAST
- All methods are invoked through the same AutomaticCellTypeIdentification interface format (eagersupervised/lazysupervised/markersupervised) so metric differences reflect the methods, not the wrappers

## Limitations

- Marker-learning methods (CellAssign, SCSA, MarkerCount) performed best within their category but with slightly lower accuracy and F1-score than the top eager-learning methods
- Preprocessing thresholds (≥200 expressed genes, ≤2× SD of mean expressed genes, ≤20% mitochondrial expression) must be applied consistently or cross-platform results are not comparable
- Cells without cell-type information must be deleted before splitting; unlabeling behavior of individual methods otherwise distorts accuracy comparisons
- No discussion-section text was available for this article, so failure-mode guidance beyond the reported comparisons is limited

## Evidence

- [readme] Package exposes three method interfaces: [evidence span withheld — anchor status: external_doc]
- [readme] Expected input format is a count matrix: [evidence span withheld — anchor status: external_doc]
