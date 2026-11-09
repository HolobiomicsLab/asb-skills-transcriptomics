---
name: pretrained-model-inference-on-ann-data
description: Use when you have raw-count AnnData (.h5ad) scRNA-seq data with cells
  from a cancerous lineage already identified and need automated malignant-vs-normal
  cell calls plus a malignancy probability from a pretrained model.
when_to_use_negative:
- Cells are not from a cancerous lineage — the tool is designed to identify malignant
  cells from cancerous lineages and relies on prior identification of basic cell types
- Cancer type has unknown or controversial cell of origin (e.g., synovial sarcoma,
  certain brain tumors) — performance is untested
- Only log1p-normalized data is available and raw counts cannot be recovered, since
  the model expects normalize_total(target_sum=1e4) output without log transformation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0204
tools:
- name: scMalignantFinder
  role: Pretrained classifier installed via pip; loads model.joblib + ordered_feature.tsv
    and predicts malignant vs normal cells with malignancy probability on normalized
    AnnData
  repo: github.com/Jonyyqn/scMalignantFinder
- name: scATOMIC
  role: Optional upstream pan-cancer cell type annotator integrated into the current
    version for automated annotation before malignancy scoring
  repo: github.com/abelson-lab/scATOMIC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:00092769b19ce1e61e41709f610733f3eb6222364382676c147eedfa54428e94
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
  - PreCanCell
  - ikarus
  - scATOMIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pretrained-model-inference-on-ann-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs a pretrained scikit-learn classifier (scMalignantFinder's model.joblib with its ordered_feature.tsv feature list) directly on AnnData/.h5ad scRNA-seq objects to output per-cell 'Normal'/'Malignant' labels and malignancy probabilities without retraining.

## When to use

Use when you have raw-count AnnData (.h5ad) scRNA-seq data with cells from a cancerous lineage already identified and need automated malignant-vs-normal cell calls plus a malignancy probability from a pretrained model.

## When NOT to use

- Cells are not from a cancerous lineage — the tool is designed to identify malignant cells from cancerous lineages and relies on prior identification of basic cell types
- Cancer type has unknown or controversial cell of origin (e.g., synovial sarcoma, certain brain tumors) — performance is untested
- Only log1p-normalized data is available and raw counts cannot be recovered, since the model expects normalize_total(target_sum=1e4) output without log transformation

## Inputs

- AnnData (.h5ad) single-cell RNA-seq count matrix with raw counts
- Pretrained model.joblib from Zenodo record 17888140
- Ordered feature list ordered_feature.tsv from Zenodo record 17888140
- Basic cell-type annotations identifying the cancerous lineage (prerequisite)

## Outputs

- Per-cell scMalignantFinder_prediction labels ('Normal'/'Malignant')
- Per-cell malignancy_probability (continuous predicted probability of malignancy)
- result AnnData returned by model.predict() containing prediction columns

## How to apply

Download the pretrained model.joblib and ordered_feature.tsv from Zenodo record 17888140 and place both in a pretrained_model directory. Load each dataset with scanpy and apply total-count normalization via sc.pp.normalize_total(adata, target_sum=1e4) (norm_type=True), deliberately skipping log-transformation since the model expects normalized (not log-transformed) counts. Instantiate the model with pretrain_dir pointing at the pretrained_model directory so the model and ordered feature list load automatically, then call model.load() and model.predict(). The prediction returns scMalignantFinder_prediction labels ('Normal'/'Malignant') and a continuous malignancy_probability per cell, which can be used to capture dynamic tumor-progression states. Because the pretrained features are ordered, the model handles feature alignment against ordered_feature.tsv automatically.

## Related tools

- **scMalignantFinder** (Pretrained classifier installed via pip; loads model.joblib + ordered_feature.tsv and predicts malignant vs normal cells with malignancy probability on normalized AnnData) — github.com/Jonyyqn/scMalignantFinder
- **scATOMIC** (Optional upstream pan-cancer cell type annotator integrated into the current version for automated annotation before malignancy scoring) — github.com/abelson-lab/scATOMIC

## Examples

```
adata = sc.read_h5ad('sample.h5ad'); sc.pp.normalize_total(adata, target_sum=1e4); model = scMalignantFinder(pretrain_dir='pretrained_model'); model.load(); result_adata = model.predict()
```

## Evaluation signals

- Prediction output must contain exactly the columns scMalignantFinder_prediction (values 'Normal'/'Malignant') and malignancy_probability
- On the article's 13 patient-derived test datasets, average accuracy should approximate 0.824 and balanced accuracy 0.732 against ground-truth labels
- Calibrated malignant training cells must show higher malignancy probability than the excluded 2.5% of cells that lost up- or downregulated cancer signatures (Supplementary Fig. 2a invariant)
- Input AnnData must be normalized with target_sum=1e4 and NOT log-transformed before predict(); verify non-integer, non-log-scale values
- model.load() must successfully resolve both model.joblib and ordered_feature.tsv from the pretrain_dir; missing files indicate a broken pretrain_dir

## Limitations

- Requires prerequisite identification of basic cell types; malignant cell identification is restricted to cancerous lineages
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- Pretrained model depends on feature ordering (ordered_feature.tsv); skipping normalization (target_sum=1e4) or adding log transformation would violate training-data conventions

## Evidence
