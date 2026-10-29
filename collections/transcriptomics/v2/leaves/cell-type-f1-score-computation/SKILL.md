---
name: cell-type-f1-score-computation
description: Use when comparing label-transfer accuracy of scRNA-seq/scATAC-seq integration
  methods on cells with held-out ground-truth annotations, especially under varying
  annotation budgets.
when_to_use_negative:
- Target cells lack held-out ground-truth annotations, so no reference labels exist
  for F1 computation
- The task measures joint embedding quality (clustering/visualization) rather than
  label transfer, where F1 against reference labels is not the appropriate metric
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides the classification metrics used to compute per-cell-type label transfer
    F1 scores
- name: scBridge
  role: Label-transfer method whose predictions are evaluated by F1 against held-out
    target annotations
  repo: https://github.com/XLearning-SCU/scBridge
- name: scJoint
  role: Semi-supervised baseline whose label-transfer predictions are evaluated by
    F1 for comparison
  repo: https://github.com/SydneyBioX/scJoint
- name: Python
  role: Implementation language for the F1 evaluation scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:aac6d1a5aaf69df4c88bd66484751792e99903d8aaab22eb4da6abb30b54887c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - Python
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Cell-type F1 score computation

## Summary

Computes average per-cell-type label-transfer F1 scores against held-out target cell annotations to quantitatively compare multi-omics integration methods (e.g., scBridge vs. scJoint), as done in the scBridge benchmark across annotation downsampling fractions.

## When to use

Use when comparing label-transfer accuracy of scRNA-seq/scATAC-seq integration methods on cells with held-out ground-truth annotations, especially under varying annotation budgets.

## When NOT to use

- Target cells lack held-out ground-truth annotations, so no reference labels exist for F1 computation
- The task measures joint embedding quality (clustering/visualization) rather than label transfer, where F1 against reference labels is not the appropriate metric

## Inputs

- Predicted cell-type labels for target scATAC-seq cells from each integration method
- Held-out ground-truth cell-type annotations for the target scATAC-seq cells
- Results for each annotation budget fraction (100%, 75%, 50%, 25%) and each method (scBridge, scJoint)

## Outputs

- Average per-cell-type label-transfer F1 score for each method and annotation fraction
- Aggregated results table, e.g., kidney_label_transfer_f1.csv

## How to apply

After each method (scBridge via `python main.py` with its default source 'Standard' / target 'TFIDF' preprocessing and reliability_threshold 0.95, and scJoint from github.com/SydneyBioX/scJoint) produces predicted labels for the target scATAC-seq cells, treat the original target annotations as held-out ground truth. Compute per-cell-type F1 with scikit-learn metrics and average F1 across cell types. Repeat at each annotation downsampling fraction (100%, 75%, 50%, 25% annotated scRNA-seq cells) to assemble a results table such as kidney_label_transfer_f1.csv. Correctness is judged by reproducing the reported trend: scBridge 77.35% at full annotation to 77.08% at 25%, versus scJoint dropping from 73.22% to 59.36% on the 10x Multiome mouse kidney benchmark (Fig. 2g).

## Related tools

- **scikit-learn** (Provides the classification metrics used to compute per-cell-type label transfer F1 scores)
- **scBridge** (Label-transfer method whose predictions are evaluated by F1 against held-out target annotations) — https://github.com/XLearning-SCU/scBridge
- **scJoint** (Semi-supervised baseline whose label-transfer predictions are evaluated by F1 for comparison) — https://github.com/SydneyBioX/scJoint
- **Python** (Implementation language for the F1 evaluation scripts)

## Examples

```
from sklearn.metrics import f1_score; f1_per_type = f1_score(y_true, y_pred, average=None); avg_f1 = f1_per_type.mean()
```

## Evaluation signals

- Average F1 at full annotation reproduces scBridge 77.35% and scJoint 73.22% on the 10x Multiome mouse kidney benchmark
- At 25% annotated scRNA-seq cells, scBridge F1 stays near full-data performance (77.08%) while scJoint drops sharply (59.36%)
- Per-cell-type F1 values are computed for every cell type, then averaged — not micro/macro over pooled cells only
- Same subsampled inputs and held-out target annotations are used for both methods so F1 differences are attributable to the methods
- Results table kidney_label_transfer_f1.csv contains one F1 value per method × annotation-fraction combination

## Limitations

- F1 evaluation requires reliable held-out target annotations; datasets with 'Unknown' annotations require exclusion before scoring
- Reported F1 values are specific to the 10x Multiome mouse kidney dataset and the chosen downsampling fractions; results may differ on other tissues or annotation budgets

## Evidence

- [full_text] Finding: annotation downsampling F1 trend: [evidence span withheld — anchor status: not_found]
- [full_text] Metric definition in the workflow: [evidence span withheld — anchor status: not_found]
- [full_text] Reproduction verification target: [evidence span withheld — anchor status: not_found]
- [discussion] Baseline adoption from official code: "we adopted the official code released in https://github.com/SydneyBioX/scJoint"
