---
name: pathologist-annotation-benchmark-comparison
description: Use when you have tumor ST spot-level predictions (e.g., malignancy_probability,
  AUCell signature scores, image scores) plus pathologist-annotated malignant regions
  and need accuracy/balanced-accuracy benchmarking against expert labels or a competing
  classifier.
when_to_use_negative:
- Input is scRNA-seq without spatial coordinates or pathology images — the multi-feature
  region clustering requires malignancy_probability, Malignant_up, and image_score;
  for non-spatial data use single-cell malignant-cell identification instead
- Cancer type has unknown or controversial origin (e.g., synovial sarcoma, certain
  brain tumors) — the tool's capabilities were not validated there
- Malignant cells have not been distinguished from basic cell types — the method relies
  on prerequisite identification of basic cell types and applies only to cancerous
  lineages
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Primary classifier producing malignancy_probability, Malignant_up AUCell scores,
    image scores, and region_prediction labels evaluated against pathologist annotations
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Cancer-Finder
  role: Competing baseline; its region-level accuracy and balanced accuracy are benchmarked
    on the same eight tumor ST datasets
  repo: Patchouli-M/SequencingCancerFinder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:3c586421269b9d3d0dc25c9186b6392d1d2b896215c7522f02dafbf9e3e2fe9c
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
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pathologist-annotation-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks model-predicted malignant regions in spatial transcriptomics against expert pathologist annotations by computing region-level accuracy and balanced accuracy, and against competing tools such as Cancer-Finder, following the scMalignantFinder eight-dataset evaluation.

## When to use

Use when you have tumor ST spot-level predictions (e.g., malignancy_probability, AUCell signature scores, image scores) plus pathologist-annotated malignant regions and need accuracy/balanced-accuracy benchmarking against expert labels or a competing classifier.

## When NOT to use

- Input is scRNA-seq without spatial coordinates or pathology images — the multi-feature region clustering requires malignancy_probability, Malignant_up, and image_score; for non-spatial data use single-cell malignant-cell identification instead
- Cancer type has unknown or controversial origin (e.g., synovial sarcoma, certain brain tumors) — the tool's capabilities were not validated there
- Malignant cells have not been distinguished from basic cell types — the method relies on prerequisite identification of basic cell types and applies only to cancerous lineages

## Inputs

- Tumor spatial transcriptomics datasets (h5ad, spot-level) with pathologist-annotated malignant regions
- Pretrained scMalignantFinder model.joblib and ordered_feature.tsv from Zenodo record 17888140
- Malignant gene signature sc_malignant_deg.gmt (Zenodo 17888140)
- Cancer-Finder region-level predictions for the same datasets

## Outputs

- Per-spot region_prediction labels (cluster and region_prediction columns in adata.obs)
- Region-level accuracy and balanced accuracy vs. pathologist annotations per dataset
- Benchmark comparison table vs. Cancer-Finder (accuracy and balanced-accuracy deltas)

## How to apply

Run the pretrained scMalignantFinder classifier (model.joblib with ordered_feature.tsv, norm_type=True, no internal log-transformation) on each tumor ST dataset to obtain a per-spot malignancy_probability. Add two complementary obs features per spot: AUCell scores for the malignant signature (utils.aucell_cal with sc_malignant_deg.gmt from Zenodo record 17888140) and image-derived features (spatial.image_cal). Aggregate the three features ['malignancy_probability', 'Malignant_up', 'image_score'] with hierarchical clustering (nclus=3), define the malignant cluster by define_feature='Malignant_up', and refine labels with spatial_nn=True to emit cluster and region_prediction columns. Compare region_prediction labels against pathologist annotations across the eight datasets (five cancer types), reporting accuracy and balanced accuracy; benchmark against Cancer-Finder on the same datasets and verify the reported margins (accuracy +0.067, balanced accuracy +0.062).

## Related tools

- **scMalignantFinder** (Primary classifier producing malignancy_probability, Malignant_up AUCell scores, image scores, and region_prediction labels evaluated against pathologist annotations) — github.com/Jonyyqn/scMalignantFinder
- **Cancer-Finder** (Competing baseline; its region-level accuracy and balanced accuracy are benchmarked on the same eight tumor ST datasets) — Patchouli-M/SequencingCancerFinder

## Examples

```
model.load(); result_adata = model.predict()
```

## Evaluation signals

- Mean accuracy against pathologist annotations across the eight datasets approximates 0.783
- Mean balanced accuracy approximates 0.800
- scMalignantFinder exceeds Cancer-Finder by ~0.067 accuracy and ~0.062 balanced accuracy
- Each dataset's adata.obs contains 'cluster' and 'region_prediction' columns after spatial.region_identification with nclus=3 and spatial_nn=True
- Malignant cluster is identified via define_feature='Malignant_up', so the selected cluster must have the highest mean Malignant_up AUCell score

## Limitations

- Region-level ST evaluation depends on expert-curated ST annotations, which are not yet broadly available; future enhancements could add expert-curated ST data and spatial features
- Additional spatial features such as spatial distribution probability of malignant cells are not yet incorporated into malignant-region identification
- The classifier requires prerequisite identification of basic cell types and identifies malignant cells only from cancerous lineages

## Evidence
