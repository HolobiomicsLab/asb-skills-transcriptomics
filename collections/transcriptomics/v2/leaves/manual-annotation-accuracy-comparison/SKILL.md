---
name: manual-annotation-accuracy-comparison
description: Use when you have manually annotated cell borders for multiplexed imaging
  TMA cores and need to quantify how closely an automated segmentation (MIRIAM, Mesmer,
  Voronoi, or marker-subset variants) matches them.
when_to_use_negative:
- When no manual cell-border ground truth exists for the imaging data — DSC/JSC against
  annotations cannot be computed
- When the comparison target is intracellular marker quantification or cell-shape
  characterization rather than segmentation-border agreement
- When images are not tiled to the same 512x512 evaluation grid, so segmentations
  and annotations cannot be spatially aligned per sub-image
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Segmentation pipeline whose output masks are scored against manual annotations
    using DSC/JSC; Python implementation available at github.com/Coffey-Lab/MIRIAM
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation used to run MIRIAM segmentation and compute per-sub-image DSC/JSC
    metrics
  repo: github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Alternative deep-learning segmentation method benchmarked against manual annotations
    with the same DSC/JSC protocol
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:35d386cf6e813f366bd21cb85e766a85f4dbda6fcb391b5fde9488f776065bcb
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
  - Mesmer
  - Python
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# manual-annotation-accuracy-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates automated cell-border segmentations (e.g., MIRIAM, Mesmer, Voronoi) against manual cell-border annotations on tiled sub-images of multiplexed immunofluorescence TMA cores, using image-level Dice (DSC) and Jaccard (JSC) similarity coefficients.

## When to use

Use when you have manually annotated cell borders for multiplexed imaging TMA cores and need to quantify how closely an automated segmentation (MIRIAM, Mesmer, Voronoi, or marker-subset variants) matches them.

## When NOT to use

- When no manual cell-border ground truth exists for the imaging data — DSC/JSC against annotations cannot be computed
- When the comparison target is intracellular marker quantification or cell-shape characterization rather than segmentation-border agreement
- When images are not tiled to the same 512x512 evaluation grid, so segmentations and annotations cannot be spatially aligned per sub-image

## Inputs

- Multiplexed immunofluorescence TMA core image stacks (multiple markers plus DAPI)
- Manual cell-border annotations for the annotated TMA cores
- Automated segmentation label masks (MIRIAM, Mesmer, or Voronoi outputs) aligned to the same 512x512 sub-image grid

## Outputs

- Per-sub-image DSC and JSC values for each segmentation configuration
- Aggregated image-level DSC and JSC reported as mean ± standard deviation per method or marker subset (e.g., segmentation_ablation_metrics.csv)

## How to apply

Tile the multiplexed immunofluorescence TMA cores and their manual cell-border annotations into 512x512 sub-images (240 sub-images across the three annotated cores in the MIRIAM study). Run the segmentation method under test — e.g., the MIRIAM pipeline on Ilastik random-forest pixel-classification output — on each sub-image. For every 512x512 sub-image, compute the Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) between the automated segmentation and the manual annotation. Aggregate image-level DSC/JSC as mean ± standard deviation and compare across segmentation methods or, in marker-composition ablations, across retrained pixel-classifier marker subsets. The full-marker MIRIAM configuration serves as the reference baseline: image-level DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08, versus Mesmer (DSC 0.59 ± 0.10, JSC 0.42 ± 0.08) and Voronoi (DSC 0.32 ± 0.11, JSC 0.20 ± 0.07).

## Related tools

- **MIRIAM** (Segmentation pipeline whose output masks are scored against manual annotations using DSC/JSC; Python implementation available at github.com/Coffey-Lab/MIRIAM) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation used to run MIRIAM segmentation and compute per-sub-image DSC/JSC metrics) — github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Alternative deep-learning segmentation method benchmarked against manual annotations with the same DSC/JSC protocol)

## Evaluation signals

- Full-marker MIRIAM configuration reproduces the reported baseline of image-level DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08
- Exactly 240 512x512 sub-images are evaluated across the three manually annotated TMA cores
- DSC and JSC are reported per sub-image and aggregated as mean ± standard deviation, not as single pooled values
- Segmentation quality is minimally degraded for leave-one-out marker subsets and membrane-marker-only subsets, but drops when membrane markers and DAPI are excluded or only NaKATPase + DAPI are used
- Benchmark comparisons rank consistently: MIRIAM (DSC 0.65) > Mesmer (DSC 0.59) > Voronoi (DSC 0.32)

## Limitations

- Results of the Matlab and Python implementations of MIRIAM show largely similar results, but no software version numbers are stated in the document
- The high-level MIRIAM baseline (DSC 0.65) is measured on only three manually annotated TMA cores, so the evaluation set is limited to 240 sub-images
- Manual annotation quality itself bounds the achievable DSC/JSC; the paper does not report inter-annotator variability

## Evidence
