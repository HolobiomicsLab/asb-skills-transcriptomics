---
name: cross-method-benchmarking-of-cell-annotation-tools
description: Use when multiple malignant-cell classifiers (e.g. scMalignantFinder,
  PreCanCell, ikarus, Cancer-Finder, CopyKAT) must be compared on the same labeled
  scRNA-seq datasets to reproduce or verify reported sensitivity/specificity benchmarks.
when_to_use_negative:
- The goal is de novo malignant-cell annotation on a new dataset rather than quantitative
  comparison of classifiers against labeled ground truth.
- Input contains no cells of cancerous lineage or no prerequisite basic cell-type
  annotation — scMalignantFinder 'is designed to identify malignant cells from cancerous
  lineages, relying on the prerequisite identification of basic cell types.'
- 'The sample is normal tissue with no expected cancer cells: ''The default scATOMIC
  model assumes the presence of at least some cancer cells in a biopsy and should
  not be used in normal tissue applications.'''
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Primary classifier under test; run on both gold-standard datasets with norm_type=True
    to yield scMalignantFinder_prediction and malignancy_probability per cell
  repo: github.com/Jonyyqn/scMalignantFinder
- name: PreCanCell
  role: Competitor ensemble k-NN classifier benchmarked on the same two test datasets
  repo: WangX-Lab/PreCanCell
- name: ikarus
  role: Competitor logistic-classifier pipeline benchmarked on the same two test datasets
  repo: BIMSBbioinfo/ikarus
- name: Cancer-Finder
  role: Competitor deep-learning method benchmarked on the same two test datasets
  repo: Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: Competitor CNV-inference-based method benchmarked on the same two test datasets
  repo: navinlabcode/copykat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:2aa578e46bdf5f00d22a44d45b0019af1d42555631129dfc86301d92b3a4237f
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
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# cross-method-benchmarking-of-cell-annotation-tools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark automated malignant-cell annotation methods head-to-head on shared scRNA-seq test sets with ground-truth malignancy labels, computing sensitivity on malignant cells and specificity on normal epithelial cells to reproduce published performance tables.

## When to use

Use when multiple malignant-cell classifiers (e.g. scMalignantFinder, PreCanCell, ikarus, Cancer-Finder, CopyKAT) must be compared on the same labeled scRNA-seq datasets to reproduce or verify reported sensitivity/specificity benchmarks.

## When NOT to use

- The goal is de novo malignant-cell annotation on a new dataset rather than quantitative comparison of classifiers against labeled ground truth.
- Input contains no cells of cancerous lineage or no prerequisite basic cell-type annotation — scMalignantFinder 'is designed to identify malignant cells from cancerous lineages, relying on the prerequisite identification of basic cell types.'
- The sample is normal tissue with no expected cancer cells: 'The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used in normal tissue applications.'

## Inputs

- test_cancerCellLine.h5ad — 53,513 gold-standard malignant cells from 198 cancer cell lines (Zenodo record 17888140)
- test_TabulaSapiens.h5ad — 104,148 gold-standard normal epithelial cells from non-cancer donors (Zenodo record 17888140)
- Pretrained scMalignantFinder model files: model.joblib and ordered_feature.tsv

## Outputs

- benchmark_results.csv — sensitivity/specificity table for all five methods (scMalignantFinder, PreCanCell, ikarus, Cancer-Finder, CopyKAT)
- Per-cell predictions (scMalignantFinder_prediction and malignancy_probability) for each test dataset

## How to apply

Download the two gold-standard test sets from the scMalignantFinder Zenodo deposit (record 17888140): test_cancerCellLine.h5ad holding 53,513 malignant cells from 198 cancer cell lines and test_TabulaSapiens.h5ad holding 104,148 normal epithelial cells from non-cancer donors. Run scMalignantFinder with the pretrained model (model.joblib + ordered_feature.tsv) using norm_type=True so sc.pp.normalize_total(adata, target_sum=1e4) is applied, collecting scMalignantFinder_prediction and malignancy_probability per cell; run PreCanCell, ikarus, Cancer-Finder, and CopyKAT on the same two datasets with their published default pipelines. Compute sensitivity as the fraction of the 53,513 gold-standard malignant cells correctly called and specificity as the fraction of the 104,148 gold-standard normal epithelial cells correctly called for each method. Compare against the reported pairs — scMalignantFinder 1.000/0.786, PreCanCell 0.996/0.503, ikarus 0.834/0.642, Cancer-Finder 1.000/0.022, CopyKAT 0.594/0.397 — and write the full table to benchmark_results.csv.

## Related tools

- **scMalignantFinder** (Primary classifier under test; run on both gold-standard datasets with norm_type=True to yield scMalignantFinder_prediction and malignancy_probability per cell) — github.com/Jonyyqn/scMalignantFinder
- **PreCanCell** (Competitor ensemble k-NN classifier benchmarked on the same two test datasets) — WangX-Lab/PreCanCell
- **ikarus** (Competitor logistic-classifier pipeline benchmarked on the same two test datasets) — BIMSBbioinfo/ikarus
- **Cancer-Finder** (Competitor deep-learning method benchmarked on the same two test datasets) — Patchouli-M/SequencingCancerFinder
- **CopyKAT** (Competitor CNV-inference-based method benchmarked on the same two test datasets) — navinlabcode/copykat

## Examples

```
wget https://zenodo.org/records/17888140/files/test_cancerCellLine.h5ad && wget https://zenodo.org/records/17888140/files/test_TabulaSapiens.h5ad && pip install scMalignantFinder
```

## Evaluation signals

- Reproduced sensitivity/specificity for each method matches the reported pairs — scMalignantFinder 1.000/0.786; PreCanCell 0.996/0.503; ikarus 0.834/0.642; Cancer-Finder 1.000/0.022; CopyKAT 0.594/0.397 — within reasonable numerical tolerance.
- Sensitivity is computed over exactly 53,513 gold-standard malignant cells; specificity over exactly 104,148 gold-standard normal epithelial cells — cell counts should match the h5ad inputs.
- All five methods are evaluated on the identical two test sets (test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad) with the same ground-truth labels.
- benchmark_results.csv contains a complete 5-method sensitivity/specificity table with one row per method.
- scMalignantFinder outputs both a binary label (scMalignantFinder_prediction) and a continuous malignancy_probability per cell; absence of either column indicates a run failure.

## Limitations

- scMalignantFinder requires prerequisite basic cell-type identification and only identifies malignant cells within cancerous lineages.
- Capabilities are untested for cancers with unknown or controversial origins (e.g. synovial sarcoma and certain brain tumors).
- CNV-inference-based competitors such as CopyKAT have limitations in cancers with minimal genomic structural variation or unknown origins, and CNV inference often requires specifying normal reference cells, hindering full automation.

## Evidence

- [readme] ikarus pipeline description from README: [evidence span withheld — anchor status: external_doc]
- [readme] CopyKAT problem framing from README: [evidence span withheld — anchor status: external_doc]
