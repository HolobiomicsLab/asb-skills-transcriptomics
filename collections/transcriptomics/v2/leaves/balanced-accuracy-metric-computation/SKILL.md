---
name: balanced-accuracy-metric-computation
description: Use when benchmarking a malignant-cell annotation method on single-cell
  datasets that have per-cell ground-truth malignant/normal labels, and you need per-method
  accuracy and balanced accuracy values for a comparison table.
when_to_use_negative:
- The dataset has no ground-truth malignant/normal labels — balanced accuracy cannot
  be computed and requires manual annotation or a different validation strategy
- The task is unsupervised tumor subcloning or CNV inference rather than binary malignant/normal
  classification — CopyKAT/inferCNV-style outputs on aneuploidy are not directly scored
  by cell-level accuracy against curated labels
- The prediction task is multi-class cell-type annotation (e.g. pan-cancer annotation
  with scATOMIC), where macro-averaged accuracy across the full label hierarchy is
  not equivalent to the two-class malignant/normal balanced accuracy defined here
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: scMalignantFinder
  role: Method under benchmarking; its scMalignantFinder_prediction ('Normal'/'Malignant')
    per-cell labels are scored for accuracy and balanced accuracy
  repo: github.com/Jonyyqn/scMalignantFinder
- name: PreCanCell
  role: Baseline ensemble-learning classifier whose malignant/non-malignant predictions
    are scored on the same 13 datasets
  repo: WangX-Lab/PreCanCell
- name: ikarus
  role: Baseline tumor/normal logistic-classifier pipeline whose predictions are scored
    on the same datasets
  repo: BIMSBbioinfo/ikarus
- name: Cancer-Finder
  role: Baseline cancer cell annotation method whose predictions are scored on the
    same datasets
  repo: Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: Baseline CNV-inference method whose tumor/normal cell separation is scored
    on the same datasets
  repo: navinlabcode/copykat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:e6cc53951334f550b4fae3cb96783bab86dd70b58b9a16e97cd82807ea1cfaf6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - Cancer-Finder
  - CopyKAT
  - PreCanCell
  - ikarus
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# balanced-accuracy-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes accuracy and balanced accuracy (plus the remaining benchmark metrics) for malignant-cell classifier predictions against per-cell ground-truth Normal/Malignant labels, enabling head-to-head comparison of tools like scMalignantFinder, PreCanCell, ikarus, Cancer-Finder, and CopyKAT on patient-derived scRNA-seq datasets.

## When to use

Use when benchmarking a malignant-cell annotation method on single-cell datasets that have per-cell ground-truth malignant/normal labels, and you need per-method accuracy and balanced accuracy values for a comparison table.

## When NOT to use

- The dataset has no ground-truth malignant/normal labels — balanced accuracy cannot be computed and requires manual annotation or a different validation strategy
- The task is unsupervised tumor subcloning or CNV inference rather than binary malignant/normal classification — CopyKAT/inferCNV-style outputs on aneuploidy are not directly scored by cell-level accuracy against curated labels
- The prediction task is multi-class cell-type annotation (e.g. pan-cancer annotation with scATOMIC), where macro-averaged accuracy across the full label hierarchy is not equivalent to the two-class malignant/normal balanced accuracy defined here

## Inputs

- Per-cell binary predictions ('Normal'/'Malignant') from scMalignantFinder's model.predict() output (AnnData with scMalignantFinder_prediction and malignancy_probability columns)
- Per-cell predictions from baseline tools (PreCanCell, ikarus, Cancer-Finder, CopyKAT) on the same datasets
- Ground-truth malignant/normal cell labels for each of the 13 patient-derived test scRNA-seq datasets (AnnData/.h5ad)

## Outputs

- Per-method, per-dataset accuracy and balanced accuracy values (seven evaluation metrics per method)
- benchmark_metrics_comparison.csv compiling per-method, per-metric results across the 13 datasets

## How to apply

