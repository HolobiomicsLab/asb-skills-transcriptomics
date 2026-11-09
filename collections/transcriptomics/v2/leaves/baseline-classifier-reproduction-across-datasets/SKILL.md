---
name: baseline-classifier-reproduction-across-datasets
description: Use when you need to reproduce or extend a head-to-head comparison of
  malignant-cell identification tools on .h5ad scRNA-seq datasets that carry ground-truth
  malignant/normal labels, and report per-method accuracy and balanced accuracy.
when_to_use_negative:
- The goal is de novo malignant-cell annotation on unannotated data — scMalignantFinder
  relies on the prerequisite identification of basic cell types and is restricted
  to cancerous lineages
- The cancer type has unknown or controversial cell of origin (e.g. synovial sarcoma,
  certain brain tumors) — the tool's capabilities are untested in these settings
- Only spatial transcriptomics malignant-region identification is needed — this skill
  covers the scRNA-seq single-cell classification benchmark, not the ST multi-feature
  clustering workflow
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Method under test; provides scMalignantFinder_prediction and malignancy_probability
    per cell
  repo: github.com/Jonyyqn/scMalignantFinder
- name: PreCanCell
  role: Baseline classifier run on the same 13 datasets via its official repository
  repo: github.com/WangX-Lab/PreCanCell
- name: ikarus
  role: Baseline tumor/normal classifier run on the same 13 datasets via its official
    repository
  repo: github.com/BIMSBbioinfo/ikarus
- name: Cancer-Finder
  role: Baseline cancer-cell annotation tool run on the same 13 datasets via its official
    repository
  repo: github.com/Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: CNV-based baseline run on the same 13 datasets via its official repository
  repo: github.com/navinlabcode/copykat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:46b97e327f5fb85ac968595390817fa6623d02e30d42c541dcd19ec8c577a773
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# baseline-classifier-reproduction-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces a multi-tool benchmark of malignant-cell classifiers by running scMalignantFinder and four baselines (PreCanCell, ikarus, Cancer-Finder, CopyKAT) on the same 13 patient-derived scRNA-seq datasets and comparing their accuracy and balanced accuracy against reported values.

## When to use

Use when you need to reproduce or extend a head-to-head comparison of malignant-cell identification tools on .h5ad scRNA-seq datasets that carry ground-truth malignant/normal labels, and report per-method accuracy and balanced accuracy.

## When NOT to use

- The goal is de novo malignant-cell annotation on unannotated data — scMalignantFinder relies on the prerequisite identification of basic cell types and is restricted to cancerous lineages
- The cancer type has unknown or controversial cell of origin (e.g. synovial sarcoma, certain brain tumors) — the tool's capabilities are untested in these settings
- Only spatial transcriptomics malignant-region identification is needed — this skill covers the scRNA-seq single-cell classification benchmark, not the ST multi-feature clustering workflow

## Inputs

- 13 patient-derived scRNA-seq test datasets in AnnData/.h5ad format with ground-truth malignant/normal cell labels
- Pretrained scMalignantFinder model.joblib from Zenodo record 17888140
- Ordered feature list ordered_feature.tsv from Zenodo record 17888140
- Official baseline tool repositories (PreCanCell, ikarus, Cancer-Finder, CopyKAT)

## Outputs

- Per-cell scMalignantFinder_prediction labels ('Normal'/'Malignant') and malignancy_probability values for each dataset
- Per-method, per-metric benchmark_metrics_comparison.csv containing accuracy and balanced accuracy (plus five other evaluation metrics) across all 13 datasets

## How to apply

Download the pretrained model.joblib and ordered_feature.tsv from Zenodo record 17888140 into a pretrained_model directory, then pip install scMalignantFinder. Preprocess each AnnData/.h5ad test set with total-count normalization only (sc.pp.normalize_total with target_sum=1e4, i.e. norm_type=True, skipping log-transformation), instantiate the model with pretrain_dir so the model and ordered feature list load automatically, then call model.load() and model.predict() to obtain scMalignantFinder_prediction ('Normal'/'Malignant') and per-cell malignancy_probability. Run the four baselines on the identical 13 datasets using their official repositories (WangX-Lab/PreCanCell, BIMSBbioinfo/ikarus, Cancer-Finder/SequencingCancerFinder, navinlabcode/copykat). Compute the seven evaluation metrics, including accuracy and balanced accuracy, against ground-truth labels for each method, and compile a per-method, per-metric benchmark_metrics_comparison.csv.

## Related tools

- **scMalignantFinder** (Method under test; provides scMalignantFinder_prediction and malignancy_probability per cell) — github.com/Jonyyqn/scMalignantFinder
- **PreCanCell** (Baseline classifier run on the same 13 datasets via its official repository) — github.com/WangX-Lab/PreCanCell
- **ikarus** (Baseline tumor/normal classifier run on the same 13 datasets via its official repository) — github.com/BIMSBbioinfo/ikarus
- **Cancer-Finder** (Baseline cancer-cell annotation tool run on the same 13 datasets via its official repository) — github.com/Patchouli-M/SequencingCancerFinder
- **CopyKAT** (CNV-based baseline run on the same 13 datasets via its official repository) — github.com/navinlabcode/copykat

## Examples

```
pip install scMalignantFinder; sc.pp.normalize_total(adata, target_sum=1e4); model = scMalignantFinder(pretrain_dir='pretrained_model'); model.load(); result_adata = model.predict()
```

## Evaluation signals

- Reproduced scMalignantFinder average accuracy is 0.824 and average balanced accuracy is 0.732 across the 13 datasets, matching the article's reported benchmark values
- Baseline values match the article: PreCanCell 0.713/0.613, ikarus 0.446/0.533, Cancer-Finder 0.654/0.531, CopyKAT 0.427/0.571 (accuracy/balanced accuracy)
- Prediction output contains both required columns: scMalignantFinder_prediction with values 'Normal'/'Malignant' and a continuous malignancy_probability per cell
- Input preprocessing applied total-count normalization (sc.pp.normalize_total, target_sum=1e4) with no log-transformation, matching norm_type=True behavior
- Results table contains seven metrics per method per dataset, compiled into benchmark_metrics_comparison.csv

## Limitations

- The tool requires prerequisite identification of basic cell types and is designed to identify malignant cells only from cancerous lineages
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- Existing supervised methods' performance is compromised by lack of optimal reference datasets that accurately annotate malignancy status; baseline CopyKAT/ikarus scores in this benchmark are substantially lower than scMalignantFinder's

## Evidence

- [methods] Normalization without log transform: "`norm_type=True`: applies `sc.pp.normalize_total(adata, target_sum=1e4)` to the input data"
- [methods] Prediction API: "model.load()
result_adata = model.predict()"
- [methods] Model resources on Zenodo: "All resources required for `scMalignantFinder` have been deposited on [Zenodo](https://zenodo.org/records/17888140)"
- [discussion] Baseline tools compared across datasets: "including ikarus, PreCanCell, Cancer-Finder, and CopyKAT, across multiple datasets from cancer cell lines, non-cancer tissues, and various patient-derived cancer types."
- [discussion] Requires basic cell type annotation first: "It is designed to identify malignant cells from cancerous lineages, relying on the prerequisite identification of basic cell types."
- [intro] Tool purpose and scope: "specially designed to distinguish malignant cell from their originating normal cells, rather than from all other non-malignant cells as existing methods usually do"
