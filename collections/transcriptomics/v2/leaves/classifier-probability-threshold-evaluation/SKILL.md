---
name: classifier-probability-threshold-evaluation
description: Use when you have gold-standard labeled scRNA-seq data (e.g. test_cancerCellLine.h5ad,
  test_TabulaSapiens.h5ad) and need to quantify a classifier's sensitivity on malignant
  cells and specificity on normal epithelial cells against published benchmarks.
when_to_use_negative:
- The input is patient tumor data without prerequisite basic cell-type annotation
  — scMalignantFinder is designed to identify malignant cells only from cancerous
  lineages, relying on prior identification of basic cell types
- The cancer type has unknown or controversial origins (e.g. synovial sarcoma, certain
  brain tumors) — capabilities are explicitly flagged as requiring further testing
- The goal is malignant-region calling in spatial transcriptomics rather than per-cell
  scRNA-seq classification — that requires the multi-feature spatial workflow (image_cal,
  spatial_nn, region_identification), not this benchmark
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scMalignantFinder
  role: Probabilistic classifier under evaluation; outputs scMalignantFinder_prediction
    and malignancy_probability per cell
  repo: github.com/Jonyyqn/scMalignantFinder
- name: PreCanCell
  role: Competitor ensemble k-NN classifier benchmarked with default pipelines on
    the same gold-standard sets
  repo: github.com/WangX-Lab/PreCanCell
- name: ikarus
  role: Competitor gene-set-ranking + logistic classifier benchmarked with default
    pipelines
  repo: github.com/BIMSBbioinfo/ikarus
- name: Cancer-Finder
  role: Competitor domain-generalization cancer classifier benchmarked with default
    pipelines
  repo: github.com/Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: Competitor CNV-inference-based malignant/non-malignant separator benchmarked
    with default pipelines
  repo: github.com/navinlabcode/copykat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:8072bf1e7af6337c82e1bd4908b42e75a8b44f1a8b2ece8839623f7ab47da787
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# classifier-probability-threshold-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a probabilistic binary cell classifier (e.g. scMalignantFinder's malignancy_probability) against gold-standard scRNA-seq labels by computing sensitivity/specificity and comparing to reported baselines (PreCanCell, ikarus, Cancer-Finder, CopyKAT).

## When to use

Use when you have gold-standard labeled scRNA-seq data (e.g. test_cancerCellLine.h5ad, test_TabulaSapiens.h5ad) and need to quantify a classifier's sensitivity on malignant cells and specificity on normal epithelial cells against published benchmarks.

## When NOT to use

- The input is patient tumor data without prerequisite basic cell-type annotation — scMalignantFinder is designed to identify malignant cells only from cancerous lineages, relying on prior identification of basic cell types
- The cancer type has unknown or controversial origins (e.g. synovial sarcoma, certain brain tumors) — capabilities are explicitly flagged as requiring further testing
- The goal is malignant-region calling in spatial transcriptomics rather than per-cell scRNA-seq classification — that requires the multi-feature spatial workflow (image_cal, spatial_nn, region_identification), not this benchmark

## Inputs

- test_cancerCellLine.h5ad — gold-standard malignant cells from 198 cancer cell lines (Zenodo record 17888140)
- test_TabulaSapiens.h5ad — gold-standard normal epithelial cells from non-cancer donors (Zenodo record 17888140)
- Pretrained model.joblib and ordered_feature.tsv feature list (Zenodo record 17888140)
- Per-cell ground-truth malignancy labels for both datasets
- Competitor method pipelines: PreCanCell, ikarus, Cancer-Finder, CopyKAT

## Outputs

- Per-cell scMalignantFinder_prediction labels and malignancy_probability values (probability in [0,1])
- benchmark_results.csv — sensitivity/specificity table for all five methods (scMalignantFinder, PreCanCell, ikarus, Cancer-Finder, CopyKAT)

## How to apply

Load the gold-standard test sets from the Zenodo deposit (record 17888140): test_cancerCellLine.h5ad (53,513 malignant cells from 198 cancer cell lines) and test_TabulaSapiens.h5ad (104,148 normal epithelial cells from non-cancer donors). Run scMalignantFinder with the pretrained model.joblib plus ordered_feature.tsv using norm_type=True, which applies sc.pp.normalize_total(adata, target_sum=1e4), and collect per-cell scMalignantFinder_prediction and malignancy_probability. Run PreCanCell, ikarus, Cancer-Finder, and CopyKAT on the same two datasets with their published default pipelines. Compute sensitivity as the fraction of the 53,513 gold-standard malignant cells correctly called and specificity as the fraction of the 104,148 normal epithelial cells correctly called. Compare against the reported pairs (scMalignantFinder 1.000/0.786; PreCanCell 0.996/0.503; ikarus 0.834/0.642; Cancer-Finder 1.000/0.022; CopyKAT 0.594/0.397) and write the full five-method table to benchmark_results.csv.

## Related tools

- **scMalignantFinder** (Probabilistic classifier under evaluation; outputs scMalignantFinder_prediction and malignancy_probability per cell) — github.com/Jonyyqn/scMalignantFinder
- **PreCanCell** (Competitor ensemble k-NN classifier benchmarked with default pipelines on the same gold-standard sets) — github.com/WangX-Lab/PreCanCell
- **ikarus** (Competitor gene-set-ranking + logistic classifier benchmarked with default pipelines) — github.com/BIMSBbioinfo/ikarus
- **Cancer-Finder** (Competitor domain-generalization cancer classifier benchmarked with default pipelines) — github.com/Patchouli-M/SequencingCancerFinder
- **CopyKAT** (Competitor CNV-inference-based malignant/non-malignant separator benchmarked with default pipelines) — github.com/navinlabcode/copykat

## Examples

```
sc.pp.normalize_total(adata, target_sum=1e4); model.load(); result_adata = model.predict()
```

## Evaluation signals

- Reproduced scMalignantFinder sensitivity = 1.000 on the 53,513 cancer cell line cells and specificity = 0.786 on the 104,148 normal epithelial cells, within reasonable numerical tolerance
- Reproduced competitor pairs match reported values: PreCanCell 0.996/0.503, ikarus 0.834/0.642, Cancer-Finder 1.000/0.022, CopyKAT 0.594/0.397
- benchmark_results.csv contains sensitivity/specificity rows for all five methods, not just scMalignantFinder
- Every cell in both test sets receives a malignancy_probability value bounded in [0,1] and a binary scMalignantFinder_prediction
- Normalization checkpoint: input was normalized with sc.pp.normalize_total(adata, target_sum=1e4) (norm_type=True) before prediction

## Limitations

- scMalignantFinder is designed to identify malignant cells from cancerous lineages and requires prerequisite identification of basic cell types; it does not classify malignancy across all non-malignant cell types
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- Rival CNV-based methods have known scope limits (e.g. CopyKAT assumes aneuploidy; the scATOMIC default model assumes some cancer cells are present and should not be used on normal-tissue samples)

## Evidence

- [readme] Competitor classifier ikarus defines the tumor/normal discrimination task: [evidence span withheld — anchor status: external_doc]
