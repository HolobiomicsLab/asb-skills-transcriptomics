---
name: segmentation-dice-jaccard-evaluation
description: Use when per-cell segmentation masks from multiplexed tissue images (MIBI
  or multiplexed IF) must be scored against manually annotated ground truth via image-level
  DSC and JSC distributions.
when_to_use_negative:
- Input is raw multiplexed image stacks without any manually annotated ground-truth
  masks — DSC/JSC cannot be computed without reference labels.
- The goal is downstream per-cell marker quantification (intensity in nucleus/membrane/cytoplasm
  compartments) or autoencoder-based cell shape characterization rather than segmentation
  accuracy evaluation.
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Produces the segmentation masks (pixel classification plus membrane extension)
    whose image-level DSC and JSC are computed
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation of the MIRIAM pipeline and computation of per-image DSC/JSC
    metrics written to miriam_mibi_segmentation_metrics.csv
- name: Matlab
  role: Alternative MIRIAM implementation with largely similar segmentation results
    used for the same DSC/JSC evaluation
- name: Mesmer
  role: Deep learning comparator segmentation method whose DSC (0.59 ± 0.10) and JSC
    (0.42 ± 0.08) distributions benchmark MIRIAM's scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:c8dee4963bfa29fbdba430e490f8acb0ca164f1dc3789f9c6e16438d1186cb11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Mesmer
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# segmentation-dice-jaccard-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies image-level single-cell segmentation quality on manually annotated multiplexed tissue imaging TMA cores (e.g., MIBI breast carcinoma, multiplexed IF colorectal adenoma) using Dice Similarity Coefficient (DSC) and Jaccard Similarity Coefficient (JSC) per sub-image. It is the evaluation layer of the MIRIAM pipeline, benchmarking segmentation against ground truth and alternative methods (Mesmer, Voronoi).

## When to use

Use when per-cell segmentation masks from multiplexed tissue images (MIBI or multiplexed IF) must be scored against manually annotated ground truth via image-level DSC and JSC distributions.

## When NOT to use

- Input is raw multiplexed image stacks without any manually annotated ground-truth masks — DSC/JSC cannot be computed without reference labels.
- The goal is downstream per-cell marker quantification (intensity in nucleus/membrane/cytoplasm compartments) or autoencoder-based cell shape characterization rather than segmentation accuracy evaluation.

## Inputs

- Manually annotated multiplexed imaging TMA core images (e.g., breast carcinoma MIBI images, 2048x2048 pixels at 0.5 um per pixel, 41 patients, 36 markers; or multiplexed IF colorectal adenoma images)
- Per-cell segmentation masks from MIRIAM (or comparator methods such as Mesmer or Voronoi dilation)
- Ground-truth single-cell annotations for the TMA cores

## Outputs

- Per-sub-image Dice Similarity Coefficient (DSC) values
- Per-sub-image Jaccard Similarity Coefficient (JSC) values
- miriam_mibi_segmentation_metrics.csv containing image-level DSC and JSC per segmentation result
- Summary statistics (mean ± standard deviation) comparable to Supplemental Figure 7 and Fig. 2H distributions

## How to apply

Run the MIRIAM pipeline (Matlab or Python implementations show largely similar results) — random forest-based machine learning pixel classification to define nucleus, membrane, and cytoplasm compartments, followed by the membrane-extension method for cells with internal aberrant membranes — to produce per-cell masks. Split manually annotated TMA cores into sub-images (the reference analysis used 240 sub-images across three annotated TMA cores); compute DSC and JSC per sub-image against the ground-truth annotations. Write per-image DSC and JSC values to a CSV (e.g., miriam_mibi_segmentation_metrics.csv) and report mean ± standard deviation, comparing against benchmark methods such as Mesmer and Voronoi. Correct application reproduces the paper's reference distributions: MIRIAM DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08, versus Mesmer DSC 0.59 ± 0.10 / JSC 0.42 ± 0.08 and Voronoi DSC 0.32 ± 0.11 / JSC 0.20 ± 0.07.

## Related tools

- **MIRIAM** (Produces the segmentation masks (pixel classification plus membrane extension) whose image-level DSC and JSC are computed) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation of the MIRIAM pipeline and computation of per-image DSC/JSC metrics written to miriam_mibi_segmentation_metrics.csv)
- **Matlab** (Alternative MIRIAM implementation with largely similar segmentation results used for the same DSC/JSC evaluation)
- **Mesmer** (Deep learning comparator segmentation method whose DSC (0.59 ± 0.10) and JSC (0.42 ± 0.08) distributions benchmark MIRIAM's scores)

## Evaluation signals

- Mean image-level DSC for MIRIAM masks falls near 0.65 ± 0.08 and JSC near 0.48 ± 0.08, consistent with the paper's manually annotated TMA core results.
- MIRIAM's DSC/JSC distribution exceeds Mesmer's (DSC 0.59 ± 0.10, JSC 0.42 ± 0.08) and clearly exceeds Voronoi's (DSC 0.32 ± 0.11, JSC 0.20 ± 0.07).
- Metrics are computed per sub-image across all analyzed sub-images (reference: 240 sub-images across three manually annotated TMA cores) rather than on whole cores only.
- Output CSV (e.g., miriam_mibi_segmentation_metrics.csv) contains one DSC and JSC row per image with values in [0, 1].
- On the breast carcinoma MIBI dataset, reproduced distributions are consistent with the representative results in Supplemental Figure 7.

## Limitations

- DSC/JSC require manually annotated ground truth; the reference evaluation used only three manually annotated TMA cores (240 sub-images).
- MIRIAM segmentation performance depends on the quality of random forest pixel classification; the article notes deep learning methods for nuclei detection were not implemented and might improve initial seeds.
- The breast cancer MIBI dataset had lower image quality than the colorectal adenoma dataset, which may depress segmentation scores.

## Evidence
