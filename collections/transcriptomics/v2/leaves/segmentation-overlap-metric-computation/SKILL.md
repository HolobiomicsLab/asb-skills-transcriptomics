---
name: segmentation-overlap-metric-computation
description: Use when a predicted cell-border segmentation must be quantitatively
  compared against manual annotations (or another segmentation method) at the image
  level to report segmentation accuracy.
when_to_use_negative:
- The ground truth lacks cell-border annotation — metrics require a matched manual
  segmentation of the same field of view
- You need per-cell identity correspondence (matching individual cells) rather than
  mask-overlap agreement; DSC/JSC measure region overlap only
- Segmentation masks and annotations come from different fields of view or are not
  spatially registered
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Produces the automated cell segmentation masks that are scored against manual
    annotations using DSC/JSC
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Implementation language for running the MIRIAM pipeline and computing overlap
    metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:64823a385b9457039b454cee261cf5e5cb9ed4e4b12cd6569acdfeec1c9cd8fe
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
  - Python
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# segmentation-overlap-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pixel-level overlap metrics — Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) — between an automated cell segmentation and a ground-truth manual annotation to quantify segmentation quality, as done to validate MIRIAM against manually annotated cell borders.

## When to use

Use when a predicted cell-border segmentation must be quantitatively compared against manual annotations (or another segmentation method) at the image level to report segmentation accuracy.

## When NOT to use

- The ground truth lacks cell-border annotation — metrics require a matched manual segmentation of the same field of view
- You need per-cell identity correspondence (matching individual cells) rather than mask-overlap agreement; DSC/JSC measure region overlap only
- Segmentation masks and annotations come from different fields of view or are not spatially registered

## Inputs

- Predicted segmentation masks (e.g., MIRIAM output per 512x512 sub-image)
- Ground-truth manual cell-border annotations (binary masks) for the same sub-images
- Evaluation sub-image set derived by tiling annotated TMA cores (240 sub-images across three cores)

## Outputs

- Per-sub-image DSC and JSC values
- Image-level mean ± standard deviation DSC and JSC (e.g., written to a table such as segmentation_ablation_metrics.csv)

## How to apply

Treat each segmentation as a binary cell-border/cell-region mask and compute, per image, the DSC (twice the intersection over the sum of the two masks) and the JSC (intersection over union) against the manual annotation. Tile larger tissue images into fixed-size evaluation sub-images (here 512x512; 240 sub-images across three manually annotated TMA cores) so the metric is computed on comparable image units. Aggregate per-sub-image values as mean ± standard deviation to report an image-level score, e.g. MIRIAM's baseline DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08. The same metric can then be recomputed under ablations (marker subsets, alternative pipelines such as Mesmer or Voronoi) and compared to the baseline to detect degradation.

## Related tools

- **MIRIAM** (Produces the automated cell segmentation masks that are scored against manual annotations using DSC/JSC) — github.com/Coffey-Lab/MIRIAM
- **Python** (Implementation language for running the MIRIAM pipeline and computing overlap metrics)

## Evaluation signals

- Full-marker MIRIAM configuration reproduces the reported image-level baseline of DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08
- DSC and JSC are consistent with each other (JSC < DSC, roughly JSC ≈ DSC/(2−DSC) for identical masks)
- All 240 sub-images across the three annotated TMA cores yield a finite metric value (no missing or degenerate overlaps)
- Metric distributions across marker-subset ablations remain minimally degraded except when membrane markers (or membrane markers plus DAPI) are excluded

## Limitations

- Metrics are computed on 512x512 tiled sub-images, so they depend on the tiling scheme and may not capture whole-core segmentation behavior
- The reported evaluation is specific to multiplexed immunofluorescence TMA cores; performance on other platforms (e.g., MIBI breast carcinoma) was demonstrated separately but with the same marker-set sensitivity caveats
- Overlap metrics penalize any border disagreement equally and cannot distinguish nuclear-seed errors from membrane-extension errors

## Evidence
