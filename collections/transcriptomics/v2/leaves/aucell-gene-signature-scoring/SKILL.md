---
name: aucell-gene-signature-scoring
description: Use when you need per-cell gene set activity scores (e.g., malignant
  up/down signature activity) to calibrate cell labels, define malignant clusters,
  or establish normal-cell baselines in scRNA-seq or spatial transcriptomics data.
when_to_use_negative:
- When you need malignancy probabilities from a trained classifier rather than signature
  activity scores — use the pretrained model.joblib via model.load()/model.predict()
  instead
- When gene sets have inconsistent transcriptional patterns across cancer types —
  scMalignantFinder's calibration rationale depends on the nine curated signatures
  exhibiting 'consistent transcriptional patterns across diverse cancer types'
- When working with non-epithelial lineages — the malignant/non-malignant distinction
  here is 'designed to identify malignant cells from cancerous lineages, relying on
  the prerequisite identification of basic cell types'
edam_operation: http://edamontology.org/operation_3745
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Provides utils.aucell_cal, which wraps pySCENIC AUCell to score signature
    activity on AnnData; used for training-set calibration and Malignant_up spot scoring
  repo: https://github.com/Jonyyqn/scMalignantFinder
- name: pySCENIC
  role: Underlying AUCell implementation used to compute per-cell gene signature activity
    rankings
  repo: https://github.com/aertslab/pySCENIC
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:e8f47d891c7c9b6277da265f40a310c9b8e81315eeee7fcdb0f511c1d4324cd5
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
  edam_operation: http://edamontology.org/operation_3745
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# aucell-gene-signature-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score per-cell/per-spot activity of curated cancer gene signatures using AUCell (from pySCENIC), as scMalignantFinder does with the nine curated pan-cancer signatures in sc_malignant_deg.gmt and the Malignant_up score used for malignant-region identification.

## When to use

Use when you need per-cell gene set activity scores (e.g., malignant up/down signature activity) to calibrate cell labels, define malignant clusters, or establish normal-cell baselines in scRNA-seq or spatial transcriptomics data.

## When NOT to use

- When you need malignancy probabilities from a trained classifier rather than signature activity scores — use the pretrained model.joblib via model.load()/model.predict() instead
- When gene sets have inconsistent transcriptional patterns across cancer types — scMalignantFinder's calibration rationale depends on the nine curated signatures exhibiting 'consistent transcriptional patterns across diverse cancer types'
- When working with non-epithelial lineages — the malignant/non-malignant distinction here is 'designed to identify malignant cells from cancerous lineages, relying on the prerequisite identification of basic cell types'

## Inputs

- AnnData object (.h5ad) with raw or normalized single-cell/spatial counts and cell-type annotations in obs (e.g., Raw_annotation with 'Malignant'/'Tumor'/'Normal' labels)
- Gene signature collection in .gmt format (e.g., sc_malignant_deg.gmt from Zenodo record 17888140)
- Nine curated pan-cancer gene sets (upregulated and downregulated malignant signatures)

## Outputs

- Per-cell/per-spot AUCell signature activity scores added to adata.obs (e.g., Malignant_up)
- Calibrated cell-label set (malignant cells failing signature-consistency criterion excluded, yielding calibrated_training_set.h5ad with 416,774 cells)
- Normal-cell baseline signature activity values used as filtering thresholds

## How to apply

Load a gene-set collection as a .gmt file (e.g., scMalignantFinder's sc_malignant_deg.gmt from Zenodo record 17888140, containing upregulated and downregulated malignant signatures) and run AUCell via scMalignantFinder's utils.aucell_cal, which wraps pySCENIC's AUCell ranking-based scoring, to append signature activity scores to adata.obs. In training-set calibration, average the up- and downregulated-signature AUCell scores among 'Normal'-annotated epithelial cells to establish a normal-cell baseline per signature, then retain malignant cells whose upregulated activity is higher and/or downregulated activity is lower than that baseline (excluding ~2.5% of originally annotated malignant cells). In spatial workflows, the Malignant_up AUCell score is computed per spot and used as one of three obs features — alongside malignancy_probability and image_score — fed into spatial.region_identification with hierarchical clustering (nclus=3), where the malignant cluster is defined by define_feature='Malignant_up'. Because AUCell is ranking-based, it is robust to differences in total counts across cells, complementing the count-normalized classifier input (norm_type=True applies sc.pp.normalize_total to target_sum=1e4).

## Related tools

- **scMalignantFinder** (Provides utils.aucell_cal, which wraps pySCENIC AUCell to score signature activity on AnnData; used for training-set calibration and Malignant_up spot scoring) — https://github.com/Jonyyqn/scMalignantFinder
- **pySCENIC** (Underlying AUCell implementation used to compute per-cell gene signature activity rankings) — https://github.com/aertslab/pySCENIC

## Examples

```
import scMalignantFinder.utils as utils; adata = utils.aucell_cal(adata, 'sc_malignant_deg.gmt')
```

## Evaluation signals

- Calibrated training set totals 416,774 cells (134,053 malignant, 282,721 normal epithelial) after excluding ~2.5% of originally annotated malignant cells
- Calibrated malignant cells show higher malignancy-signature activity than the excluded 2.5% (criterion of Supplementary Fig. 2a)
- Per-spot Malignant_up AUCell scores are present in adata.obs and computable for all spots before spatial.region_identification is called
- Signature scores use the exact .gmt file from the Zenodo deposit (sc_malignant_deg.gmt), matching the nine curated pan-cancer signatures

## Limitations

- The training calibration assumes the nine curated signatures exhibit consistent transcriptional patterns across cancer types; performance may degrade for cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors)
- AUCell scores here serve as one of three features for ST region identification; expert-curated ST reference data for validating malignant regions is not yet available at scale
- Requires prior basic cell-type identification to restrict scoring to cancerous lineages

## Evidence

- [methods] aucell_cal wrapper: "adata = utils.aucell_cal(adata, sc_gmt)"
- [abstract] signature calibration rationale: "the initially annotated malignant cells were calibrated using nine carefully curated pan-cancer gene signatures"
- [discussion] exclusion criterion: "scMalignantFinder assigned higher malignancy probability to the calibrated malignant cells in the training set compared to the 2.5% of cells that were excluded (Supplementary Fig. 2a)."
- [discussion] ST multi-feature use: "Through a multi-feature clustering framework, incorporating malignant features, image-based characteristics, and spatial neighborhood networks"
- [readme] AUCell implementation source: [evidence span withheld — anchor status: external_doc]
