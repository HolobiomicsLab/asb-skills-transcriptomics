---
name: cross-modality-segmentation-transfer
description: Use when a segmentation pipeline validated on one multiplexed imaging
  modality (e.g., MIBI, IMC, imaging mass cytometry) must be evaluated on another
  platform or marker set to demonstrate cross-tissue/platform transferability.
when_to_use_negative:
- Input data is the original multiplexed immunofluorescence colorectal adenoma dataset
  — that is the primary development/validation use case, not a transfer test.
- No manually annotated reference segmentation is available, so DSC/JSC cannot be
  computed to verify transfer quality.
- The goal is whole-image segmentation benchmarking against Mesmer or Voronoi baselines
  on a single modality rather than assessing cross-platform generalization.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2640
tools:
- name: MIRIAM
  role: Segmentation pipeline (random forest pixel classification + membrane extension)
    run on the transferred MIBI images
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation language used to execute the MIRIAM pipeline on MIBI data;
    shown to give largely similar results to the Matlab implementation (Supplemental
    Figure 6)
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Required by the MIRIAM Matlab pipeline for some tasks (e.g., random forest
    pixel classification); install from https://www.ilastik.org/download.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:f4e08783b1402bc5c61f8e6664c6a69df912eddcb3819522efd9d12375bb90ee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - Ilastik
  - MIRIAM
  - Python
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-modality-segmentation-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Test whether the MIRIAM segmentation pipeline — random forest pixel classification plus membrane extension for internal aberrant membranes — generalizes to a different imaging platform, marker set, and tissue type (breast carcinoma MIBI TMA images) than the multiplexed immunofluorescence colorectal adenoma data it was developed on.

## When to use

Use when a segmentation pipeline validated on one multiplexed imaging modality (e.g., MIBI, IMC, imaging mass cytometry) must be evaluated on another platform or marker set to demonstrate cross-tissue/platform transferability.

## When NOT to use

- Input data is the original multiplexed immunofluorescence colorectal adenoma dataset — that is the primary development/validation use case, not a transfer test.
- No manually annotated reference segmentation is available, so DSC/JSC cannot be computed to verify transfer quality.
- The goal is whole-image segmentation benchmarking against Mesmer or Voronoi baselines on a single modality rather than assessing cross-platform generalization.

## Inputs

- Breast carcinoma MIBI TMA images (Keren et al.; 41 patients, 36 markers, 2048x2048 pixels, 0.5 um per pixel) from the MIRIAM repository public deposit
- MIRIAM repository (github.com/Coffey-Lab/MIRIAM) with Python implementation and tutorial

## Outputs

- Per-cell segmentation masks for each MIBI TMA image (nucleus, membrane, cytoplasm compartments)
- Image-level Dice Similarity Coefficient (DSC) and Jaccard Similarity Coefficient (JSC) per MIBI image
- miriam_mibi_segmentation_metrics.csv containing DSC and JSC values per image for comparison to Supplemental Figure 7

## How to apply

Obtain the publicly described breast carcinoma MIBI TMA images (Keren et al.; 41 patients, 36 markers, 2048x2048 pixels at 0.5 um per pixel) from the deposit accompanying the Coffey-Lab/MIRIAM repository. Run the Python implementation of MIRIAM on the MIBI images, beginning with random forest-based machine learning pixel classification (via Ilastik) to define nucleus, membrane, and cytoplasm compartments from marker intensities. Apply the MIRIAM membrane-extension method to identify and separate cells with internal aberrant membranes, yielding per-cell segmentation masks for each TMA image. Compute image-level Dice Similarity Coefficient and Jaccard Similarity Coefficient per image, write them to miriam_mibi_segmentation_metrics.csv, and compare the distribution to the representative results in Supplemental Figure 7 to judge whether segmentation quality was retained on the new modality.

## Related tools

- **MIRIAM** (Segmentation pipeline (random forest pixel classification + membrane extension) run on the transferred MIBI images) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation language used to execute the MIRIAM pipeline on MIBI data; shown to give largely similar results to the Matlab implementation (Supplemental Figure 6)) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Required by the MIRIAM Matlab pipeline for some tasks (e.g., random forest pixel classification); install from https://www.ilastik.org/download.html)

## Examples

```
git clone https://github.com/Coffey-Lab/MIRIAM; python run_miriam.py --input breast_mibi_tma/ --metrics miriam_mibi_segmentation_metrics.csv
```

## Evaluation signals

- Segmentation quality on MIBI images is consistent with the representative MIBI results in Supplemental Figure 7 (high-quality single-cell segmentation on a different modality and marker set).
- Image-level DSC and JSC are computed for every MIBI image and written to miriam_mibi_segmentation_metrics.csv without missing entries.
- Per-cell masks correctly assign nucleus, membrane, and cytoplasm compartments and separate cells with internal aberrant membranes (the failure mode MIRIAM was designed to fix).
- Python and Matlab implementations of MIRIAM give largely similar results, as shown for the original dataset in Supplemental Figure 6.

## Limitations

- The breast carcinoma MIBI dataset has lower image quality than the colorectal adenoma dataset, which may affect transfer performance.
- No software version numbers are stated for the Matlab or Python implementations, so exact reproducibility may vary.
- Segmentation quality on the transferred dataset is demonstrated only in Supplemental Figure 7; the paper reports no DSC/JSC numbers for the MIBI case, so comparison is qualitative unless a reference annotation is supplied.

## Evidence
