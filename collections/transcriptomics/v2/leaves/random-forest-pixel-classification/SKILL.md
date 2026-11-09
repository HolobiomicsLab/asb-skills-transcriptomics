---
name: random-forest-pixel-classification
description: Use when multiplexed immunofluorescence or MIBI tissue images need pixel-level
  compartment labeling (nucleus, membrane, cytoplasm) to seed single-cell segmentation
  and enable per-cell marker quantification.
when_to_use_negative:
- Input lacks membrane-localized markers (or NaKATPase + DAPI alone) — the article
  shows excluding membrane markers and DAPI, or using only NaKATPase and DAPI, markedly
  decreases similarity to manually annotated cell borders
- Input is already a per-cell segmentation or feature table — this skill operates
  at the raw pixel level of multiplexed image stacks
- Single-channel or non-multiplexed images where compartments cannot be distinguished
  by marker intensity
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Pipeline whose initial stage is the random forest pixel classification; its
    Python implementation consumes the classifier output and applies membrane extension
    to separate cells with internal membranes
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Required dependency for the random forest-based machine learning pixel classification
    step of the MIRIAM pipeline
  repo: https://www.ilastik.org/download.html
- name: Python
  role: Implementation language of the MIRIAM pipeline variant used to run pixel classification
    and downstream segmentation; largely similar results to the Matlab implementation
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:8210cdc44b63b2caa24e9132c895bec31d345644b83af5ca62fcc865cd669944
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# random-forest-pixel-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Supervised random forest-based machine learning pixel classification (via Ilastik) that labels each pixel of multiplexed tissue images as nucleus, membrane, or cytoplasm. It is the initial input stage of the MIRIAM pipeline, feeding the novel membrane-extension method for separating cells with internal aberrant membranes.

## When to use

Use when multiplexed immunofluorescence or MIBI tissue images need pixel-level compartment labeling (nucleus, membrane, cytoplasm) to seed single-cell segmentation and enable per-cell marker quantification.

## When NOT to use

- Input lacks membrane-localized markers (or NaKATPase + DAPI alone) — the article shows excluding membrane markers and DAPI, or using only NaKATPase and DAPI, markedly decreases similarity to manually annotated cell borders
- Input is already a per-cell segmentation or feature table — this skill operates at the raw pixel level of multiplexed image stacks
- Single-channel or non-multiplexed images where compartments cannot be distinguished by marker intensity

## Inputs

- Multiplexed tissue image stacks (e.g., 15-marker + DAPI multiplexed immunofluorescence TMA cores; or 36-marker breast carcinoma MIBI TMA images, 2048x2048 pixels at 0.5 um per pixel)
- Manual pixel/cell-border annotations for training and evaluation (tiled into 512x512 sub-images)
- Ilastik pixel-classification project (.ilp) defining compartment label classes

## Outputs

- Per-pixel compartment probability maps / class labels (nucleus, membrane, cytoplasm) per marker image
- Pixel-classification output consumed by the MIRIAM membrane-extension segmentation to produce per-cell masks
- Image-level segmentation metrics (Dice Similarity Coefficient and Jaccard Similarity Coefficient) written to CSV (e.g., miriam_mibi_segmentation_metrics.csv, segmentation_ablation_metrics.csv)

## How to apply

Train an Ilastik random forest pixel classifier on the multiplexed marker image stack, annotating pixels belonging to nucleus, membrane, and cytoplasm compartments; MIRIAM's Python implementation (github.com/Coffey-Lab/MIRIAM) then consumes the classifier's pixel-probability output. Use all available markers plus DAPI when possible — for the colorectal adenoma TMA this was 15 markers + DAPI — since marker composition strongly affects quality: the full-marker configuration yielded image-level DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08 against manual annotations. For robustness or ablation studies, retrain the classifier separately per marker subset (leave-one-out, membrane-markers-only, NaKATPase+DAPI only, or no membrane markers) and rerun the downstream MIRIAM membrane-extension segmentation for each. Evaluate by tiling annotated TMA cores into 512x512 sub-images (240 sub-images across three cores) and computing per-image Dice and Jaccard similarity coefficients between automated and manual cell borders.

## Related tools

- **MIRIAM** (Pipeline whose initial stage is the random forest pixel classification; its Python implementation consumes the classifier output and applies membrane extension to separate cells with internal membranes) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Required dependency for the random forest-based machine learning pixel classification step of the MIRIAM pipeline) — https://www.ilastik.org/download.html
- **Python** (Implementation language of the MIRIAM pipeline variant used to run pixel classification and downstream segmentation; largely similar results to the Matlab implementation) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Full-marker (15 markers + DAPI) configuration reproduces the reported baseline image-level DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08 against manual cell-border annotations
- Leave-one-out marker exclusion or membrane-markers-only subsets produce minimal change in DSC/JSC relative to the full-marker baseline
- Excluding membrane markers and DAPI, or using only NaKATPase and DAPI, shows a measurable decrease in DSC/JSC — an expected degradation pattern that confirms the classifier is genuinely using membrane signal
- Applied to the breast carcinoma MIBI dataset, the resulting segmentation distribution is consistent with the representative results in Supplemental Figure 7
- Matlab vs Python implementation outputs show largely similar results (Supplemental Figure 6)

## Limitations

- Segmentation quality degrades when membrane markers are unavailable or when only NaKATPase and DAPI are used, since the classifier cannot delineate cell borders
- Deep learning methods for nuclei detection were not implemented; the article notes their addition may help improve initial seeds
- No software version numbers are stated for the Matlab or Python implementations, and no changelog is available in the document
- Example data in the repository omits the raw image stacks (too large); users must regenerate or download them separately
- The MIBI breast carcinoma dataset has lower image quality than the colorectal adenoma dataset used for benchmarking

## Evidence
