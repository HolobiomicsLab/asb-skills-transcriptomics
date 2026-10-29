---
name: progression-stage-stratified-analysis
description: Use when stage-resolved epithelial scRNA-seq matrices (e.g., .h5ad per
  normal mucosa, polyp, tumor, CAG, IM, EGC stage) are available and the goal is to
  test whether predicted malignancy increases monotonically across carcinogenesis
  stages.
when_to_use_negative:
- Input is normal (non-cancerous) tissue or a lineage that has not been identified
  as the cancerous epithelial lineage — the tool requires prerequisite identification
  of basic cell types and is restricted to cancerous lineages
- Cancers with unknown or controversial cell of origin, such as synovial sarcoma or
  certain brain tumors, where capabilities are untested
- The goal is de novo malignant-cell discovery without pretrained features — alternative
  tools like ikarus, Cancer-Finder, or CopyKAT would be required instead
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3391
tools:
- name: scMalignantFinder
  role: Pretrained malignancy classifier run per carcinogenesis stage on epithelial-subset
    .h5ad inputs to produce Malignant/Normal labels and malignancy probabilities
  repo: github.com/Jonyyqn/scMalignantFinder
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:0b18b9470d75aff692b66cd5a7691c5f67762af580d8e72a014fa6ad12d7543c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# progression-stage-stratified-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the pretrained scMalignantFinder classifier stage-by-stage across colorectal and gastric carcinogenesis scRNA-seq datasets (normal mucosa → polyps/CAG → IM → tumors/EGC) to quantify how the fraction of epithelial cells predicted 'Malignant' and their mean malignancy probability change with progression stage.

## When to use

Use when stage-resolved epithelial scRNA-seq matrices (e.g., .h5ad per normal mucosa, polyp, tumor, CAG, IM, EGC stage) are available and the goal is to test whether predicted malignancy increases monotonically across carcinogenesis stages.

## When NOT to use

- Input is normal (non-cancerous) tissue or a lineage that has not been identified as the cancerous epithelial lineage — the tool requires prerequisite identification of basic cell types and is restricted to cancerous lineages
- Cancers with unknown or controversial cell of origin, such as synovial sarcoma or certain brain tumors, where capabilities are untested
- The goal is de novo malignant-cell discovery without pretrained features — alternative tools like ikarus, Cancer-Finder, or CopyKAT would be required instead

## Inputs

- Stage-resolved single-cell expression matrices (.h5ad) for colorectal (normal mucosa, polyps, tumors) and gastric (CAG, IM, tumors/EGC) progression datasets
- Pretrained classifier model.joblib from Zenodo record 17888140
- Ordered feature list ordered_feature.tsv from Zenodo record 17888140

## Outputs

- result_adata with per-cell scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability columns in .obs
- progression_summary.csv with per-stage percentage of Malignant-labeled cells and mean malignancy probability

## How to apply

Download the pretrained resources model.joblib and ordered_feature.tsv from Zenodo record 17888140 into a pretrained_model/ directory. For each progression stage, load the .h5ad expression matrix as the test_input and subset to the epithelial lineage before prediction, since scMalignantFinder is designed to identify malignant cells only from cancerous lineages after basic cell-type annotation. Run classifier.scMalignantFinder with pretrain_dir='pretrained_model/', norm_type=True (sc.pp.normalize_total with target_sum=1e4) and n_thread=1, then call model.load() and model.predict() to get result_adata. Extract the scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability columns from result_adata.obs, group by tissue and stage, and compute the percentage of Malignant-labeled cells plus mean malignancy probability per stage. Verify the expected ordering — nearly 0% malignant in normal mucosa/NAG, IM exceeding CAG, peaks in tumors/EGC — and write the table to progression_summary.csv.

## Related tools

- **scMalignantFinder** (Pretrained malignancy classifier run per carcinogenesis stage on epithelial-subset .h5ad inputs to produce Malignant/Normal labels and malignancy probabilities) — github.com/Jonyyqn/scMalignantFinder

## Examples

```
from scMalignantFinder import classifier; result_adata = classifier.scMalignantFinder(adata_epi, pretrain_dir='pretrained_model/', norm_type=True, n_thread=1); classifier.model.load(); result_adata = classifier.model.predict()
```

## Evaluation signals

- Percentage of epithelial cells predicted Malignant is nearly 0% in normal mucosa (and NAG) samples
- Gastric intermediate metaplasia (IM) stages show a higher malignant fraction and/or mean malignancy probability than chronic atrophic gastritis (CAG) stages
- Malignant percentage and mean malignancy_probability peak in tumor (colorectal) and early gastric cancer (EGC) stages, completing the monotonic trend
- progression_summary.csv contains a row per tissue and progression stage with the two computed metrics
- Predictions are only made on the epithelial-lineage subset, never on the full mixed-cell matrix

## Limitations

- scMalignantFinder requires prerequisite identification of basic cell types and only identifies malignant cells from cancerous lineages
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- Relying on the pretrained model assumes the input expression matrices are raw counts amenable to target_sum=1e4 total-count normalization

## Evidence
