---
name: cancer-signature-baseline-thresholding
description: Use when building a supervised malignant-vs-normal training set from
  scRNA-seq data whose 'Malignant'/'Tumor' labels were assigned by upstream studies
  of variable quality, and you need to remove mislabeled cells before classifier training.
when_to_use_negative:
- The dataset contains no reliably annotated normal epithelial cells, so the normal-cell
  AUCell baseline cannot be computed
- You are annotating malignant cells de novo rather than calibrating pre-existing
  'Malignant'/'Tumor' labels for training-set construction (use scMalignantFinder's
  predict instead)
- The input is ST data or a cell-type annotation task where the basic cell types have
  not yet been identified (the method requires prerequisite identification of basic
  cell types)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Provides utils.aucell_cal used to score the nine curated cancer gene signatures
    and to calibrate the training malignant cells
  repo: github.com/Jonyyqn/scMalignantFinder
- name: pySCENIC
  role: Supplies the AUCell implementation underlying per-cell signature activity
    scoring during calibration
  repo: github.com/aertslab/pySCENIC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:8e047f9e74298832d2d4931c6cdb55199b4be69629110f1787a62d762f48db2c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - pySCENIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cancer-signature-baseline-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates an initially annotated malignant-cell training set by scoring curated pan-cancer gene signatures (AUCell) and filtering out cells whose signature activity is inconsistent with malignancy, using normal epithelial cells as the baseline. This removes label noise before supervised classifier training.

## When to use

Use when building a supervised malignant-vs-normal training set from scRNA-seq data whose 'Malignant'/'Tumor' labels were assigned by upstream studies of variable quality, and you need to remove mislabeled cells before classifier training.

## When NOT to use

- The dataset contains no reliably annotated normal epithelial cells, so the normal-cell AUCell baseline cannot be computed
- You are annotating malignant cells de novo rather than calibrating pre-existing 'Malignant'/'Tumor' labels for training-set construction (use scMalignantFinder's predict instead)
- The input is ST data or a cell-type annotation task where the basic cell types have not yet been identified (the method requires prerequisite identification of basic cell types)

## Inputs

- combine_training.h5ad — merged anndata object with per-cell labels in adata.obs['Raw_annotation'] ('Malignant'/'Tumor'/'Normal'), from Zenodo records/17888140
- sc_malignant_deg.gmt — GMT file of nine curated pan-cancer gene signatures (upregulated and downregulated malignant signature gene sets)

## Outputs

- calibrated_training_set.h5ad — anndata with calibrated labels totaling 416,774 cells (134,053 malignant, 282,721 normal epithelial)
- Set of excluded malignant cells (~2.5% of originally annotated malignant cells) that failed the signature-consistency criterion

## How to apply

Load combine_training.h5ad (Zenodo records/17888140) and read original labels from adata.obs['Raw_annotation'], treating both 'Malignant' and 'Tumor' as malignant. Compute per-cell AUCell scores (via scMalignantFinder's utils.aucell_cal, which wraps AUCell from pySCENIC) for the nine curated pan-cancer gene signatures — including both upregulated and downregulated malignant signature sets — across all five training datasets. For each signature, average AUCell scores among 'Normal'-annotated epithelial cells to establish a normal-cell baseline. Retain malignant cells whose upregulated-signature activity is higher and/or downregulated-signature activity is lower than the normal-cell average, which excludes ~2.5% of originally annotated malignant cells that fail the signature-consistency criterion. The rationale is that originally annotated malignant cells come from heterogeneous methodologies in the source studies, and curated signatures with consistent transcriptional patterns across cancer types provide an independent quality control on those labels.

## Related tools

- **scMalignantFinder** (Provides utils.aucell_cal used to score the nine curated cancer gene signatures and to calibrate the training malignant cells) — github.com/Jonyyqn/scMalignantFinder
- **pySCENIC** (Supplies the AUCell implementation underlying per-cell signature activity scoring during calibration) — github.com/aertslab/pySCENIC

## Examples

```
adata = utils.aucell_cal(adata, sc_gmt)
```

## Evaluation signals

- Calibrated set totals 416,774 cells with 134,053 malignant and 282,721 normal epithelial cells
- Excluded cells represent ~2.5% of the originally annotated malignant training cells
- Calibrated malignant cells show higher malignancy probability than the excluded 2.5% (criterion of Supplementary Fig. 2a)
- calibrated_training_set.h5ad exists with 416,774 rows and Raw_annotation labels present
- Per-cell AUCell scores were computed for both upregulated and downregulated malignant signature sets across all five training datasets

## Limitations

- Requires prerequisite identification of basic cell types and is restricted to identifying malignant cells from cancerous lineages
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- The filter assumes curated signatures exhibit consistent transcriptional patterns across cancer types; cancers diverging from these patterns may be mislabeled

## Evidence

- [readme] AUCell implementation source: [evidence span withheld — anchor status: external_doc]
