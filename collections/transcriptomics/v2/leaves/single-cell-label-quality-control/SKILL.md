---
name: single-cell-label-quality-control
description: Use when a training/reference single-cell dataset carries 'Malignant'/'Tumor'
  labels assigned by heterogeneous original-study methods, and you must calibrate
  or validate those labels before training a classifier.
when_to_use_negative:
- The dataset's malignant labels already come from a signature-calibrated or experimentally
  validated source, so re-filtering would double-correct
- The sample is a non-cancer tissue or cancer of unknown/controversial origin (e.g.,
  synovial sarcoma, certain brain tumors), where the curated cancer signatures may
  not apply
- You only have spatial transcriptomics spots rather than single cells — use the malignant-region
  identification workflow instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Provides utils.aucell_cal to score cells against the sc_gmt cancer signatures
    used for label calibration
  repo: https://github.com/Jonyyqn/scMalignantFinder
- name: pySCENIC
  role: Underlying AUCell implementation used to compute per-cell signature activity
    scores
  repo: https://github.com/aertslab/pySCENIC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:d32842136ce38ccb7ff8c02de3afac1374b17fc51914536ce14ca6568e64cc7b
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# single-cell-label-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control routine that rescues noisy cell labels by re-scoring originally annotated malignant cells against curated cancer gene signatures and removing label-inconsistent cells. In scMalignantFinder, nine curated pan-cancer signatures were scored with AUCell and ~2.5% of initially annotated malignant training cells were filtered out.

## When to use

Use when a training/reference single-cell dataset carries 'Malignant'/'Tumor' labels assigned by heterogeneous original-study methods, and you must calibrate or validate those labels before training a classifier.

## When NOT to use

- The dataset's malignant labels already come from a signature-calibrated or experimentally validated source, so re-filtering would double-correct
- The sample is a non-cancer tissue or cancer of unknown/controversial origin (e.g., synovial sarcoma, certain brain tumors), where the curated cancer signatures may not apply
- You only have spatial transcriptomics spots rather than single cells — use the malignant-region identification workflow instead

## Inputs

- Annotated single-cell RNA-seq training dataset in .h5ad format (e.g., combine_training.h5ad) with original labels in adata.obs['Raw_annotation']
- Gene signature file in GMT format (sc_malignant_deg.gmt) containing nine curated pan-cancer up- and downregulated malignant signatures
- A set of cells reliably annotated as normal epithelial ('Normal') to serve as the signature-activity baseline

## Outputs

- A calibrated training set (calibrated_training_set.h5ad) totaling 416,774 cells: 134,053 malignant and 282,721 normal epithelial cells
- Per-cell AUCell scores for the nine cancer signatures
- The excluded set of label-inconsistent malignant cells (~2.5% of the originally annotated malignant cells)

## How to apply

Load the annotated dataset (e.g., combine_training.h5ad from Zenodo records/17888140) and read original labels from adata.obs['Raw_annotation'], treating both 'Malignant' and 'Tumor' as malignant. Score each cell for the nine curated pan-cancer gene signatures (upregulated and downregulated malignant sets, provided as sc_malignant_deg.gmt) using scMalignantFinder's utils.aucell_cal, which wraps AUCell from pySCENIC. Establish a per-signature baseline by averaging the up- and downregulated-signature AUCell scores among 'Normal'-annotated epithelial cells. Retain malignant cells whose upregulated-signature activity is higher and/or downregulated-signature activity is lower than that normal-cell average; the ~2.5% of training cells failing this consistency criterion are excluded. The rationale is that labels derived from various methodologies in the original studies may include cells that lack the consistent transcriptional cancer pattern captured by the curated signatures, so removing them yields a cleaner classifier training set.

## Related tools

- **scMalignantFinder** (Provides utils.aucell_cal to score cells against the sc_gmt cancer signatures used for label calibration) — https://github.com/Jonyyqn/scMalignantFinder
- **pySCENIC** (Underlying AUCell implementation used to compute per-cell signature activity scores) — https://github.com/aertslab/pySCENIC

## Examples

```
from scMalignantFinder import utils; adata = utils.aucell_cal(adata, sc_gmt)  # score nine pan-cancer signatures, then filter vs Normal-epithelial baseline
```

## Evaluation signals

- The calibrated training set contains exactly 416,774 cells (134,053 malignant, 282,721 normal epithelial)
- calibrated_training_set.h5ad loads with 416,774 rows and Raw_annotation labels present
- Roughly 2.5% of the originally annotated malignant cells are excluded by the signature-consistency filter
- Calibrated malignant cells show higher malignancy-signature activity than the excluded 2.5% (the Supplementary Fig. 2a criterion)
- Scikit check: retained malignant cells have upregulated-signature AUCell activity above, and/or downregulated-signature activity below, the normal-epithelial average

## Limitations

- The filter assumes 'Normal'-annotated epithelial cells provide a valid baseline; datasets lacking such cells cannot establish the per-signature normal-cell average
- Calibration only refines malignant-vs-normal labels within cancerous lineages; it relies on prerequisite basic cell-type identification
- Applicability to cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors) is untested

## Evidence
