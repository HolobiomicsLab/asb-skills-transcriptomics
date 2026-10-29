---
name: multi-feature-hierarchical-clustering-of-st-regions
description: Use when annotating spatial transcriptomics spots as malignant vs non-malignant
  regions in tumor ST datasets, especially when pathologist-annotated regions are
  the benchmark and no ST-retrained classifier exists.
when_to_use_negative:
- Data are single-cell scRNA-seq without spatial coordinates or image — use scMalignantFinder's
  cell-level malignant identification instead of region identification
- The goal is pan-cancer cell type annotation of non-cancerous lineages — the classifier
  is designed to identify malignant cells only within cancerous lineages after basic
  cell type identification
- The cancer type has unknown or controversial origin (e.g., synovial sarcoma, certain
  brain tumors) — capabilities are untested there
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Provides malignancy_probability per spot (model.predict) and the spatial.region_identification
    function that performs the hierarchical clustering and spatial_nn label refinement
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Cancer-Finder
  role: Benchmark comparator for region-level malignant annotation on the same eight
    ST datasets
  repo: Patchouli-M/SequencingCancerFinder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:84f30a3f2dc2f7a4ab52fd0de00a984bee15ff82d430b829d803b02b7bf9e082
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-feature-hierarchical-clustering-of-st-regions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies malignant regions in tumor spatial transcriptomics (ST) data by hierarchically clustering spots on three obs features — scMalignantFinder's malignancy_probability, the AUCell Malignant_up signature score, and an image_score — then refining labels with spatial neighborhood networks.

## When to use

Use when annotating spatial transcriptomics spots as malignant vs non-malignant regions in tumor ST datasets, especially when pathologist-annotated regions are the benchmark and no ST-retrained classifier exists.

## When NOT to use

- Data are single-cell scRNA-seq without spatial coordinates or image — use scMalignantFinder's cell-level malignant identification instead of region identification
- The goal is pan-cancer cell type annotation of non-cancerous lineages — the classifier is designed to identify malignant cells only within cancerous lineages after basic cell type identification
- The cancer type has unknown or controversial origin (e.g., synovial sarcoma, certain brain tumors) — capabilities are untested there

## Inputs

- Tumor spatial transcriptomics dataset (AnnData/h5ad) with raw or total-count-normalizable spot expression and spatial coordinates plus matched H&E image
- Pretrained scMalignantFinder model.joblib with ordered_feature.tsv (Zenodo record 17888140)
- Malignant gene signature sc_malignant_deg.gmt (Zenodo record 17888140)

## Outputs

- Per-spot malignancy_probability from model.predict()
- Per-spot Malignant_up AUCell signature score
- Per-spot image_score from spatial.image_cal
- Obs columns 'cluster' and 'region_prediction' labeling malignant vs non-malignant regions
- Region-level accuracy and balanced accuracy vs pathologist annotations

## How to apply

First compute the three per-spot features: run the pretrained scMalignantFinder classifier (model.joblib with ordered_feature.tsv, norm_type=True, no internal log-transformation) via model.load()/model.predict() to get malignancy_probability; compute AUCell scores for the malignant signature from sc_malignant_deg.gmt (Zenodo record 17888140) with utils.aucell_cal to get Malignant_up; extract image-derived features per spot with spatial.image_cal to get image_score. Then call spatial.region_identification on the obs features ['malignancy_probability', 'Malignant_up', 'image_score'] with hierarchical clustering at nclus=3, defining the malignant cluster by define_feature='Malignant_up', and set spatial_nn=True so region labels are refined using spatial neighborhood information. This multi-feature framework combines malignant features, image-based characteristics, and spatial neighborhood networks because scMalignantFinder's single-cell classifier was extended to ST without retraining. Validate by comparing region_prediction labels against pathologist annotations using accuracy and balanced accuracy; on eight tumor ST datasets spanning five cancer types this approach reached average accuracy 0.783 and balanced accuracy 0.800, exceeding Cancer-Finder by 0.067 and 0.062 respectively.

## Related tools

- **scMalignantFinder** (Provides malignancy_probability per spot (model.predict) and the spatial.region_identification function that performs the hierarchical clustering and spatial_nn label refinement) — github.com/Jonyyqn/scMalignantFinder
- **Cancer-Finder** (Benchmark comparator for region-level malignant annotation on the same eight ST datasets) — Patchouli-M/SequencingCancerFinder

## Examples

```
adata = spatial.region_identification(adata, features=["malignancy_probability", "Malignant_up", "image_score"], nclus=3, define_feature='Malignant_up', spatial_nn=True)
```

## Evaluation signals

- region_prediction column is produced alongside a cluster column in adata.obs with exactly nclus=3 clusters
- The malignant cluster is the one selected by define_feature='Malignant_up' (highest mean AUCell Malignant_up score)
- Accuracy against pathologist annotations is approximately 0.783 and balanced accuracy approximately 0.800 across eight tumor ST datasets
- Region-level margin over Cancer-Finder is approximately +0.067 accuracy and +0.062 balanced accuracy
- Each of the three obs features (malignancy_probability, Malignant_up, image_score) is present and non-null for every spot before clustering

## Limitations

- ST malignant-region identification depends on continuous accumulation of expert-curated ST data, which is not yet available; the classifier was extended to ST without retraining
- Additional spatial features, such as the spatial distribution probability of malignant cells, are not yet incorporated into malignant-region identification
- Requires prior identification of basic cell types, as malignant identification is restricted to cancerous lineages

## Evidence
