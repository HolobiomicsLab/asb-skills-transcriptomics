---
name: single-cell-malignancy-classification
description: Use when scRNA-seq .h5ad data from a known cancer lineage must be split
  into malignant vs normal epithelial cells without CNV inference, especially to track
  malignancy probability across progression stages.
when_to_use_negative:
- Cancers with unknown or controversial cell of origin (e.g., synovial sarcoma, certain
  brain tumors) — capabilities are untested there
- Datasets without prerequisite basic cell-type identification, since malignant identification
  is restricted to a known cancerous lineage
- Spatial transcriptomics region calling needing expert-curated ST references that
  are not yet available
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scMalignantFinder
  role: 'Primary classifier: loads pretrained model.joblib/ordered_feature.tsv, normalizes
    input (target_sum=1e4), predicts Normal/Malignant labels and malignancy_probability
    per cell'
  repo: github.com/Jonyyqn/scMalignantFinder
- name: scATOMIC
  role: Optional upstream pan-cancer cell type annotation integrated into the current
    version of scMalignantFinder
  repo: github.com/abelson-lab/scATOMIC
- name: CopyKAT
  role: CNV-based alternative benchmarked in the article; avoids the normal-reference
    requirement of CNV inference but the skill's classifier route does not need CNVs
  repo: github.com/navinlabcode/copykat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:7a1d59bf57eb89129593515f2d02cae6ee99f44b316dce5c20b2ce69ab9f7dd1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - CopyKAT
  - scATOMIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-malignancy-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply scMalignantFinder, a machine-learning classifier trained on signature-calibrated malignant cells, to label epithelial cells as Normal/Malignant and output a malignancy_probability, enabling quantification of malignancy trends across carcinogenesis stages (e.g., normal mucosa → polyp → tumor).

## When to use

Use when scRNA-seq .h5ad data from a known cancer lineage must be split into malignant vs normal epithelial cells without CNV inference, especially to track malignancy probability across progression stages.

## When NOT to use

- Cancers with unknown or controversial cell of origin (e.g., synovial sarcoma, certain brain tumors) — capabilities are untested there
- Datasets without prerequisite basic cell-type identification, since malignant identification is restricted to a known cancerous lineage
- Spatial transcriptomics region calling needing expert-curated ST references that are not yet available

## Inputs

- Per-stage single-cell expression matrices in .h5ad format (colorectal/gastric normal mucosa, CAG, IM, polyp, tumor/EGC stages)
- Pretrained model.joblib from Zenodo record 17888140
- Ordered feature list ordered_feature.tsv from Zenodo record 17888140
- Epithelial-lineage cell annotations used to subset before prediction

## Outputs

- result_adata with obs columns scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability
- Per-stage summary table (progression_summary.csv) with % malignant cells and mean malignancy probability per tissue and stage

## How to apply

First subset the expression matrix to the cancerous (epithelial) lineage, since scMalignantFinder is designed to distinguish malignant cells from their originating normal cells rather than from all non-malignant cells. Download the pretrained resources model.joblib and ordered_feature.tsv from Zenodo record 17888140 into a pretrained_model/ directory, load each stage's .h5ad as test_input, and run classifier.scMalignantFinder with pretrain_dir=pretrained_model, norm_type=True (sc.pp.normalize_total with target_sum=1e4) and n_thread=1; then call model.load() and model.predict() to obtain result_adata. Extract the scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability columns from result_adata.obs, group cells by tissue and progression stage, and compute the percentage of Malignant-labeled cells plus the mean malignancy probability per stage (e.g., write progression_summary.csv). Correct application shows the reported monotonic trend: ~0% malignant in normal mucosa/NAG, significantly increased in polyps/IM, peaking in tumors/EGC.

## Related tools

- **scMalignantFinder** (Primary classifier: loads pretrained model.joblib/ordered_feature.tsv, normalizes input (target_sum=1e4), predicts Normal/Malignant labels and malignancy_probability per cell) — github.com/Jonyyqn/scMalignantFinder
- **scATOMIC** (Optional upstream pan-cancer cell type annotation integrated into the current version of scMalignantFinder) — github.com/abelson-lab/scATOMIC
- **CopyKAT** (CNV-based alternative benchmarked in the article; avoids the normal-reference requirement of CNV inference but the skill's classifier route does not need CNVs) — github.com/navinlabcode/copykat

## Examples

```
result = classifier.scMalignantFinder(test_input=adata_epi, pretrain_dir='pretrained_model/', norm_type=True, n_thread=1); model.load(); result_adata = model.predict()
```

## Evaluation signals

- Normal mucosa/NAG samples yield nearly 0% cells labeled Malignant
- Malignant percentage increases monotonically across stages: normal < polyp/CAG < IM < tumor/EGC, with IM exceeding CAG in gastric progression
- Every cell in result_adata.obs carries both scMalignantFinder_prediction and malignancy_probability columns with probabilities in [0,1]
- Predictions restricted to the epithelial lineage subset, not the whole mixed-cell dataset
- Input counts were normalized via sc.pp.normalize_total(target_sum=1e4) (norm_type=True) before inference

## Limitations

- Requires prior identification of basic cell types; malignant cell identification is designed for cancerous lineages only
- Untested for cancers with unknown or controversial origins such as synovial sarcoma and certain brain tumors
- ST malignant-region identification depends on continuous accumulation of expert-curated ST data, which is not yet available
- Molecular mechanisms underlying intermediate states in gastric and colorectal carcinogenesis are not yet resolved

## Evidence
