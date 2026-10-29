---
name: spatial-neighborhood-label-refinement
description: Use when annotating malignant vs non-malignant regions on spatial transcriptomics
  spots from spot-level malignancy probabilities and image features, and neighboring
  spots' labels need spatial-coherence refinement.
when_to_use_negative:
- Data is single-cell scRNA-seq without spatial coordinates — spatial neighborhood
  refinement is undefined; use the single-cell malignant cell identification mode
  instead
- Cell-type annotation for all lineages is needed — scMalignantFinder identifies malignant
  cells only from cancerous lineages and requires prerequisite identification of basic
  cell types
- Cancer with unknown/controversial origin (e.g. synovial sarcoma, certain brain tumors)
  — the article states capabilities require further testing there
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Provides the pretrained classifier (model.load/model.predict), utils.aucell_cal,
    spatial.image_cal, and spatial.region_identification with spatial_nn refinement
    used in this workflow
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Cancer-Finder
  role: Benchmark comparator for region-level malignant-region identification on the
    same eight ST datasets
  repo: Patchouli-M/SequencingCancerFinder
- name: scATOMIC
  role: Optional upstream pan-cancer cell type annotation integrated in the current
    version
  repo: github.com/abelson-lab/scATOMIC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:6c5b8947e9c46a0ce87f4da78681a57b065d7ab4a4098b87d47878d675aa2519
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spatial-neighborhood-label-refinement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines per-spot region labels in tumor spatial transcriptomics by combining three obs features (malignancy_probability, AUCell-based Malignant_up score, image_score) via hierarchical clustering and then smoothing cluster/region calls using a spatial neighborhood network (spatial_nn=True), as implemented in scMalignantFinder's spatial.region_identification.

## When to use

Use when annotating malignant vs non-malignant regions on spatial transcriptomics spots from spot-level malignancy probabilities and image features, and neighboring spots' labels need spatial-coherence refinement.

## When NOT to use

- Data is single-cell scRNA-seq without spatial coordinates — spatial neighborhood refinement is undefined; use the single-cell malignant cell identification mode instead
- Cell-type annotation for all lineages is needed — scMalignantFinder identifies malignant cells only from cancerous lineages and requires prerequisite identification of basic cell types
- Cancer with unknown/controversial origin (e.g. synovial sarcoma, certain brain tumors) — the article states capabilities require further testing there

## Inputs

- Spatial transcriptomics adata (per-spot counts, e.g. 10x/Visium or similar ST format) with spatial coordinates
- model.joblib pretrained scMalignantFinder model plus ordered_feature.tsv feature list (Zenodo 17888140)
- sc_malignant_deg.gmt malignant gene signature (Zenodo 17888140)
- H&E/image-derived spot features consumed by spatial.image_cal

## Outputs

- adata.obs['malignancy_probability'] per-spot malignancy probability
- adata.obs['Malignant_up'] AUCell signature score per spot
- adata.obs['image_score'] per spot
- adata.obs['cluster'] hierarchical clustering label (nclus=3)
- adata.obs['region_prediction'] spatially refined malignant/non-malignant region label per spot

## How to apply

First compute three per-spot features: run the pretrained scMalignantFinder classifier (model.joblib with ordered_feature.tsv, norm_type=True, no internal log-transformation) to get malignancy_probability; score the malignant signature with utils.aucell_cal using sc_malignant_deg.gmt from the Zenodo deposit (Zenodo record 17888140) to get Malignant_up; and extract image features with spatial.image_cal to get image_score. Then call spatial.region_identification on the three-feature list with hierarchical clustering (nclus=3), defining the malignant cluster by define_feature='Malignant_up'. Set spatial_nn=True so region labels are refined using spatial neighborhood information before writing the final cluster and region_prediction columns. The rationale is the article's multi-feature clustering framework that incorporates malignant features, image-based characteristics, and spatial neighborhood networks; validated across eight tumor ST datasets it yields ~0.783 accuracy and ~0.800 balanced accuracy versus pathologist annotations, exceeding Cancer-Finder by 0.067/0.062.

## Related tools

- **scMalignantFinder** (Provides the pretrained classifier (model.load/model.predict), utils.aucell_cal, spatial.image_cal, and spatial.region_identification with spatial_nn refinement used in this workflow) — github.com/Jonyyqn/scMalignantFinder
- **Cancer-Finder** (Benchmark comparator for region-level malignant-region identification on the same eight ST datasets) — Patchouli-M/SequencingCancerFinder
- **scATOMIC** (Optional upstream pan-cancer cell type annotation integrated in the current version) — github.com/abelson-lab/scATOMIC

## Examples

```
adata = spatial.region_identification(adata, features=['malignancy_probability', 'Malignant_up', 'image_score'], nclus=3, define_feature='Malignant_up', spatial_nn=True)
```

## Evaluation signals

- region_prediction accuracy versus expert pathologist annotations should approximate 0.783 and balanced accuracy approximately 0.800 across the eight tumor ST datasets
- Comparison against Cancer-Finder on the same datasets should show scMalignantFinder margins of roughly +0.067 accuracy and +0.062 balanced accuracy
- The malignant cluster must be defined via define_feature='Malignant_up' (highest AUCell signature activity), not by malignancy_probability alone
- Cluster count fixed at nclus=3 on the three-feature list ['malignancy_probability', 'Malignant_up', 'image_score']; fewer/more clusters indicates a misconfigured run
- With spatial_nn=True, refined labels should show spatial coherence (neighboring spots sharing labels) relative to raw cluster labels

## Limitations

- ST malignant-region identification depends on expert-curated ST data that is still accumulating; additional spatial features such as spatial distribution probability of malignant cells are not yet incorporated
- Requires prerequisite identification of basic cell types; malignant identification is restricted to cancerous lineages
- Untested for cancers with unknown or controversial origins such as synovial sarcoma and certain brain tumors

## Evidence
