---
name: pretrained-classifier-inference-on-st-spots
description: Use when the input is a tumor ST dataset (h5ad spots with spatial coordinates
  and H&E image) and per-spot malignant-vs-normal malignancy probabilities are needed
  as features for malignant-region calling.
when_to_use_negative:
- Input is scRNA-seq or ST from normal/non-cancerous tissue — the classifier is designed
  to distinguish malignant cells from their originating normal cells within cancerous
  lineages
- Cancer type has unknown or controversial origin (e.g., synovial sarcoma, certain
  brain tumors) — capabilities are untested there
- You need copy-number-based annotation without a pretrained model (e.g., aneuploidy
  inference with CopyKAT) — this skill is signature/classifier-based inference, not
  CNV inference
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3391
tools:
- name: scMalignantFinder
  role: Provides the pretrained classifier (model.joblib, ordered_feature.tsv) loaded
    via model.load() and applied via model.predict() to yield malignancy_probability
    per ST spot
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Cancer-Finder
  role: Benchmark comparator for region-level malignant-region identification accuracy
    on the same eight tumor ST datasets
  repo: Patchouli-M/SequencingCancerFinder
- name: scATOMIC
  role: Optional upstream pan-cancer cell type annotation integrated into the current
    scMalignantFinder version to identify cancerous lineages before malignancy scoring
  repo: github.com/abelson-lab/scATOMIC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:f03f539f13c57b8880d600f5d130faa9766e3805fa9eed86a5797e76179bd6fa
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
  - scATOMIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pretrained-classifier-inference-on-st-spots

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the pretrained scMalignantFinder classifier (model.joblib with ordered_feature.tsv) on spatial transcriptomics spots to obtain per-spot malignancy_probability without retraining, as the malignant-feature input for downstream ST region identification.

## When to use

Use when the input is a tumor ST dataset (h5ad spots with spatial coordinates and H&E image) and per-spot malignant-vs-normal malignancy probabilities are needed as features for malignant-region calling.

## When NOT to use

- Input is scRNA-seq or ST from normal/non-cancerous tissue — the classifier is designed to distinguish malignant cells from their originating normal cells within cancerous lineages
- Cancer type has unknown or controversial origin (e.g., synovial sarcoma, certain brain tumors) — capabilities are untested there
- You need copy-number-based annotation without a pretrained model (e.g., aneuploidy inference with CopyKAT) — this skill is signature/classifier-based inference, not CNV inference

## Inputs

- Tumor spatial transcriptomics count matrix (h5ad) with spot-level spatial coordinates
- Pretrained model.joblib (Zenodo record 17888140)
- ordered_feature.tsv feature ordering list
- sc_malignant_deg.gmt malignant gene signature file

## Outputs

- adata.obs['malignancy_probability'] per spot (predicted probability of being malignant)
- Result adata (result_adata) from model.predict() for downstream region identification

## How to apply

Load the pretrained model from the Zenodo deposit (model.joblib, record 17888140) with model.load(), using ordered_feature.tsv to ensure the feature vector matches training order. Apply norm_type=True so sc.pp.normalize_total(adata, target_sum=1e4) is applied to the input counts; the workflow specifies no internal log-transformation. Call model.predict() to produce the malignancy_probability obs feature per spot. Because the classifier was trained on scRNA-seq and extended to ST without retraining, its spot-level probability should be combined with the Malignant_up AUCell signature score (sc_malignant_deg.gmt) and the image_score before spatial.region_identification rather than thresholded directly.

## Related tools

- **scMalignantFinder** (Provides the pretrained classifier (model.joblib, ordered_feature.tsv) loaded via model.load() and applied via model.predict() to yield malignancy_probability per ST spot) — github.com/Jonyyqn/scMalignantFinder
- **Cancer-Finder** (Benchmark comparator for region-level malignant-region identification accuracy on the same eight tumor ST datasets) — Patchouli-M/SequencingCancerFinder
- **scATOMIC** (Optional upstream pan-cancer cell type annotation integrated into the current scMalignantFinder version to identify cancerous lineages before malignancy scoring) — github.com/abelson-lab/scATOMIC

## Examples

```
model.load(); result_adata = model.predict()
```

## Evaluation signals

- Across eight tumor ST datasets spanning five cancer types, region-level accuracy vs pathologist annotations is approximately 0.783 and balanced accuracy approximately 0.800
- scMalignantFinder exceeds Cancer-Finder by ~0.067 accuracy and ~0.062 balanced accuracy on the same eight datasets
- Every spot in the result_adata carries a numeric malignancy_probability in [0,1]; spots missing this obs column indicate a feature-ordering or normalization mismatch
- Ordered features must match ordered_feature.tsv and counts must be normalized with norm_type=True (normalize_total to target_sum=1e4); skipping this changes predicted probabilities
- The classifier runs without retraining on ST data — any retraining or fine-tuning step signals deviation from the validated workflow

## Limitations

- The tool requires prerequisite identification of basic cell types and is restricted to identifying malignant cells from cancerous lineages
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- ST malignant-region identification still depends on the future accumulation of expert-curated ST data; additional spatial features (e.g., spatial distribution probability of malignant cells) are not yet incorporated

## Evidence
