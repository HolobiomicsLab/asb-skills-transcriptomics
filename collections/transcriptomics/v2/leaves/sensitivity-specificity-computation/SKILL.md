---
name: sensitivity-specificity-computation
description: Use when reproducing or validating malignant-cell-identification benchmarks
  on labeled scRNA-seq test sets (e.g., test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad)
  where ground-truth malignant/normal labels exist and method sensitivity/specificity
  must be quantified.
when_to_use_negative:
- The dataset lacks ground-truth malignant/normal labels, so sensitivity and specificity
  cannot be computed — use unsupervised evaluation instead.
- The goal is patient-derived tumor annotation rather than the gold-standard cell-line/non-cancer-donor
  benchmark; the article notes scMalignantFinder is designed to identify malignant
  cells from cancerous lineages only.
- Analyzing cancers with unknown or controversial origins (e.g., synovial sarcoma,
  certain brain tumors), where the tool's capabilities are explicitly untested.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMalignantFinder
  role: Primary classifier under benchmarking; run with pretrained model.joblib +
    ordered_feature.tsv and norm_type=True (target_sum=1e4)
  repo: github.com/Jonyyqn/scMalignantFinder
- name: PreCanCell
  role: Competitor method benchmarked on the same gold-standard datasets using its
    published default ensemble k-NN pipeline
  repo: github.com/WangX-Lab/PreCanCell
- name: ikarus
  role: Competitor method benchmarked on the same gold-standard datasets using its
    published default logistic-classifier pipeline
  repo: github.com/BIMSBbioinfo/ikarus
- name: Cancer-Finder
  role: Competitor method benchmarked on the same gold-standard datasets using its
    published default pipeline
  repo: github.com/Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: Competitor CNV-based method benchmarked on the same gold-standard datasets
    using its published default pipeline
  repo: github.com/navinlabcode/copykat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:6c72e8cbe828b31b91ae6ed360b8e22f650e660f5ff906928cbab669c1f661e8
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
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sensitivity-specificity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for computing sensitivity and specificity of malignant-cell classifiers against gold-standard scRNA-seq labels (cancer cell lines vs. non-cancer donor epithelium) and comparing the reproduced values with reported benchmarks.

## When to use

Use when reproducing or validating malignant-cell-identification benchmarks on labeled scRNA-seq test sets (e.g., test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad) where ground-truth malignant/normal labels exist and method sensitivity/specificity must be quantified.

## When NOT to use

- The dataset lacks ground-truth malignant/normal labels, so sensitivity and specificity cannot be computed — use unsupervised evaluation instead.
- The goal is patient-derived tumor annotation rather than the gold-standard cell-line/non-cancer-donor benchmark; the article notes scMalignantFinder is designed to identify malignant cells from cancerous lineages only.
- Analyzing cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors), where the tool's capabilities are explicitly untested.

## Inputs

- test_cancerCellLine.h5ad (gold-standard malignant cells, 53,513 cells from 198 cancer cell lines)
- test_TabulaSapiens.h5ad (gold-standard normal epithelial cells, 104,148 cells from non-cancer donors)
- Pretrained model files from Zenodo record 17888140: model.joblib and ordered_feature.tsv
- Ground-truth malignancy labels per cell

## Outputs

- Per-cell predictions (scMalignantFinder_prediction, malignancy_probability) for each method on both test sets
- benchmark_results.csv — sensitivity/specificity table for scMalignantFinder, PreCanCell, ikarus, Cancer-Finder, and CopyKAT

## How to apply

Download the gold-standard test data from Zenodo record 17888140 (test_cancerCellLine.h5ad with 53,513 malignant cells from 198 cancer cell lines; test_TabulaSapiens.h5ad with 104,148 normal epithelial cells from non-cancer donors). Run scMalignantFinder on each dataset with the pretrained model.joblib + ordered_feature.tsv using norm_type=True (sc.pp.normalize_total with target_sum=1e4), collecting scMalignantFinder_prediction and malignancy_probability per cell; run PreCanCell, ikarus, Cancer-Finder, and CopyKAT with their published default pipelines on the same data. Compute sensitivity as the fraction of the 53,513 gold-standard malignant cells correctly called and specificity as the fraction of the 104,148 normal epithelial cells correctly called for each method. Validate by comparing to the reported pairs — scMalignantFinder 1.000/0.786, PreCanCell 0.996/0.503, ikarus 0.834/0.642, Cancer-Finder 1.000/0.022, CopyKAT 0.594/0.397 — within reasonable numerical tolerance, then write the five-method table to benchmark_results.csv.

## Related tools

- **scMalignantFinder** (Primary classifier under benchmarking; run with pretrained model.joblib + ordered_feature.tsv and norm_type=True (target_sum=1e4)) — github.com/Jonyyqn/scMalignantFinder
- **PreCanCell** (Competitor method benchmarked on the same gold-standard datasets using its published default ensemble k-NN pipeline) — github.com/WangX-Lab/PreCanCell
- **ikarus** (Competitor method benchmarked on the same gold-standard datasets using its published default logistic-classifier pipeline) — github.com/BIMSBbioinfo/ikarus
- **Cancer-Finder** (Competitor method benchmarked on the same gold-standard datasets using its published default pipeline) — github.com/Patchouli-M/SequencingCancerFinder
- **CopyKAT** (Competitor CNV-based method benchmarked on the same gold-standard datasets using its published default pipeline) — github.com/navinlabcode/copykat

## Examples

```
result = model.predict(); sens = (result.scMalignantFinder_prediction[y==1]==1).mean(); spec = (result.scMalignantFinder_prediction[y==0]==0).mean()
```

## Evaluation signals

- Reproduced sensitivity/specificity for scMalignantFinder matches the reported 1.000/0.786 within reasonable numerical tolerance.
- Reproduced competitor values match reported benchmarks: PreCanCell 0.996/0.503, ikarus 0.834/0.642, Cancer-Finder 1.000/0.022, CopyKAT 0.594/0.397.
- Sensitivity is computed over exactly 53,513 gold-standard malignant cells and specificity over exactly 104,148 gold-standard normal epithelial cells.
- benchmark_results.csv contains a complete five-method sensitivity/specificity table.
- scMalignantFinder predictions on test data include per-cell malignancy_probability values after normalize_total(target_sum=1e4) preprocessing.

## Limitations

- scMalignantFinder is designed to identify malignant cells from cancerous lineages and relies on the prerequisite identification of basic cell types.
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors.
- Large CNV-based competitor differences (e.g., Cancer-Finder specificity of 0.022 on non-cancer tissue) indicate method-specific failure modes on non-cancer donor data that must be reproduced, not averaged away.

## Evidence

- [results] On the pooled gold-standard datasets, scMalignantFinder achieved a sensitivity of 1.000 (cancer cell lines) and a specificity of 0.786 (non-cancer tissues), outperforming PreCanCell, ikarus, Cancer-Finder, and CopyKAT.: [evidence span withheld — anchor status: not_found]
