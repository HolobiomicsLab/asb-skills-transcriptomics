---
name: tma-core-multiplexed-image-processing
description: Use when single-cell segmentation and per-compartment marker quantification
  is needed on multiplexed immunofluorescence or MIBI tissue images, especially TMA
  cores with columnar epithelial cells and incomplete membranes that nuclei-dilation
  or Voronoi methods undersegment.
when_to_use_negative:
- Input is already a per-cell marker intensity or feature table — segmentation is
  already done and quantification steps alone should be used
- Images lack the membrane markers MIRIAM relies on to extend incomplete cell membranes
  (e.g., single-channel nuclear-stain-only data)
- Goal is general image inspection or visualization rather than single-cell segmentation
  and per-compartment quantification of multiplexed tissue images
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Primary segmentation and quantification pipeline run on the TMA sub-image
    stacks
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: One of the two implementations of the MIRIAM pipeline used to generate segmentation
    masks
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative implementation of the MIRIAM pipeline, run on identical image
    stacks for cross-implementation comparison
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Required dependency used by the MIRIAM pipeline for pixel-classification tasks
  repo: https://www.ilastik.org/download.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:66addae1c8128e9ab48bde26665f6824fddc897018f895e0b91b1d1eb72b3d68
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
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tma-core-multiplexed-image-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Process manually annotated multiplexed immunofluorescence TMA core images with the MIRIAM pipeline (Matlab or Python implementation) to segment single cells and quantify marker intensity in nucleus, membrane, and cytoplasm compartments, benchmarked against manual annotation with DSC/JSC metrics.

## When to use

Use when single-cell segmentation and per-compartment marker quantification is needed on multiplexed immunofluorescence or MIBI tissue images, especially TMA cores with columnar epithelial cells and incomplete membranes that nuclei-dilation or Voronoi methods undersegment.

## When NOT to use

- Input is already a per-cell marker intensity or feature table — segmentation is already done and quantification steps alone should be used
- Images lack the membrane markers MIRIAM relies on to extend incomplete cell membranes (e.g., single-channel nuclear-stain-only data)
- Goal is general image inspection or visualization rather than single-cell segmentation and per-compartment quantification of multiplexed tissue images

## Inputs

- Multiplexed immunofluorescence TMA core image stacks (multi-channel)
- Manually annotated membrane/cell ground-truth masks for the TMA sub-images
- MIRIAM source code (Matlab and/or Python implementations) from github.com/Coffey-Lab/MIRIAM
- Ilastik installation for pixel-classification tasks

## Outputs

- Membrane and cell segmentation masks per TMA sub-image
- Per-cell marker intensity table quantified over entire cell, nucleus, membrane, and cytoplasm
- Per-image DSC and JSC agreement metrics against manual annotation (summary reported as mean ± standard deviation)

## How to apply

Install Ilastik (required by the MIRIAM pipeline for pixel-classification tasks) and obtain the MIRIAM source from github.com/Coffey-Lab/MIRIAM, which ships both a Matlab implementation and a Python implementation. Split each manually annotated TMA core into sub-image stacks — the reference study analyzed 240 sub-images across three annotated TMA cores — then run random-forest-based machine learning pixel classification to define nucleus, membrane, and cytoplasm compartments, followed by MIRIAM's novel method for extending incomplete cell membranes to separate cells with internal membranes. Quantify each marker as image intensity over the entire cell and per compartment, and optionally encode cell shapes with the autoencoder. To validate, compute image-level Dice (DSC) and Jaccard (JSC) similarity coefficients against manual annotation; the paper reports MIRIAM at DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08 on an image level. If cross-implementation consistency is the goal, run both the Matlab and Python implementations on identical image stacks and confirm membrane agreement falls within DSC 0.84 ± 0.06 and JSC 0.74 ± 0.09.

## Related tools

- **MIRIAM** (Primary segmentation and quantification pipeline run on the TMA sub-image stacks) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (One of the two implementations of the MIRIAM pipeline used to generate segmentation masks) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative implementation of the MIRIAM pipeline, run on identical image stacks for cross-implementation comparison) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Required dependency used by the MIRIAM pipeline for pixel-classification tasks) — https://www.ilastik.org/download.html

## Evaluation signals

- Image-level DSC against manual annotation should fall within the reported band 0.65 ± 0.08 across the analyzed sub-images
- Image-level JSC against manual annotation should fall within the reported band 0.48 ± 0.08
- Matlab vs. Python membrane segmentations on identical stacks should agree at DSC 0.84 ± 0.06 and JSC 0.74 ± 0.09, confirming 'largely similar results'
- Sub-image count should match the analyzed cohort (240 sub-images across the three manually annotated TMA cores)
- Segmentation must resolve cells towards the lumen — Voronoi-style undersegmentation (DSC 0.32 ± 0.11) indicates the membrane-extension step was not applied correctly

## Limitations

- No software version numbers for the Matlab or Python implementations are stated in the article, so exact version reproduction requires checking the repository
- Example data in the repository contains all generated files except the image stacks, which are too large and must be regenerated/downloaded by the user
- The pipeline requires Ilastik as an external dependency for pixel-classification tasks
- MIRIAM was validated on colorectal adenoma multiplexed immunofluorescence and a breast carcinoma MIBI dataset; other tissue types or marker panels may require retraining the pixel classifier

## Evidence
