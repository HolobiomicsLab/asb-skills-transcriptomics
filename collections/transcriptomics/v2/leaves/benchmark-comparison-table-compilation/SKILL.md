---
name: benchmark-comparison-table-compilation
description: Use when several malignant-cell classifiers must be evaluated on identical
  scRNA-seq datasets with shared ground-truth labels and their metrics need compiling
  into one reproducible comparison table.
when_to_use_negative:
- Only one method's output exists and no head-to-head comparison against PreCanCell/ikarus/Cancer-Finder/CopyKAT
  is intended
- Datasets lack ground-truth malignant/normal labels, so accuracy and balanced accuracy
  cannot be computed against a reference
- The input is spatial transcriptomics spot data without per-cell ground truth — scMalignantFinder
  region identification rather than cell-level benchmarking is the relevant workflow
tools:
- name: scMalignantFinder
  role: Method under benchmarking; predictions obtained via model.load() and model.predict()
    after total-count normalization
  repo: github.com/Jonyyqn/scMalignantFinder
- name: PreCanCell
  role: Baseline classifier run on the same 13 datasets using its official repository
  repo: WangX-Lab/PreCanCell
- name: ikarus
  role: Baseline classifier run on the same 13 datasets using its official repository
  repo: BIMSBbioinfo/ikarus
- name: Cancer-Finder
  role: Baseline classifier run on the same 13 datasets using its official repository
  repo: Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: Baseline CNV-based classifier run on the same 13 datasets using its official
    repository
  repo: navinlabcode/copykat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:a9c1616647751898b23258a6046c836a61dd9aca84fd6edb8a34a44563aaeaae
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# benchmark-comparison-table-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compiles head-to-head benchmark results for malignant-cell identification methods (scMalignantFinder vs PreCanCell, ikarus, Cancer-Finder, CopyKAT) into a per-method, per-metric table (benchmark_metrics_comparison.csv) across the same 13 patient-derived scRNA-seq test datasets, using accuracy and balanced accuracy as headline metrics.

## When to use

Use when several malignant-cell classifiers must be evaluated on identical scRNA-seq datasets with shared ground-truth labels and their metrics need compiling into one reproducible comparison table.

## When NOT to use

- Only one method's output exists and no head-to-head comparison against PreCanCell/ikarus/Cancer-Finder/CopyKAT is intended
- Datasets lack ground-truth malignant/normal labels, so accuracy and balanced accuracy cannot be computed against a reference
- The input is spatial transcriptomics spot data without per-cell ground truth — scMalignantFinder region identification rather than cell-level benchmarking is the relevant workflow

## Inputs

- 13 patient-derived scRNA-seq test datasets (AnnData/.h5ad) with ground-truth malignant/normal cell labels
- Per-cell prediction labels from scMalignantFinder ('Normal'/'Malignant' plus malignancy_probability)
- Per-cell prediction labels from PreCanCell, ikarus, Cancer-Finder, and CopyKAT run via their official repositories
- Pretrained model.joblib and ordered_feature.tsv from Zenodo record 17888140 for the scMalignantFinder run

## Outputs

- benchmark_metrics_comparison.csv with per-method, per-metric rows (seven evaluation metrics including accuracy and balanced accuracy) across the 13 datasets

## How to apply

Run every method (scMalignantFinder plus the four baselines) on the same 13 test AnnData/.h5ad datasets, preprocessing each identically — e.g. total-count normalization via sc.pp.normalize_total(adata, target_sum=1e4) with norm_type=True and no log-transformation, and loading the pretrained model.joblib with ordered_feature.tsv for scMalignantFinder. For each method, call model.load() then model.predict() to obtain per-cell 'Normal'/'Malignant' labels and malignancy_probability, and run baselines from their official repositories. Compute seven evaluation metrics per method against ground-truth malignant/normal labels — at minimum accuracy and balanced accuracy — per dataset and averaged. Compile the per-method, per-metric rows into benchmark_metrics_comparison.csv and validate against the reported figures: scMalignantFinder 0.824 accuracy / 0.732 balanced accuracy, PreCanCell 0.713/0.613, ikarus 0.446/0.533, Cancer-Finder 0.654/0.531, CopyKAT 0.427/0.571.

## Related tools

- **scMalignantFinder** (Method under benchmarking; predictions obtained via model.load() and model.predict() after total-count normalization) — github.com/Jonyyqn/scMalignantFinder
- **PreCanCell** (Baseline classifier run on the same 13 datasets using its official repository) — WangX-Lab/PreCanCell
- **ikarus** (Baseline classifier run on the same 13 datasets using its official repository) — BIMSBbioinfo/ikarus
- **Cancer-Finder** (Baseline classifier run on the same 13 datasets using its official repository) — Patchouli-M/SequencingCancerFinder
- **CopyKAT** (Baseline CNV-based classifier run on the same 13 datasets using its official repository) — navinlabcode/copykat

## Examples

```
df['accuracy'] = [accuracy_score(t,p) for t,p in zip(df.truth, df.pred)]; df['balanced_accuracy'] = [balanced_accuracy_score(t,p) for t,p in zip(df.truth, df.pred)]; df.to_csv('benchmark_metrics_comparison.csv', index=False)
```

## Evaluation signals

- Reproduced scMalignantFinder averages across the 13 datasets equal accuracy 0.824 and balanced accuracy 0.732
- Baseline averages match reported values: PreCanCell 0.713/0.613, ikarus 0.446/0.533, Cancer-Finder 0.654/0.531, CopyKAT 0.427/0.571
- benchmark_metrics_comparison.csv contains rows for all five methods with all seven evaluation metrics, per dataset and averaged
- All methods were applied to the same 13 preprocessed datasets (identical normalization, no log-transformation) so metric differences reflect the classifiers, not preprocessing
- Per-cell predictions are binary 'Normal'/'Malignant' labels directly comparable with ground-truth annotations

## Limitations

- CopyKAT and other CNV-inference baselines are limited for cancers with minimal genomic structural variation, and CNV methods require specification of appropriate normal reference cells
- Benchmark conclusions are specific to patient-derived datasets with ground-truth malignant/normal labels; scMalignantFinder capabilities are untested for cancers with unknown or controversial origins such as synovial sarcoma and certain brain tumors
- scMalignantFinder requires prerequisite identification of basic cell types and only identifies malignant cells within cancerous lineages, so baselines must be compared under the same cell-type prefiltering assumptions

## Evidence

- [readme] ikarus baseline description: [evidence span withheld — anchor status: external_doc]
