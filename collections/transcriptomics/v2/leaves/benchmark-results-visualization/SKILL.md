---
name: benchmark-results-visualization
description: Use when multiple automatic cell-type identification methods have been
  run over shared train/test scRNA-seq splits (e.g., Smart-seq2 training, 10x testing
  in Tabula Muris) and you must rank them by accuracy, F1-score, and unlabeling rate
  and render comparison figures.
when_to_use_negative:
- Only a single method is evaluated, so no cross-method ranking or grouped comparison
  is meaningful
- Labels are predicted without a verified ground-truth annotation, so accuracy/F1-score
  cannot be validly computed
- The evaluation target is running time or memory rather than prediction accuracy,
  F1-score and unlabeling rate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper that generates the per-method prediction outputs on Tabula Muris
    test data that the metrics and plots summarize
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: R
  role: Environment in which metric aggregation, CSV export and grouped plotting are
    performed
- name: Seurat
  role: Preprocessing pipeline (filtering, NormalizeData, marker finding) that yields
    the labeled matrices underlying the benchmark comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:17611a898e6d381ea1e4e9f8146a599bf74faecc25bd70f2259b7a108fc080bc
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# benchmark-results-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-method benchmark metrics (accuracy, F1-score, unlabeling rate) from automatic cell-type identification runs on scRNA-seq data into a summary table and grouped comparison plots, enabling ranking of methods as in the Tabula Muris evaluation.

## When to use

Use when multiple automatic cell-type identification methods have been run over shared train/test scRNA-seq splits (e.g., Smart-seq2 training, 10x testing in Tabula Muris) and you must rank them by accuracy, F1-score, and unlabeling rate and render comparison figures.

## When NOT to use

- Only a single method is evaluated, so no cross-method ranking or grouped comparison is meaningful
- Labels are predicted without a verified ground-truth annotation, so accuracy/F1-score cannot be validly computed
- The evaluation target is running time or memory rather than prediction accuracy, F1-score and unlabeling rate

## Inputs

- Per-method, per-tissue prediction labels from AutomaticCellTypeIdentification runs on Tabula Muris 10x test tissues
- Manually annotated cell-type labels for the same cells
- Per-method metric values (accuracy, F1-score, unlabeling rate)

## Outputs

- cross_platform_performance.csv — per-method metric table (accuracy, F1-score, unlabeling rate per tissue and aggregated)
- Grouped comparison plot ranking methods across the seven test tissues (reproducing Fig. 2C)

## How to apply

For each method (e.g., clustifyr, scLearn, scHPL, SciBet, SingleR) and each test tissue, compare predictions against the manually annotated Tabula Muris labels and compute accuracy, the mean per-type harmonic mean of precision and recall (F1-score), and the unlabeling rate. Aggregate the metrics across the seven 10x test tissues, then rank methods per metric to expose which perform best — clustifyr, scLearn and scHPL for accuracy; scHPL, SciBet and SingleR for F1-score. Persist the per-method metric table (e.g., cross_platform_performance.csv) and render a grouped bar comparison plot per metric and tissue to reproduce figures such as Fig. 2C. Judge correctness by checking the reproduced ranking matches the reported top performers and that metric values are consistent between table and plot.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper that generates the per-method prediction outputs on Tabula Muris test data that the metrics and plots summarize) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **R** (Environment in which metric aggregation, CSV export and grouped plotting are performed)
- **Seurat** (Preprocessing pipeline (filtering, NormalizeData, marker finding) that yields the labeled matrices underlying the benchmark comparison)

## Examples

```
write.csv(per_method_metrics, "cross_platform_performance.csv"); ggplot(per_method_metrics, aes(method, accuracy, fill = tissue)) + geom_col(position = "dodge")
```

## Evaluation signals

- Reproduced accuracy ranking matches the reported top performers: clustifyr, scLearn and scHPL among eager learning methods
- Reproduced F1-score ranking matches the reported top performers: scHPL, SciBet and SingleR
- cross_platform_performance.csv contains one row per method per test tissue plus aggregated metrics across the seven 10x tissues
- F1-score values equal the mean per-type harmonic mean of precision and recall and are identical in table and plot
- Plot reproduces the grouped layout of Fig. 2C with consistent per-tissue and per-method grouping

## Limitations

- Rankings are specific to the Tabula Muris cross-platform setting (Smart-seq2 training on four tissues, 10x testing on seven); other datasets or platforms may reorder methods
- The evaluation emphasizes accuracy, F1-score, unlabeling rate and running time; other quality dimensions such as sensitivity and specificity are part of the overall performance score but not the headline ranking metrics
- Metrics depend on the upstream preprocessing thresholds (>=200 genes per cell, <=2 SD of mean expressed genes, <=20% mitochondrial reads, non-NA labels), so changes to filtering invalidate comparisons

## Evidence

- [abstract] we discuss and evaluate thirty-two published automatic methods for scRNA-seq data analysis in terms of their prediction accuracy, F1-score, unlabeling rate and running time: "Here, we discuss and evaluate thirty-two published automatic methods for scRNA-seq data analysis in terms of their prediction accuracy, F1-score, unlabeling rate and running time."
- [methods] Overall performance of automatic methods consists of accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity: "Overall performance of automatic methods consists of accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity"
- [results] clustifyr, scLearn and scHPL perform better than the other eager learning methods regarding accuracy, while scHPL, SciBet and SingleR have the highest F1-score.: [evidence span withheld — anchor status: not_found]
- [readme] Now, three interface of eagersupervised, lazysupervised, markersupervised methods supports the available automatic methods: [evidence span withheld — anchor status: not_found]
