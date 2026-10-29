---
name: dice-coefficient-segmentation-accuracy-evaluation
description: Use when predicted cell segmentation masks (e.g. MIRIAM, Mesmer, or nuclei-based
  Voronoi) must be quantitatively scored against manually annotated cell borders on
  tiled multiplexed immunofluorescence TMA images.
when_to_use_negative:
- Input segmentation has no paired manual cell-border annotations, so no overlap metric
  against ground truth can be computed
- Goal is qualitative visual correspondence (e.g. 'generally high correspondence between
  MIRIAM and Mesmer segmentation in epithelial cells') rather than quantitative benchmarking
  against ground truth
- Input data is a downstream feature/quantification table (per-cell marker intensities,
  t-SNE inputs) rather than raw segmentation masks to validate
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Segmentation method under evaluation; Python implementation run on each 512x512
    sub-image to produce cell masks scored against manual annotations
  repo: github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Deep-learning segmentation baseline; run on DAPI and NaKATPase channels on
    the same sub-images and scored by DSC/JSC
- name: Voronoi
  role: Nuclei-seeded tessellation baseline segmentation; scored by DSC/JSC against
    manual cell borders
- name: Matlab
  role: Alternative MIRIAM implementation producing largely similar segmentation results
    used in comparison
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:3976085c447c40516479a7eddc4595d46fa8c6c1af425311d9e0d8262d10546b
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
  - Voronoi
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# dice-coefficient-segmentation-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates single-cell segmentation accuracy on multiplexed tissue images by computing Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) of predicted cell masks against manually annotated cell borders, per 512x512 sub-image and per individual cell, as used to benchmark MIRIAM, Mesmer, and Voronoi on colonic adenoma TMA cores.

## When to use

Use when predicted cell segmentation masks (e.g. MIRIAM, Mesmer, or nuclei-based Voronoi) must be quantitatively scored against manually annotated cell borders on tiled multiplexed immunofluorescence TMA images.

## When NOT to use

- Input segmentation has no paired manual cell-border annotations, so no overlap metric against ground truth can be computed
- Goal is qualitative visual correspondence (e.g. 'generally high correspondence between MIRIAM and Mesmer segmentation in epithelial cells') rather than quantitative benchmarking against ground truth
- Input data is a downstream feature/quantification table (per-cell marker intensities, t-SNE inputs) rather than raw segmentation masks to validate

## Inputs

- 512x512 tiled sub-images of multiplexed immunofluorescence TMA cores
- Manual ground-truth cell boundary annotations on the same sub-images
- Predicted cell segmentation masks per method (MIRIAM, Mesmer, Voronoi) on identical sub-image grid

## Outputs

- Per-sub-image DSC and JSC values for each segmentation method
- Per-individual-cell DSC and JSC values for each method
- Aggregated mean ± standard deviation of DSC/JSC at image and cell levels (e.g. MIRIAM 0.65±0.08/0.48±0.08; Mesmer 0.59±0.10/0.42±0.08; Voronoi 0.32±0.11/0.20±0.07)
- segmentation_evaluation_metrics.csv

## How to apply

Tile each manually annotated TMA core into 512x512 sub-images (the published scheme used 240 sub-images across three cores) so predictions and ground truth are aligned at identical pixel resolution. Run each segmentation method (MIRIAM Python implementation, Mesmer on DAPI/NaKATPase channels, or nuclei-seeded Voronoi tessellation) on the same sub-images, then compute DSC = 2|A∩B|/(|A|+|B|) and JSC = |A∩B|/|A∪B| between predicted and manual cell borders both per sub-image and per individual cell. Aggregate mean ± standard deviation at image level and cell level per method; expect higher cell-level than image-level scores (e.g. MIRIAM 0.65±0.08 DSC image-level vs 0.67±0.14 DSC cell-level). Use the resulting ranking to judge segmentation quality — in the published benchmark MIRIAM outperformed Mesmer, which outperformed Voronoi, at both levels.

## Related tools

- **MIRIAM** (Segmentation method under evaluation; Python implementation run on each 512x512 sub-image to produce cell masks scored against manual annotations) — github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Deep-learning segmentation baseline; run on DAPI and NaKATPase channels on the same sub-images and scored by DSC/JSC)
- **Voronoi** (Nuclei-seeded tessellation baseline segmentation; scored by DSC/JSC against manual cell borders)
- **Matlab** (Alternative MIRIAM implementation producing largely similar segmentation results used in comparison) — github.com/Coffey-Lab/MIRIAM

## Examples

```
python -c "import numpy as np; dsc = 2*np.logical_and(pred, gt).sum()/(pred.sum()+gt.sum()); jsc = np.logical_and(pred, gt).sum()/np.logical_or(pred, gt).sum()"
```

## Evaluation signals

- 240 sub-images are analyzed across the three manually annotated TMA cores before metric aggregation
- Cell-level DSC/JSC exceed image-level DSC/JSC for the same method (e.g. MIRIAM 0.67±0.14 vs 0.65±0.08 DSC)
- Aggregated values fall within published ranges: MIRIAM image-level DSC 0.65±0.08 / JSC 0.48±0.08; Mesmer 0.59±0.10 / 0.42±0.08; Voronoi 0.32±0.11 / 0.20±0.07
- Method ranking reproduces MIRIAM > Mesmer > Voronoi at both image and cell levels
- All per-sub-image, per-cell, and aggregated metrics are written to segmentation_evaluation_metrics.csv

## Limitations

- Nuclei-based Voronoi undersegments cells towards the lumen because it does not take cell membranes into account, yielding low overlap scores (image-level DSC 0.32±0.11)
- No software version numbers stated for the Matlab or Python implementations used in the comparison
- Deep learning methods for nuclei detection were not implemented, which may limit the initial seeds used for segmentation being scored

## Evidence
