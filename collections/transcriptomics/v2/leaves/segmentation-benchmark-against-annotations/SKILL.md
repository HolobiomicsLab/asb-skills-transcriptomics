---
name: segmentation-benchmark-against-annotations
description: Use when manually annotated cell boundaries exist for multiplexed tissue
  images and you need quantitative agreement metrics (DSC/JSC) to compare candidate
  segmentation methods at image and individual-cell level.
when_to_use_negative:
- Input images lack manual ground-truth cell boundary annotations, so no DSC/JSC agreement
  can be computed
- The goal is cell phenotyping (marker quantification, t-SNE clustering) rather than
  validating segmentation accuracy against annotations
- Images are single-channel or lack the membrane marker channels (e.g., NaKATPase)
  needed by methods like Mesmer
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Candidate segmentation method benchmarked against manual annotations (Python
    implementation run on each 512x512 sub-image)
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Deep-learning segmentation baseline run on DAPI and NaKATPase channels of
    the same sub-images
- name: Python
  role: Implementation language for running the MIRIAM pipeline and computing DSC/JSC
    evaluation metrics
- name: Matlab
  role: Alternative MIRIAM implementation; results show largely similar performance
    to the Python implementation
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:87ceac44f45bc944908e60f809b7407622ac9f0d1d1fdb800b45e3078016165d
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
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# segmentation-benchmark-against-annotations

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cell segmentation methods (MIRIAM, Mesmer, nuclei-based Voronoi) against manually annotated cell borders on multiplexed immunofluorescence TMA cores by computing per-sub-image and per-cell Dice (DSC) and Jaccard (JSC) similarity coefficients, establishing whether resegmentation improves on dilation/Voronoi baselines for irregular columnar epithelial cells.

## When to use

Use when manually annotated cell boundaries exist for multiplexed tissue images and you need quantitative agreement metrics (DSC/JSC) to compare candidate segmentation methods at image and individual-cell level.

## When NOT to use

- Input images lack manual ground-truth cell boundary annotations, so no DSC/JSC agreement can be computed
- The goal is cell phenotyping (marker quantification, t-SNE clustering) rather than validating segmentation accuracy against annotations
- Images are single-channel or lack the membrane marker channels (e.g., NaKATPase) needed by methods like Mesmer

## Inputs

- Manually annotated multiplexed immunofluorescence TMA core images with ground-truth cell boundary annotations
- 512x512 tiled sub-images derived from the annotated TMA cores
- Channel stacks for each segmentation method (e.g., DAPI and NaKATPase channels for Mesmer)
- Segmentation masks from each candidate method (MIRIAM, Mesmer, Voronoi)

## Outputs

- Per-sub-image DSC and JSC values for each method
- Per-individual-cell DSC and JSC values for each method
- Aggregated mean ± standard deviation DSC and JSC at image and cell levels
- segmentation_evaluation_metrics.csv containing all metrics for all methods

## How to apply

Load the ground-truth manually annotated multiplexed immunofluorescence TMA core images together with their cell boundary annotations, then tile each core into 512x512 sub-images (240 sub-images across three manually annotated TMA cores in the reference study). Run each candidate method on the identical sub-images: the MIRIAM Python pipeline, Mesmer using the DAPI and NaKATPase channels, and nuclei-seeded Voronoi tessellation. Compute DSC and JSC per 512x512 sub-image and per individual cell against the manual annotations, then aggregate mean ± standard deviation. In the reference benchmark, MIRIAM reached image-level DSC 0.65 ± 0.08 / JSC 0.48 ± 0.08 (cell-level 0.67 ± 0.14 / 0.52 ± 0.15), Mesmer 0.59 ± 0.10 / 0.42 ± 0.08 (cell-level 0.61 ± 0.19 / 0.46 ± 0.18), and Voronoi 0.32 ± 0.11 / 0.20 ± 0.07 (cell-level 0.47 ± 0.21 / 0.33 ± 0.18); expect membrane-aware methods to outperform Voronoi, which undersegments cells towards the lumen because it ignores cell membranes. Write per-sub-image, per-cell, and aggregated metrics to a CSV (segmentation_evaluation_metrics.csv) for comparison against reported values.

## Related tools

- **MIRIAM** (Candidate segmentation method benchmarked against manual annotations (Python implementation run on each 512x512 sub-image)) — https://github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Deep-learning segmentation baseline run on DAPI and NaKATPase channels of the same sub-images)
- **Python** (Implementation language for running the MIRIAM pipeline and computing DSC/JSC evaluation metrics)
- **Matlab** (Alternative MIRIAM implementation; results show largely similar performance to the Python implementation) — https://github.com/Coffey-Lab/MIRIAM

## Examples

```
python run_segmentation_benchmark.py --miriam MIRIAM_masks/ --mesmer_dapi DAPI.tif --mesmer_membrane NaKATPase.tif --annotations ground_truth/ --tile 512 --out segmentation_evaluation_metrics.csv
```

## Evaluation signals

- Aggregated image-level DSC/JSC for MIRIAM should approximate 0.65 ± 0.08 / 0.48 ± 0.08 on the colonic adenoma TMA benchmark data
- Cell-level DSC/JSC for MIRIAM should approximate 0.67 ± 0.14 / 0.52 ± 0.15; Mesmer 0.61 ± 0.19 / 0.46 ± 0.18; Voronoi 0.47 ± 0.21 / 0.33 ± 0.18
- Sub-image count must total 240 across the three manually annotated TMA cores
- Ranking invariant: MIRIAM > Mesmer > Voronoi at both image-level and cell-level DSC/JSC
- Output CSV (segmentation_evaluation_metrics.csv) must contain per-sub-image, per-cell, and aggregated rows for all three methods

## Limitations

- Voronoi undersegments cells towards the lumen because it does not take into account cell membranes, so it is a weak baseline for columnar epithelial tissue
- MIRIAM's deep learning method for nuclei detection was not implemented; the addition of deep learning methods for nuclei detection may help improve initial seeds
- No software version numbers stated for the Matlab or Python implementations, complicating exact reproduction
- Matlab and Python MIRIAM implementations show only largely similar, not identical, results (Supplemental Figure 6)

## Evidence