For each test dataset, take the per-cell binary predictions ('Normal'/'Malignant') produced by each method — e.g. scMalignantFinder's scMalignantFinder_prediction column, or labels from PreCanCell, ikarus, Cancer-Finder, and CopyKAT run via their official repositories — and compare them against the ground-truth malignant/normal annotations. Compute accuracy as the fraction of correctly classified cells and balanced accuracy as the macro-average of per-class recall (sensitivity on malignant cells and specificity on normal cells), so that class imbalance in patient-derived tumors does not inflate scores. Compute the seven evaluation metrics per method per dataset, then average across the 13 test scRNA-seq datasets to obtain method-level scores. Validate by comparing against the published benchmark: scMalignantFinder 0.824 accuracy / 0.732 balanced accuracy vs PreCanCell (0.713/0.613), ikarus (0.446/0.533), Cancer-Finder (0.654/0.531), and CopyKAT (0.427/0.571); mismatches indicate a prediction-normalization or label-alignment error upstream.

## Related tools

- **scMalignantFinder** (Method under benchmarking; its scMalignantFinder_prediction ('Normal'/'Malignant') per-cell labels are scored for accuracy and balanced accuracy) — github.com/Jonyyqn/scMalignantFinder
- **PreCanCell** (Baseline ensemble-learning classifier whose malignant/non-malignant predictions are scored on the same 13 datasets) — WangX-Lab/PreCanCell
- **ikarus** (Baseline tumor/normal logistic-classifier pipeline whose predictions are scored on the same datasets) — BIMSBbioinfo/ikarus
- **Cancer-Finder** (Baseline cancer cell annotation method whose predictions are scored on the same datasets) — Patchouli-M/SequencingCancerFinder
- **CopyKAT** (Baseline CNV-inference method whose tumor/normal cell separation is scored on the same datasets) — navinlabcode/copykat

## Examples

```
from sklearn.metrics import balanced_accuracy_score, accuracy_score; acc = accuracy_score(y_true, y_pred); bacc = balanced_accuracy_score(y_true, y_pred)
```

## Evaluation signals

- Reproduced scMalignantFinder average accuracy across the 13 datasets must equal 0.824
- Reproduced scMalignantFinder average balanced accuracy must equal 0.732
- Baseline averages must match: PreCanCell 0.713/0.613, ikarus 0.446/0.533, Cancer-Finder 0.654/0.531, CopyKAT 0.427/0.571 (accuracy/balanced accuracy)
- All seven evaluation metrics must be populated for every method–dataset pair in benchmark_metrics_comparison.csv, with predictions aligned one-to-one to ground-truth cell barcodes
- Balanced accuracy should deviate from raw accuracy where class imbalance exists (e.g. ikarus 0.446 accuracy vs 0.533 balanced accuracy), confirming per-class recalls are computed correctly

## Limitations

- Balanced accuracy reflects the fixed ground-truth labels, which the paper notes were imperfect: 'the performance of these supervised learning methods would be greatly compromised by the lack of optimal reference datasets that accurately annotate the malignancy status of each cell'
- Metrics only cover cancerous lineages — the tool requires 'prerequisite identification of basic cell types', so non-tumor-lineage misclassifications are outside the scored scope
- Accuracy/balanced accuracy on these 13 datasets does not cover cancers with unknown or controversial origins, 'such as synovial sarcoma and certain brain tumors'

## Evidence

- [methods] Seven evaluation metrics computed per method against ground-truth labels: [evidence span withheld — anchor status: not_found]
- [results] Published benchmark values for all five methods: [evidence span withheld — anchor status: not_found]
- [intro] Motivating the per-cell evaluation task: "Identifying malignant cells and distinguishing them from other non-malignant cells is a crucial step in dissecting tumor heterogeneity from cancer scRNA-seq experiments"
- [readme] Baseline tool ikarus description from its README: [evidence span withheld — anchor status: external_doc]
