---
name: multi-method-cell-type-benchmarking
description: Use when you have single-cell count matrices with manually annotated
  labels and need to quantitatively compare which automatic cell-type identification
  methods (eager, lazy or marker-based) perform , e.g. across platforms or tissues.
when_to_use_negative:
- The task is a single-method prediction on a new dataset rather than a comparative
  benchmark against manual labels — use one method directly instead of the full evaluation
  harness
- Only marker lists and no labeled reference/training data are available — markersupervised
  interfaces cover this, but accuracy/F1-score against manual labels cannot be computed
- The training and testing data are not cell-type-labeled count matrices (e.g. already-annotated
  feature tables or non scRNA-seq data)
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AutomaticCellTypeIdentification
  role: Unified R wrapper (eagersupervised, lazysupervised, markersupervised interfaces)
    used to run all evaluated cell-type identification methods in a uniform usage
    format
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: Seurat
  role: Preprocessing and marker identification; the Seurat pipeline is used to find
    cell type specific markers via FindAllMarkers with manually annotated labels as
    'active.ident'
- name: clustifyr
  role: Evaluated eager-supervised method; top performer on accuracy in the Tabula
    Muris cross-platform evaluation
- name: scLearn
  role: Evaluated eager-supervised method; top performer on accuracy in the cross-platform
    applicability test
  repo: https://github.com/bm2-lab/scLearn
- name: scHPL
  role: Evaluated method; top performer for both accuracy and F1-score in the cross-platform
    test
- name: SciBet
  role: Evaluated eager-supervised method; among the highest F1-score performers in
    the cross-platform test
  repo: https://github.com/zwj-tina/scibetR
- name: SingleR
  role: Evaluated eager-supervised method; among the highest F1-score performers in
    the cross-platform test
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:10b46f7619b89e1fa81616d3088e3b2a24810c2419be6e38049162be07da1083
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
  - AutomaticCellTypeIdentification
  - R
  - SciBet
  - Seurat
  - SingleR
  - clustifyr
  - scHPL
  - scLearn
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# multi-method-cell-type-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple automatic cell-type identification methods for scRNA-seq data (e.g. clustifyr, scLearn, scHPL, SciBet, SingleR) by running them through the AutomaticCellTypeIdentification R package on shared train/test splits and comparing accuracy, F1-score and unlabeling rate. It exists because manual annotation is time-consuming and knowledge-dependent, while automatic methods vary in performance.

## When to use

Use when you have single-cell count matrices with manually annotated labels and need to quantitatively compare which automatic cell-type identification methods (eager, lazy or marker-based) perform best, e.g. across platforms or tissues.

## When NOT to use

- The task is a single-method prediction on a new dataset rather than a comparative benchmark against manual labels — use one method directly instead of the full evaluation harness
- Only marker lists and no labeled reference/training data are available — markersupervised interfaces cover this, but accuracy/F1-score against manual labels cannot be computed
- The training and testing data are not cell-type-labeled count matrices (e.g. already-annotated feature tables or non scRNA-seq data)

## Inputs

- scRNA-seq count matrix (genes as rows, cells as columns) with manually annotated cell-type labels
- Tabula Muris Smart-seq2 and 10x tissue datasets from https://tabula-muris.ds.czbiohub.org/
- Cell-type marker lists (e.g. from PanglaoDB, CellMarker or CancerSEA) for markersupervised methods

## Outputs

- Per-method per-tissue metric table (accuracy, F1-score, unlabeling rate), e.g. cross_platform_performance.csv
- Grouped comparison plot of method rankings (e.g. reproducing Fig. 2C)
- Predicted cell-type labels for each test tissue per method

## How to apply

Preprocess the Tabula Muris count matrices (https://tabula-muris.ds.czbiohub.org/) in R/Seurat: drop genes not expressed in any cell, remove cells with fewer than 200 expressed genes, cells exceeding twice the standard deviation of the mean expressed-gene count, cells with >20% mitochondrial reads, and cells lacking a manual label. Split by platform (Smart-seq2 tissues of limb muscle, liver, thymus and tongue as training; 10x tissues of the other seven tissues as testing) or partition with the caret 'createDataPartition'/'createFolds' functions for five-fold cross-validation. Run each method through the AutomaticCellTypeIdentification package using its uniform interface (eagersupervised, lazysupervised or markersupervised, e.g. eagersupervised(train, test, label_train, method='SciBet')). Compare predictions to the manual labels, computing accuracy, F1-score (mean per-type harmonic mean of precision and recall) and unlabeling rate, then aggregate across test tissues and rank methods.

## Related tools

- **AutomaticCellTypeIdentification** (Unified R wrapper (eagersupervised, lazysupervised, markersupervised interfaces) used to run all evaluated cell-type identification methods in a uniform usage format) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **Seurat** (Preprocessing and marker identification; the Seurat pipeline is used to find cell type specific markers via FindAllMarkers with manually annotated labels as 'active.ident')
- **clustifyr** (Evaluated eager-supervised method; top performer on accuracy in the Tabula Muris cross-platform evaluation)
- **scLearn** (Evaluated eager-supervised method; top performer on accuracy in the cross-platform applicability test) — https://github.com/bm2-lab/scLearn
- **scHPL** (Evaluated method; top performer for both accuracy and F1-score in the cross-platform test)
- **SciBet** (Evaluated eager-supervised method; among the highest F1-score performers in the cross-platform test) — https://github.com/zwj-tina/scibetR
- **SingleR** (Evaluated eager-supervised method; among the highest F1-score performers in the cross-platform test)

## Examples

```
eagersupervised(train,test,label_train,method='SciBet')
```

## Evaluation signals

- Reproduced accuracy ranking should surface clustifyr, scLearn and scHPL as the best eager-learning methods
- Reproduced F1-score ranking should surface scHPL, SciBet and SingleR as the top performers
- The per-method metric table (cross_platform_performance.csv) contains accuracy, F1-score and unlabeling rate for every method on each of the seven 10x test tissues
- Each method is invoked through the AutomaticCellTypeIdentification package's uniform usage format so results are directly comparable
- Preprocessing invariants hold: no genes with zero expression across all cells, no cells with <200 expressed genes or >20% mitochondrial reads, and no cells lacking a manual label

## Limitations

- Method rankings are context-dependent: different methods lead under different metrics (clustifyr/scLearn/scHPL for accuracy vs scHPL/SciBet/SingleR for F1-score), so method choice should follow the available information and prioritized metric
- Cross-platform generalization is evaluated only with Smart-seq2 as training and 10x as testing in this configuration; other platform/train-test direction pairs are not covered
- Running time and unlabeling rate must be weighed alongside accuracy; methods that leave cells unlabeled trade coverage for precision
- The article index provides no discussion-section text, so unresolved challenges and caveats could not be extracted

## Evidence

- [readme] Uniform interfaces: [evidence span withheld — anchor status: not_found]
