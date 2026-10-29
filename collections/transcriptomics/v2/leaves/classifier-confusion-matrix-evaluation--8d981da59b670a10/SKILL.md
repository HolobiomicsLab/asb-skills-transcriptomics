---
name: classifier-confusion-matrix-evaluation
description: Use when a trained or packaged classifier (e.g. TILPRED on a SingleCellExperiment
  of CD8 TIL profiles) must be validated on a held-out test split by computing accuracy,
  per-class specificity/sensitivity from a confusion matrix against annotated labels.
when_to_use_negative:
- The input contains non-CD8 T cells and you have not applied the classifier's filtering
  (filterCD8T / Treg, CD4T, NKT classification) — predictions would be assigned to
  wrong cell classes before evaluation.
- The goal is de novo cell-state discovery or clustering rather than benchmarking
  an existing multinomial classifier against annotated labels.
- The input data is bulk RNA-seq or lacks per-cell reference labels, so no prediction-vs-truth
  confusion matrix can be constructed.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TILPRED
  role: Logistic regression classifier under evaluation; predictTilState() produces
    predictedState labels on the held-out test split
  repo: github.com/carmonalab/TILPRED
- name: R
  role: Environment in which TILPRED runs; used to partition the test split, build
    the confusion matrix and compute accuracy/specificity/sensitivity metrics
- name: SingleCellExperiment
  role: Data container for the test-split expression matrix and annotated CD8 TIL
    state labels
- name: testTILPRED
  role: R Notebook demonstrating example usage of the TILPRED classifier
  repo: github.com/carmonalab/testTILPRED
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:234e97466b1f7d7df2399173d65c10670a6ff78f71f5299b19ee50d1f03d7e6e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  tools:
  - R
  - SingleCellExperiment
  - TILPRED
  - doParallel
  - doRNG
  - testTILPRED
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# classifier-confusion-matrix-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a single-cell classifier (here, the TILPRED multinomial logistic regression CD8 TIL state predictor) by comparing predicted labels against held-out reference labels to build a four-state confusion matrix and derive accuracy with 95% CI, mean specificity and mean sensitivity.

## When to use

Use when a trained or packaged classifier (e.g. TILPRED on a SingleCellExperiment of CD8 TIL profiles) must be validated on a held-out test split by computing accuracy, per-class specificity/sensitivity from a confusion matrix against annotated labels.

## When NOT to use

- The input contains non-CD8 T cells and you have not applied the classifier's filtering (filterCD8T / Treg, CD4T, NKT classification) — predictions would be assigned to wrong cell classes before evaluation.
- The goal is de novo cell-state discovery or clustering rather than benchmarking an existing multinomial classifier against annotated labels.
- The input data is bulk RNA-seq or lacks per-cell reference labels, so no prediction-vs-truth confusion matrix can be constructed.

## Inputs

- SingleCellExperiment object with CD8 T cell expression profiles and annotated state labels (e.g. B16CD8TIL_SCE example dataset)
- 30% held-out test split of cells retaining reference CD8 TIL state labels
- predictTilState() outputs: predictedState labels and stateProbabilityMatrix

## Outputs

- Confusion matrix over the four CD8 TIL states (exhausted, memory-like, naive, EM-like)
- Overall accuracy with 95% confidence interval (expected 0.94, CI 0.92–0.95)
- Mean specificity (expected 0.98) and mean sensitivity (expected 0.90)
- Metrics table, e.g. tilpred_test_metrics.csv

## How to apply

Load the annotated example dataset (B16CD8TIL_SCE SingleCellExperiment shipped with TILPRED) and partition cells into a training portion and a held-out 30% test split, keeping the annotated CD8 TIL state labels for the test cells. Run predictTilState() from TILPRED in mouse mode (human=FALSE) with default scoreThreshold, cellCycleThreshold and filterCD8T settings to obtain predictedState labels and the stateProbabilityMatrix for the four CD8 TIL states: exhausted, memory-like, naive and EM-like. Build the confusion matrix by comparing predictedState against the held-out reference labels, then compute overall accuracy with a 95% confidence interval and the mean specificity and mean sensitivity across classes. Persist the metrics (e.g. to tilpred_test_metrics.csv) and check them against the reported benchmark: accuracy 0.94 (95% CI 0.92–0.95), mean specificity 0.98, mean sensitivity 0.90. Correctness is judged by the confusion matrix being square over the four states and the metrics matching published values within the reported CI.

## Related tools

- **TILPRED** (Logistic regression classifier under evaluation; predictTilState() produces predictedState labels on the held-out test split) — github.com/carmonalab/TILPRED
- **R** (Environment in which TILPRED runs; used to partition the test split, build the confusion matrix and compute accuracy/specificity/sensitivity metrics)
- **SingleCellExperiment** (Data container for the test-split expression matrix and annotated CD8 TIL state labels)
- **testTILPRED** (R Notebook demonstrating example usage of the TILPRED classifier) — github.com/carmonalab/testTILPRED

## Examples

```
data(B16CD8TIL_SCE); pred <- predictTilState(B16CD8TIL_SCE, human=FALSE, filterCD8T=TRUE)
```

## Evaluation signals

- Confusion matrix is square over exactly the four CD8 TIL states: exhausted, memory-like, naive, EM-like
- Overall accuracy falls within the reported 95% CI of 0.92–0.95 (point estimate 0.94)
- Mean specificity ≈ 0.98 and mean sensitivity ≈ 0.90 across the four classes
- Test split corresponds to the remaining 30% of cells held out from training
- Written metrics file (tilpred_test_metrics.csv) reproduces the benchmark values

## Limitations

- TILPRED is no longer maintained; the authors recommend using ProjecTILs instead (github.com/carmonalab/ProjecTILs)
- Benchmark metrics (0.94 accuracy, 0.98 specificity, 0.90 sensitivity) were obtained on the mouse B16 melanoma CD8 TIL dataset (smart-seq2/droplet technologies); other datasets, species modes or thresholds may give different performance
- TILPRED classifies non-CD8 T cells (Treg, CD4T, NKT) via gene-signature enrichment, so contaminated inputs can degrade evaluation

## Evidence

- [results] benchmark accuracy and per-class metrics: [evidence span withheld — anchor status: not_found]
