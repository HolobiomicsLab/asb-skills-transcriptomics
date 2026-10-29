---
name: spatial-transcriptomics-malignant-region-identification
description: Use when analyzing tumor ST spot-level data to locate malignant regions
  against pathologist annotations, especially when comparing spot-region calls to
  benchmarks such as Cancer-Finder.
when_to_use_negative:
- The input is single-cell scRNA-seq rather than spatial transcriptomics — use scMalignantFinder's
  malignant cell identification workflow instead of the ST region workflow.
- The task requires pan-cancer cell type annotation of the tumor microenvironment
  — use scATOMIC integration rather than region-level malignancy calling.
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma, certain
  brain tumors) — capabilities are untested there.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3678
tools:
- name: scMalignantFinder
  role: Pretrained classifier (model.predict) providing malignancy_probability per
    spot; spatial module (image_cal, region_identification, spatial_nn) for region
    calling
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Cancer-Finder
  role: Benchmark tool for region-level comparison on the same eight ST datasets
  repo: Patchouli-M/SequencingCancerFinder
- name: pySCENIC
  role: AUCell-based gene set scoring implementation underlying aucell_cal for the
    Malignant_up signature score
  repo: aertslab/pySCENIC
- name: scATOMIC
  role: Optional upstream pan-cancer cell type annotation integrated into scMalignantFinder
  repo: github.com/abelson-lab/scATOMIC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:de012f66d195939da08ded955a7f2628cc29d86874163e53d45bfe3e8705a883
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
  - pySCENIC
  - scATOMIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spatial-transcriptomics-malignant-region-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify malignant regions in tumor spatial transcriptomics (ST) data by combining scMalignantFinder's pretrained malignancy probability, AUCell scores of a malignant gene signature, and image-derived features through hierarchical clustering with spatial-neighbor refinement, without retraining the classifier.

## When to use

Use when analyzing tumor ST spot-level data to locate malignant regions against pathologist annotations, especially when comparing spot-region calls to benchmarks such as Cancer-Finder.

## When NOT to use

- The input is single-cell scRNA-seq rather than spatial transcriptomics — use scMalignantFinder's malignant cell identification workflow instead of the ST region workflow.
- The task requires pan-cancer cell type annotation of the tumor microenvironment — use scATOMIC integration rather than region-level malignancy calling.
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors) — capabilities are untested there.

## Inputs

- tumor spatial transcriptomics dataset(s) (h5ad) with per-spot counts and spatial coordinates
- pretrained scMalignantFinder model.joblib and ordered_feature.tsv (Zenodo record 17888140)
- sc_malignant_deg.gmt malignant gene signature file (Zenodo record 17888140)
- H&E image data paired with the ST spots (for image-derived features)
- pathologist region annotations for validation (optional, for accuracy assessment)

## Outputs

- per-spot malignancy_probability in adata.obs
- per-spot Malignant_up AUCell signature score in adata.obs
- per-spot image_score in adata.obs
- per-spot cluster labels (nclus=3) and region_prediction (malignant vs non-malignant region) columns in adata.obs
- accuracy and balanced accuracy of region_prediction vs pathologist annotations

## How to apply

For each tumor ST dataset, load the pretrained model (model.joblib with ordered_feature.tsv) via model.load() and model.predict() with norm_type=True and no internal log-transformation to obtain a per-spot malignancy_probability. Compute AUCell scores for the malignant signature using utils.aucell_cal with sc_malignant_deg.gmt from the Zenodo deposit (record 17888140), yielding the Malignant_up score, and extract image-derived per-spot features with spatial.image_cal to get the image_score. Run spatial.region_identification on the three obs features ['malignancy_probability', 'Malignant_up', 'image_score'] with hierarchical clustering (nclus=3), defining the malignant cluster by define_feature='Malignant_up', and set spatial_nn=True so region labels are refined using spatial neighborhood information. Compare the resulting region_prediction labels against expert pathologist annotations using accuracy and balanced accuracy; the article reports an average accuracy of 0.783 and balanced accuracy of 0.800 across eight datasets spanning five cancer types, exceeding Cancer-Finder by 0.067 and 0.062 respectively. The multi-feature framework works because combining malignant features, image-based characteristics, and spatial neighborhood networks improves region identification beyond any single feature.

## Related tools

- **scMalignantFinder** (Pretrained classifier (model.predict) providing malignancy_probability per spot; spatial module (image_cal, region_identification, spatial_nn) for region calling) — github.com/Jonyyqn/scMalignantFinder
- **Cancer-Finder** (Benchmark tool for region-level comparison on the same eight ST datasets) — Patchouli-M/SequencingCancerFinder
- **pySCENIC** (AUCell-based gene set scoring implementation underlying aucell_cal for the Malignant_up signature score) — aertslab/pySCENIC
- **scATOMIC** (Optional upstream pan-cancer cell type annotation integrated into scMalignantFinder) — github.com/abelson-lab/scATOMIC

## Examples

```
model.load(); result_adata = model.predict(); adata = utils.aucell_cal(adata, sc_malignant_deg.gmt); adata = spatial.image_cal(adata); adata = spatial.region_identification(adata, features=['malignancy_probability','Malignant_up','image_score'], nclus=3, define_feature='Malignant_up', spatial_nn=True)
```

## Evaluation signals

- region_prediction vs pathologist annotations yields average accuracy ≈ 0.783 and average balanced accuracy ≈ 0.800 across the eight tumor ST datasets
- benchmark comparison shows scMalignantFinder exceeds Cancer-Finder by ~0.067 accuracy and ~0.062 balanced accuracy on the same datasets
- malignant cluster is defined by define_feature='Malignant_up' — the cluster with highest Malignant_up AUCell score must be labeled malignant
- malignancy_probability is present in adata.obs and falls within [0, 1] after model.predict with norm_type=True
- with spatial_nn=True, refined region labels change only for spots whose spatial neighborhoods disagree with the initial cluster assignment

## Limitations

- scMalignantFinder is designed to identify malignant cells from cancerous lineages and relies on the prerequisite identification of basic cell types.
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors.
- ST malignant-region identification depends on continuous accumulation of expert-curated ST data, which is not yet available; additional spatial features such as spatial distribution probability of malignant cells are not yet incorporated.
- The classifier is extended to ST data without retraining, so spot-level accuracy is bounded by cell-level model transfer.

## Evidence
