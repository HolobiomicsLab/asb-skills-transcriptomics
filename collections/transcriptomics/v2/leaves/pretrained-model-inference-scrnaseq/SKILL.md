---
name: pretrained-model-inference-scrnaseq
description: Use when raw scRNA-seq count matrices (.h5ad) from progression-stage
  samples (normal mucosa, polyps, tumors) must be scored for malignancy with a pretrained
  classifier instead of retraining, and per-stage malignant-cell percentages are needed.
when_to_use_negative:
- When no basic cell-type annotation is available — the tool 'is designed to identify
  malignant cells from cancerous lineages, relying on the prerequisite identification
  of basic cell types'
- For cancers with unknown or controversial origins (e.g., synovial sarcoma, certain
  brain tumors) — capabilities are untested there
- When the goal is de novo malignant-cell annotation without a pretrained model —
  this skill covers inference with the provided Zenodo model, not training a new classifier
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scMalignantFinder
  role: Pretrained machine-learning classifier whose model.joblib and ordered_feature.tsv
    are loaded to predict Normal/Malignant labels and malignancy_probability from
    epithelial scRNA-seq matrices
  repo: github.com/Jonyyqn/scMalignantFinder
- name: scATOMIC
  role: Optional upstream pan-cancer cell type annotator integrated into the current
    scMalignantFinder version for basic cell-type identification before malignancy
    prediction
  repo: github.com/abelson-lab/scATOMIC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:7bef72c1b293c260180e394604ad394fa5cc3f0146d5a9e60fc20b1cade3150c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - scATOMIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pretrained-model-inference-scrnaseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply scMalignantFinder's pretrained classifier (model.joblib + ordered_feature.tsv from Zenodo record 17888140) to epithelial scRNA-seq expression matrices (.h5ad) to obtain per-cell Normal/Malignant predictions and malignancy probabilities, enabling quantification of malignancy trends across carcinogenesis stages.

## When to use

Use when raw scRNA-seq count matrices (.h5ad) from progression-stage samples (normal mucosa, polyps, tumors) must be scored for malignancy with a pretrained classifier instead of retraining, and per-stage malignant-cell percentages are needed.

## When NOT to use

- When no basic cell-type annotation is available — the tool 'is designed to identify malignant cells from cancerous lineages, relying on the prerequisite identification of basic cell types'
- For cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors) — capabilities are untested there
- When the goal is de novo malignant-cell annotation without a pretrained model — this skill covers inference with the provided Zenodo model, not training a new classifier

## Inputs

- Stage-specific scRNA-seq expression matrices in .h5ad format (colorectal/gastric progression datasets: normal mucosa, CAG, IM, polyps, tumors, EGC)
- Pretrained classifier artifacts: model.joblib and ordered_feature.tsv from Zenodo record 17888140
- Epithelial-lineage cell annotations used to subset each .h5ad before prediction

## Outputs

- result_adata with scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability columns in .obs
- Per-stage summary table progression_summary.csv with percentage of Malignant-labeled cells and mean malignancy probability per tissue and stage

## How to apply

Download model.joblib and ordered_feature.tsv from Zenodo record 17888140 into a pretrained_model/ directory, then load each stage's expression matrix (.h5ad) as test_input and subset to the epithelial lineage, since the classifier is designed to distinguish malignant cells from their originating normal cells within a cancerous lineage. Run classifier.scMalignantFinder with pretrain_dir pointing to pretrained_model/, norm_type=True (applies sc.pp.normalize_total with target_sum=1e4) and n_thread=1, then call model.load() and model.predict() to get result_adata. Extract the scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability columns from result_adata.obs. Group cells by tissue and progression stage, compute the percentage of Malignant-labeled cells and mean malignancy probability per stage, verify the monotonic stage ordering, and write the summary to progression_summary.csv.

## Related tools

- **scMalignantFinder** (Pretrained machine-learning classifier whose model.joblib and ordered_feature.tsv are loaded to predict Normal/Malignant labels and malignancy_probability from epithelial scRNA-seq matrices) — github.com/Jonyyqn/scMalignantFinder
- **scATOMIC** (Optional upstream pan-cancer cell type annotator integrated into the current scMalignantFinder version for basic cell-type identification before malignancy prediction) — github.com/abelson-lab/scATOMIC

## Examples

```
result = classifier.scMalignantFinder(test_input=epi_adata, pretrain_dir='pretrained_model/', norm_type=True, n_thread=1); result_adata = model.predict()
```

## Evaluation signals

- Normal mucosa / NAG samples yield approximately 0% of epithelial cells predicted as Malignant
- Malignant percentage increases monotonically across stages: near 0% in normal mucosa, significantly higher in polyps, peaking in tumors/EGC
- Gastric progression shows IM exceeding CAG in predicted malignancy before peaking in EGC
- malignancy_probability values are present for every cell in result_adata.obs and are higher in tumor stages than in normal mucosa
- progression_summary.csv contains one row per tissue-stage combination with both percent-malignant and mean-probability columns populated

## Limitations

- The classifier requires prerequisite identification of basic cell types and is restricted to cancerous lineages; predictions outside the epithelial lineage of origin are unsupported
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- The malignancy probability captures dynamic progression characteristics but the molecular mechanisms underlying intermediate states (gastric/colorectal) are not yet resolved

## Evidence
