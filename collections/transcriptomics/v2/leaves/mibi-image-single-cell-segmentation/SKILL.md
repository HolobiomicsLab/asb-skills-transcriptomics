---
name: mibi-image-single-cell-segmentation
description: Use when segmenting single cells in MIBI or other multiplexed tissue
  imaging data (e.g., breast carcinoma TMA images) where cells have internal aberrant
  or incomplete membranes that defeat standard segmentation approaches.
when_to_use_negative:
- The input is already a per-cell feature/quantification table and no segmentation
  masks are needed
- The images are simple, uniformly shaped cells without internal or incomplete membranes,
  where simpler methods (dilation from nuclei, Voronoi tessellation) suffice — though
  note the article shows Voronoi undersegments cells towards the lumen
- 'The task is generic deep-learning segmentation rather than reproducing MIRIAM''s
  membrane-extension approach (e.g., Mesmer alone, which MIRIAM outperformed: DSC
  0.65 ± 0.08 vs 0.59 ± 0.10)'
edam_operation: http://edamontology.org/operation_3684
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: 'Core pipeline: random forest pixel classification followed by membrane extension
    to segment cells with internal aberrant membranes in MIBI images'
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation of the MIRIAM pipeline run on the MIBI images (largely similar
    results to the Matlab implementation)
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Alternative implementation of MIRIAM; the README describes Matlab code to
    run cell segmentation on multiplexed imaging data
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:0f237adaf75df23349f3e78bf86ad5f28981f8cf6fc402959480340066a544ff
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
  - Python
  edam_operation: http://edamontology.org/operation_3684
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mibi-image-single-cell-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the MIRIAM pipeline (Matlab or Python implementation) to Multiplexed Ion Beam Imaging (MIBI) breast carcinoma tissue images to produce single-cell segmentation masks, demonstrating cross-platform transfer from multiplexed immunofluorescence to a different modality and marker set.

## When to use

Use when segmenting single cells in MIBI or other multiplexed tissue imaging data (e.g., breast carcinoma TMA images) where cells have internal aberrant or incomplete membranes that defeat standard segmentation approaches.

## When NOT to use

- The input is already a per-cell feature/quantification table and no segmentation masks are needed
- The images are simple, uniformly shaped cells without internal or incomplete membranes, where simpler methods (dilation from nuclei, Voronoi tessellation) suffice — though note the article shows Voronoi undersegments cells towards the lumen
- The task is generic deep-learning segmentation rather than reproducing MIRIAM's membrane-extension approach (e.g., Mesmer alone, which MIRIAM outperformed: DSC 0.65 ± 0.08 vs 0.59 ± 0.10)

## Inputs

- Breast carcinoma MIBI TMA images (41 patients, 36 markers, 2048x2048 pixels at 0.5 um per pixel) from the public deposit accompanying github.com/Coffey-Lab/MIRIAM
- MIRIAM repository (github.com/Coffey-Lab/MIRIAM) with Matlab/Python pipeline code and example data (image stacks downloadable from the linked Dropbox deposit)

## Outputs

- Per-cell segmentation masks for each MIBI TMA image
- Per-image segmentation metrics (Dice Similarity Coefficient and Jaccard Similarity Coefficient), e.g., written to miriam_mibi_segmentation_metrics.csv

## How to apply

Obtain the breast carcinoma MIBI TMA images (41 patients, 36 markers, 2048x2048 pixels at 0.5 um per pixel; Keren et al.) from the public deposit accompanying github.com/Coffey-Lab/MIRIAM, and clone the repository. Clone MIRIAM and run the Python implementation (the article notes Matlab and Python implementations give largely similar results; the README notes the pipeline requires Ilastik for some tasks). First perform random forest-based machine learning pixel classification (via ilastik) to define cellular compartments — nucleus, membrane, cytoplasm — on the MIBI marker images. Then apply the MIRIAM membrane-extension method to identify and separate cells with internal aberrant membranes, producing per-cell segmentation masks for each TMA image. Finally compute image-level segmentation metrics — Dice Similarity Coefficient (DSC) and Jaccard Similarity Coefficient (JSC) — per image and compare against the reference performance of MIRIAM on the colorectal adenoma data (DSC 0.65 ± 0.08, JSC 0.48 ± 0.08) and the representative MIBI results in Supplemental Figure 7.

## Related tools

- **MIRIAM** (Core pipeline: random forest pixel classification followed by membrane extension to segment cells with internal aberrant membranes in MIBI images) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation of the MIRIAM pipeline run on the MIBI images (largely similar results to the Matlab implementation)) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Alternative implementation of MIRIAM; the README describes Matlab code to run cell segmentation on multiplexed imaging data) — github.com/Coffey-Lab/MIRIAM

## Examples

```
git clone https://github.com/Coffey-Lab/MIRIAM && python miriam_pipeline.py --input mibi_breast_carcinoma_tma --output miriam_mibi_segmentation_metrics.csv
```

## Evaluation signals

- Image-level DSC and JSC are computed for every segmented MIBI TMA image and written to miriam_mibi_segmentation_metrics.csv
- The DSC/JSC distribution is consistent with the representative MIBI results in Supplemental Figure 7 (plausibly comparable to the adenoma reference of DSC 0.65 ± 0.08, JSC 0.48 ± 0.08)
- Segmentation masks correctly separate cells with internal aberrant membranes rather than merging them (the failure mode of dilation/Voronoi approaches)
- Running the Matlab vs Python implementations yields largely similar segmentation results (per Supplemental Figure 6)
- MIRIAM performance is not degraded relative to Mesmer (DSC 0.59 ± 0.10, JSC 0.42 ± 0.08) despite the lower image quality of the MIBI breast carcinoma dataset

## Limitations

- The MIBI breast carcinoma dataset has lower image quality than the colorectal adenoma dataset and a different marker set, which may affect segmentation accuracy
- No software version numbers are stated for the Matlab or Python implementations
- The deep learning method for nuclei detection was not implemented in MIRIAM; the authors note it may help improve initial seeds
- The README indicates image stacks are not included in the example data (too large) and must be regenerated or downloaded separately

## Evidence
