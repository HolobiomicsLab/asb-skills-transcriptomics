---
name: scrna-seq-training-dataset-construction
description: Use when building a supervised malignant-vs-normal epithelial training
  set from labeled scRNA-seq datasets where the raw 'Malignant'/'Tumor' annotations
  may be noisy and need refinement via cancer signature activity.
when_to_use_negative:
- Your goal is malignant-cell prediction on new data, not training-set curation —
  use the pretrained model.joblib predict step instead of rebuilding the training
  set.
- The dataset lacks reliable basic cell-type/epithelial annotations, since the baseline
  is defined on 'Normal'-annotated epithelial cells and the tool relies on prerequisite
  identification of basic cell types.
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma or
  certain brain tumors), where the curated signatures' transcriptional patterns have
  not been validated.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Provides utils.aucell_cal to score the nine curated cancer gene signatures
    for training-set calibration and supplies the combined training data
  repo: https://github.com/Jonyyqn/scMalignantFinder
- name: pySCENIC
  role: AUCell implementation used under the hood to compute per-cell signature activity
    scores
  repo: https://github.com/aertslab/pySCENIC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:739db2d3cb176d7fbdb3343973e42282b87d351a5a03b86b2703817f391d52a8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  repo_url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - pySCENIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# scrna-seq-training-dataset-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a signature-calibrated scRNA-seq training set for malignant-cell classifiers by scoring nine curated pan-cancer gene signatures (AUCell) on originally annotated malignant cells and removing cells inconsistent with normal-epithelial baseline signature activity. In scMalignantFinder this yielded a calibrated set of 416,774 cells (134,053 malignant, 282,721 normal epithelial).

## When to use

Use when building a supervised malignant-vs-normal epithelial training set from labeled scRNA-seq datasets where the raw 'Malignant'/'Tumor' annotations may be noisy and need refinement via cancer signature activity.

## When NOT to use

- Your goal is malignant-cell prediction on new data, not training-set curation — use the pretrained model.joblib predict step instead of rebuilding the training set.
- The dataset lacks reliable basic cell-type/epithelial annotations, since the baseline is defined on 'Normal'-annotated epithelial cells and the tool relies on prerequisite identification of basic cell types.
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma or certain brain tumors), where the curated signatures' transcriptional patterns have not been validated.

## Inputs

- combine_training.h5ad (Zenodo record 17888140) with raw labels in adata.obs['Raw_annotation']
- Nine curated pan-cancer gene signatures as a GMT file (sc_malignant_deg.gmt from Zenodo record 17888140)
- Raw single-cell RNA-seq count matrices from the five training datasets (both 'Malignant'/'Tumor' and 'Normal' epithelial annotations required)

## Outputs

- calibrated_training_set.h5ad (416,774 cells: 134,053 signature-consistent malignant, 282,721 normal epithelial) with Raw_annotation labels preserved
- Per-cell AUCell scores for the nine upregulated/downregulated cancer signatures
- The excluded ~2.5% of malignant cells failing signature-consistency, available for diagnostic comparison (Supplementary Fig. 2a)

## How to apply

Load the combined training data (combine_training.h5ad from Zenodo record 17888140) and read original labels from adata.obs['Raw_annotation'], treating both 'Malignant' and 'Tumor' as malignant. Compute per-cell AUCell scores for the nine curated pan-cancer gene sets (upregulated and downregulated malignant signatures) across the five training datasets using scMalignantFinder's utils.aucell_cal, which wraps pySCENIC's AUCell. For each signature, average AUCell scores among 'Normal'-annotated epithelial cells to establish a normal-cell baseline. Retain malignant cells whose upregulated-signature activity is higher and/or downregulated-signature activity is lower than that normal baseline; this excludes roughly 2.5% of the originally annotated malignant cells that fail the signature-consistency criterion. Combine the retained malignant cells with normal epithelial cells to form the calibrated training set (416,774 cells total in the paper) and verify calibrated malignant cells show higher malignancy-signature activity than the excluded 2.5% (criterion of Supplementary Fig. 2a).

## Related tools

- **scMalignantFinder** (Provides utils.aucell_cal to score the nine curated cancer gene signatures for training-set calibration and supplies the combined training data) — https://github.com/Jonyyqn/scMalignantFinder
- **pySCENIC** (AUCell implementation used under the hood to compute per-cell signature activity scores) — https://github.com/aertslab/pySCENIC

## Examples

```
adata = utils.aucell_cal(adata, sc_gmt)
```

## Evaluation signals

- calibrated_training_set.h5ad exists with exactly 416,774 rows and Raw_annotation labels present
- Calibrated malignant cell count equals 134,053 and normal epithelial count equals 282,721
- Approximately 2.5% of originally annotated malignant cells are excluded by the signature-consistency filter
- Calibrated malignant cells show higher malignancy-signature AUCell activity than the excluded 2.5% (Supplementary Fig. 2a criterion)
- Downstream scMalignantFinder assigns higher malignancy probability to calibrated malignant cells than to the excluded cells

## Limitations

- Relies on prerequisite identification of basic cell types; the approach is designed to identify malignant cells only from cancerous lineages.
- Curated cancer signatures exhibit consistent patterns across diverse cancer types, so cancers with unknown or controversial origins (e.g., synovial sarcoma and certain brain tumors) remain untested.
- The calibration baseline depends on accurately annotated 'Normal' epithelial cells in the source studies; mislabeled normal cells would shift the baseline.

## Evidence
