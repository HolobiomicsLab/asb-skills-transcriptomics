---
name: unlabeling-rate-quantification
description: Use when benchmarking automatic cell-type identification methods on labeled
  scRNA-seq test data and you need to expose methods that abstain too often, complementing
  accuracy and F1-score on cross-platform or cross-validation splits.
when_to_use_negative:
- The test cells have no reference labels at all, so accuracy, F1-score and unlabeling-based
  abstention cannot be jointly interpreted
- You are only evaluating marker-learning methods invoked as markersupervised(test,
  marker, ...), where the abstention semantics differ and the metric must be compared
  within-category
- The comparison goal is pure runtime or speed benchmarking, where unlabeling is not
  the discriminative metric
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: AutomaticCellTypeIdentification
  role: Wrapper R package used to run the supervised/lazy/marker methods with a uniform
    usage format and collect unlabeled predictions for metric computation
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret (R package)
  role: Provides createFolds to split each PBMC dataset into five folds so unlabeling
    rate is averaged over cross-validation rounds
- name: Seurat
  role: Used for preprocessing and FindAllMarkers-based marker sets that feed marker-learning
    methods whose unlabeling behavior is scored
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:2b336e33ec728f9e3c5f03eee6d94a54d479276702bb6bd8fc534920c8747d9f
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
  - SingleR
  - caret
  - caret (R package)
  - clustifyr
  - scHPL
  - scID
  - scLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# unlabeling-rate-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the fraction of test cells that an automatic cell-type identification method leaves unlabeled (rejects/abstains), one of the four core evaluation metrics (accuracy, F1-score, unlabeling rate, running time) used in Xie et al. 2021 to benchmark 32 scRNA-seq cell-type identification methods.

## When to use

Use when benchmarking automatic cell-type identification methods on labeled scRNA-seq test data and you need to expose methods that abstain too often, complementing accuracy and F1-score on cross-platform or cross-validation splits.

## When NOT to use

- The test cells have no reference labels at all, so accuracy, F1-score and unlabeling-based abstention cannot be jointly interpreted
- You are only evaluating marker-learning methods invoked as markersupervised(test, marker, ...), where the abstention semantics differ and the metric must be compared within-category
- The comparison goal is pure runtime or speed benchmarking, where unlabeling is not the discriminative metric

## Inputs

- Count matrix (genes x cells) of test cells with manually annotated cell-type labels
- Per-cell prediction outputs from AutomaticCellTypeIdentification (including abstention/'unlabeled' calls)
- Preprocessed training/testing splits (e.g. Tabula Muris Smart-seq2 -> 10x; PBMC SCP424 seven-platform data)

## Outputs

- Unlabeling rate per method per test tissue or platform pair
- Aggregated metric table (accuracy, mean F1-score, unlabeling rate) e.g. cross_platform_performance.csv or pbmc_cross_platform_results.csv
- Grouped comparison figures such as Fig. 2C / Fig. 3A

## How to apply

Run each candidate method (e.g. via the AutomaticCellTypeIdentification R package interfaces eagersupervised, lazysupervised or markersupervised) on count-matrix inputs (genes as rows, cells as columns) over the same training/testing split — such as Smart-seq2-trained models tested on 10x tissues, or PBMC five-fold cross-validation built with caret's createFolds. For each method and test set, count cells whose predicted label is 'unlabeled' (the method's abstention/rejection) and divide by the total number of test cells to get the unlabeling rate. Critically, accuracy is computed excluding unlabeled cells, so the unlabeling rate must be reported jointly with accuracy and mean per-cell-type F1-score: a method can look accurate simply by abstaining on hard cells. Aggregate the metric per method per tissue/platform pair, rank methods, and write tables such as cross_platform_performance.csv or pbmc_cross_platform_results.csv for figure reproduction.

## Related tools

- **AutomaticCellTypeIdentification** (Wrapper R package used to run the supervised/lazy/marker methods with a uniform usage format and collect unlabeled predictions for metric computation) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret (R package)** (Provides createFolds to split each PBMC dataset into five folds so unlabeling rate is averaged over cross-validation rounds)
- **Seurat** (Used for preprocessing and FindAllMarkers-based marker sets that feed marker-learning methods whose unlabeling behavior is scored)

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- Unlabeling rate must lie in [0, 1] and sum with the proportion of labeled cells to 1 per method/test-set pair
- Accuracy must be recomputed excluding unlabeled cells — verify the reported accuracy table states this exclusion
- Per-method metric table contains an unlabeling-rate column alongside accuracy and mean per-cell-type F1-score
- Reproduced rankings should match the article: cross-platform accuracy winners clustifyr/scLearn/scHPL and F1 winners scHPL/SciBet/SingleR, with unlabeling rate reported jointly
- The output CSV (e.g. pbmc_cross_platform_results.csv) covers every method x platform-pair combination before aggregation

## Limitations

- Unlabeling rate can mask poor performance if reported alone: methods that abstain on many cells inflate their accuracy since accuracy excludes unlabeled cells
- Methods that never emit an abstention have a trivially zero unlabeling rate, giving the metric no discriminative power for them
- Unlabeling behavior is platform-pair dependent, so rates from Smart-seq2->10x transfers may not generalize to within-platform cross-validation

## Evidence

- [readme] Count matrix orientation required as package input: [evidence span withheld — anchor status: external_doc]
