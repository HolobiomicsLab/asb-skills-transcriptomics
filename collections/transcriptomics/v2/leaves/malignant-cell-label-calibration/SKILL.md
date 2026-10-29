---
name: malignant-cell-label-calibration
description: Use when building or cleaning a malignant-vs-normal training set for
  cancer scRNA-seq cell annotation — i.e., when 'Malignant'/'Tumor' labels from original
  studies may be noisy and curated cancer gene signatures are available to validate
  them.
when_to_use_negative:
- Input is a dataset without pre-identified basic cell types — scMalignantFinder is
  restricted to cancerous lineages and relies on prerequisite identification of basic
  cell types, not whole-transcriptome annotation from scratch
- Cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain
  tumors) — the calibrated signatures have not been validated there
- The dataset has no confidently annotated normal epithelial cells, since the filtering
  baseline is defined by AUCell activity in normal epithelial cells
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Provides utils.aucell_cal for scoring the nine curated cancer gene signatures
    per cell and hosts the calibrated training data workflow
  repo: github.com/Jonyyqn/scMalignantFinder
- name: pySCENIC
  role: Supplies the AUCell implementation underlying the per-cell signature activity
    scoring
  repo: aertslab/pySCENIC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:01c53618eb806c962b27299eee14d6da39e826eedc77dd92b2dcd1276a347e89
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# malignant-cell-label-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates initially annotated 'Malignant'/'Tumor' cells in a single-cell training set by scoring nine curated pan-cancer gene signatures (up- and downregulated malignant signatures) with AUCell and filtering out cells (~2.5%) whose signature activity is inconsistent with malignancy.

## When to use

Use when building or cleaning a malignant-vs-normal training set for cancer scRNA-seq cell annotation — i.e., when 'Malignant'/'Tumor' labels from original studies may be noisy and curated cancer gene signatures are available to validate them.

## When NOT to use

- Input is a dataset without pre-identified basic cell types — scMalignantFinder is restricted to cancerous lineages and relies on prerequisite identification of basic cell types, not whole-transcriptome annotation from scratch
- Cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors) — the calibrated signatures have not been validated there
- The dataset has no confidently annotated normal epithelial cells, since the filtering baseline is defined by AUCell activity in normal epithelial cells

## Inputs

- combine_training.h5ad (an AnnData single-cell object with original labels in adata.obs['Raw_annotation']; 'Malignant' and 'Tumor' both treated as malignant)
- sc_malignant_deg.gmt — nine curated pan-cancer gene signature sets (upregulated and downregulated malignant signatures)
- Normal-annotated epithelial cells within the same training datasets (to establish the per-signature AUCell baseline)

## Outputs

- calibrated_training_set.h5ad — 416,774 cells (134,053 malignant, 282,721 normal epithelial) with refined malignant labels
- Per-cell AUCell scores for the nine up- and downregulated cancer signatures
- Set of excluded cells (~2.5% of originally annotated malignant cells) failing the signature-consistency criterion

## How to apply

Load combine_training.h5ad from Zenodo (records/17888140) and read original labels from adata.obs['Raw_annotation'], treating both 'Malignant' and 'Tumor' as malignant. Compute per-cell AUCell scores (via pySCENIC) for the nine curated pan-cancer gene sets — split into upregulated and downregulated malignant signatures — across all five training datasets using scMalignantFinder's utils.aucell_cal with the sc_malignant_deg.gmt signature file. Establish a per-signature baseline by averaging the AUCell scores among cells annotated as 'Normal' epithelial. Retain malignant cells whose upregulated-signature activity is higher and/or downregulated-signature activity is lower than that normal-cell average, excluding the ~2.5% of training cells that fail this signature-consistency criterion. Combine the retained malignant cells with normal epithelial cells to yield the calibrated training set (416,774 cells: 134,053 malignant, 282,721 normal epithelial). The rationale is that labels from heterogeneous original annotation methods are unreliable, and signature activity provides an orthogonal, knowledge-driven check before supervised model training.

## Related tools

- **scMalignantFinder** (Provides utils.aucell_cal for scoring the nine curated cancer gene signatures per cell and hosts the calibrated training data workflow) — github.com/Jonyyqn/scMalignantFinder
- **pySCENIC** (Supplies the AUCell implementation underlying the per-cell signature activity scoring) — aertslab/pySCENIC

## Examples

```
adata = utils.aucell_cal(adata, sc_gmt)  # per-cell AUCell scores for the nine curated pan-cancer signatures
```

## Evaluation signals

- The calibrated training set contains exactly 416,774 cells (134,053 malignant, 282,721 normal epithelial) with Raw_annotation labels present
- The excluded fraction is ~2.5% of the originally annotated malignant cells
- Calibrated malignant cells show higher malignancy-signature activity than the excluded 2.5% (criterion of Supplementary Fig. 2a; scMalignantFinder assigns them higher malignancy probability)
- AUCell scores are computed separately for the upregulated and downregulated signature groups, and retention uses the normal-epithelial-average baseline per signature
- calibrated_training_set.h5ad exists with the expected number of rows and both up- and downregulated-signature filtering applied

## Limitations

- Calibration depends on labels and cell-type annotations from the original studies, which used various methodologies; the signatures only prune the ~2.5% most inconsistent cells
- The approach requires normal epithelial reference cells to define the baseline, so datasets lacking confident normal-epithelial annotation cannot be calibrated this way
- Curated cancer gene signatures may not transfer to cancers with unknown or controversial origins such as synovial sarcoma and certain brain tumors

## Evidence
