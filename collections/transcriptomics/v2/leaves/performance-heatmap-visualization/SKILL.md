---
name: performance-heatmap-visualization
description: Use when comparing automatic cell-type identification methods across
  tissues/datasets and the per-method, per-tissue metrics (accuracy, F1-score, unlabeling
  rate) from self-projection cross-validation need to be summarized as a comparable
  heatmap.
when_to_use_negative:
- You only have a single method's metrics on a single dataset — no method-by-tissue
  comparison matrix can be formed
- The evaluation task is running-time or sensitivity/specificity comparison rather
  than accuracy, F1-score and unlabeling rate from self-projection
- Metrics are not bounded in [0, 1] (e.g., raw running times), so the paper's validation
  range check does not apply
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: AutomaticCellTypeIdentification
  role: R package whose self-projection function produces the per-method, per-tissue
    accuracy, F1-score and unlabeling-rate values visualized in the heatmap
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret
  role: Provides the createFolds function used to generate the five cell-type-stratified
    cross-validation folds underlying the averaged metrics
- name: R
  role: Environment in which the self-projection evaluation is run and the metric
    matrices are assembled and plotted as heatmaps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:44743e2c908c46967e4cf12f2da3ec8085fd32d74996805244c012c0416de2dc
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
  - caret
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# performance-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles accuracy, mean per-cell-type F1-score, and unlabeling-rate values (averaged over five cross-validation rounds) into a method-by-tissue matrix and visualizes it as a heatmap, mirroring the paper's Fig. 2B self-projection evaluation layout for the eleven Tabula Muris tissues.

## When to use

Use when comparing automatic cell-type identification methods across tissues/datasets and the per-method, per-tissue metrics (accuracy, F1-score, unlabeling rate) from self-projection cross-validation need to be summarized as a comparable heatmap.

## When NOT to use

- You only have a single method's metrics on a single dataset — no method-by-tissue comparison matrix can be formed
- The evaluation task is running-time or sensitivity/specificity comparison rather than accuracy, F1-score and unlabeling rate from self-projection
- Metrics are not bounded in [0, 1] (e.g., raw running times), so the paper's validation range check does not apply

## Inputs

- Per-method, per-tissue accuracy values computed on labeled cells
- Mean per-cell-type F1-score values from five-fold cross-validation
- Unlabeling-rate values per method and tissue
- Method-by-tissue metric matrices averaged over five cross-validation rounds

## Outputs

- Heatmap of mean accuracy by method and tissue (Fig. 2B layout)
- Heatmap of mean F1-score by method and tissue
- Heatmap of mean unlabeling rate by method and tissue

## How to apply

Run each automatic cell-type identification method through the AutomaticCellTypeIdentification package's self-projection function using five cell-type-stratified folds created with caret's createFolds, training on four subsets and testing on the held-out subset over five rounds. For each method and tissue compute accuracy on labeled cells, the mean per-cell-type F1-score, and the unlabeling rate, then average each metric across the five cross-validation rounds to suppress fold-to-fold variance. Assemble the three averaged metrics into method-by-tissue matrices (one row per evaluated method, one column per retained tissue) and render each as a heatmap in the Fig. 2B layout so method-level biases toward specific tissues become visually apparent. Validate that all cell values fall within the 0–1 range expected for accuracy, F1-score and unlabeling rate before interpreting.

## Related tools

- **AutomaticCellTypeIdentification** (R package whose self-projection function produces the per-method, per-tissue accuracy, F1-score and unlabeling-rate values visualized in the heatmap) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret** (Provides the createFolds function used to generate the five cell-type-stratified cross-validation folds underlying the averaged metrics)
- **R** (Environment in which the self-projection evaluation is run and the metric matrices are assembled and plotted as heatmaps)

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- The heatmap file exists and contains one row per evaluated method and one column per each of the eleven retained Tabula Muris tissues
- All plotted metric values fall within the 0–1 range expected for accuracy, F1-score and unlabeling rate
- Each cell in the matrix is the mean of the metric over five cross-validation rounds, not a single-fold value
- The layout matches the paper's Fig. 2B heatmap arrangement of methods (rows) by tissues (columns)
- Methods generally show high accuracy and F1-score with low unlabeling rate across tissues except marrow, reproducing the reported finding

## Limitations

- Only tissues with both 10x and Smart-seq2 data are included; aorta, brain myeloid, brain non-myeloid, diaphragm, fat, heart, large intestine and skin are excluded, so the heatmap covers eleven tissues
- Marrow is an outlier where most methods do not achieve the high accuracy/low unlabeling pattern seen in other tissues
- Preprocessing exclusions (cells with fewer than 200 expressed genes, >2 SD above the mean gene count, >20% mitochondrial expression, or no cell-type label) shape the underlying metrics and must match the protocol for comparability

## Evidence
