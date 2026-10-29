---
name: jaccard-coefficient-cell-level-overlap-quantification
description: Use when evaluating how well an automated tissue-cell segmentation (e.g.
  MIRIAM, Mesmer, Voronoi) agrees with manual cell-boundary annotations on multiplexed
  images, at both the individual-cell and 512x512 sub-image level.
when_to_use_negative:
- Input is not a segmentation mask overlap problem — e.g. you are scoring marker-intensity
  quantification accuracy rather than cell-border agreement with manual annotations
- No manually annotated ground-truth cell borders exist for the multiplexed imaging
  data, so no reference overlap can be computed
- Comparison is at the whole-slide or multi-organ level rather than the tiled 512x512
  sub-image / individual-cell scheme used in the benchmark
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3395
tools:
- name: MIRIAM
  role: Segmentation method whose predicted cell masks are scored with cell-level
    and image-level JSC against manual annotations
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Deep-learning segmentation baseline run on DAPI and NaKATPase channels and
    benchmarked with JSC/DSC against the same manual annotations
- name: Python
  role: Implementation language for the MIRIAM pipeline used to generate masks scored
    by JSC (Matlab and Python implementations show largely similar results)
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Alternative MIRIAM implementation used for generating segmentation masks on
    multiplexed imaging data
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:47281f709df58f1071e5d246de25522e5c42b7d3c33518acf90e981b324eddb1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  repo_url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Mesmer
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# jaccard-coefficient-cell-level-overlap-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies single-cell segmentation accuracy by computing the Jaccard similarity coefficient (JSC) of cell-region overlap against manually annotated ground-truth cell borders, per individual cell and per image tile, alongside the Dice similarity coefficient (DSC).

## When to use

Use when evaluating how well an automated tissue-cell segmentation (e.g. MIRIAM, Mesmer, Voronoi) agrees with manual cell-boundary annotations on multiplexed images, at both the individual-cell and 512x512 sub-image level.

## When NOT to use

- Input is not a segmentation mask overlap problem — e.g. you are scoring marker-intensity quantification accuracy rather than cell-border agreement with manual annotations
- No manually annotated ground-truth cell borders exist for the multiplexed imaging data, so no reference overlap can be computed
- Comparison is at the whole-slide or multi-organ level rather than the tiled 512x512 sub-image / individual-cell scheme used in the benchmark

## Inputs

- Manually annotated cell-boundary ground-truth masks for multiplexed immunofluorescence TMA cores
- Automated cell segmentation masks (e.g. from MIRIAM, Mesmer, or Voronoi tessellation) on the same 512x512 sub-images

## Outputs

- Per-sub-image JSC (and DSC) values, e.g. MIRIAM image-level JSC 0.48 +/- 0.08
- Per-individual-cell JSC (and DSC) values, e.g. MIRIAM cell-level JSC 0.52 +/- 0.15
- Aggregated mean +/- standard deviation comparison table for all segmentation methods (segmentation_evaluation_metrics.csv)

## How to apply

Tile each annotated TMA core multiplexed immunofluorescence image into 512x512 sub-images to match the evaluation scheme (240 sub-images across the three manually annotated colonic adenoma TMA cores). Run each segmentation method (MIRIAM Python implementation, Mesmer using DAPI and NaKATPase channels, or nuclei-seeded Voronoi tessellation) on the same sub-images, then compute the Jaccard similarity coefficient of pixel overlap between each predicted cell mask and the manual ground-truth cell borders — per individual cell and aggregated per 512x512 sub-image — reporting mean +/- standard deviation. Compare against the paper's benchmarks to judge whether the skill was applied correctly: cell-level JSC of 0.52 +/- 0.15 for MIRIAM, 0.46 +/- 0.18 for Mesmer, and 0.33 +/- 0.18 for Voronoi; image-level JSC of 0.48 +/- 0.08 (MIRIAM), 0.42 +/- 0.08 (Mesmer), and 0.20 +/- 0.07 (Voronoi). Always compute the companion DSC in the same evaluation since MIRIAM outperformed Mesmer at both levels (image-level DSC 0.65 +/- 0.08 vs 0.59 +/- 0.10; cell-level 0.67 +/- 0.14 vs 0.61 +/- 0.19).

## Related tools

- **MIRIAM** (Segmentation method whose predicted cell masks are scored with cell-level and image-level JSC against manual annotations) — https://github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Deep-learning segmentation baseline run on DAPI and NaKATPase channels and benchmarked with JSC/DSC against the same manual annotations)
- **Python** (Implementation language for the MIRIAM pipeline used to generate masks scored by JSC (Matlab and Python implementations show largely similar results)) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Alternative MIRIAM implementation used for generating segmentation masks on multiplexed imaging data) — https://github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Cell-level JSC for MIRIAM on colonic adenoma TMA sub-images should fall near 0.52 +/- 0.15; deviations suggest a mismatch in tiling or mask alignment
- Image-level JSC ordering should be MIRIAM (0.48 +/- 0.08) > Mesmer (0.42 +/- 0.08) >> Voronoi (0.20 +/- 0.07); Voronoi undersegments cells towards the lumen because it ignores cell membranes
- Sub-image count must match the evaluation scheme: 240 sub-images across the three manually annotated TMA cores
- JSC and DSC must be reported together at both levels with mean +/- standard deviation, as in segmentation_evaluation_metrics.csv

## Limitations

- Voronoi undersegments cells towards the lumen since it does not take into account cell membranes, depressing its JSC/DSC scores
- Manual annotations are required; without ground-truth cell borders no overlap coefficient can be computed
- MIRIAM's deep-learning nuclei detection was not implemented and could improve initial seeds — nuclei errors propagate into the scored masks

## Evidence
