---
name: multiplexed-image-cell-segmentation
description: Use when segmenting individual cells from multiplexed immunofluorescence
  or MIBI tissue images where columnar/epithelial morphology causes dilation- or Voronoi-based
  methods to undersegment, and per-cell marker quantification in nucleus/membrane/cytoplasm
  compartments is needed.
when_to_use_negative:
- Input is already a per-cell feature/intensity table — this skill operates on raw
  multiplexed image stacks prior to quantification
- Cells are round and well-isolated such that nuclei-seeded Voronoi tessellation suffices
  — the article shows Voronoi undersegments columnar epithelial cells toward the lumen,
  but Voronoi is simpler when membranes are not informative
- Only nuclei detection is needed — the article notes deep learning nuclei detection
  was not implemented in MIRIAM
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: 'Core pipeline: pixel classification, internal-membrane resegmentation, per-cell
    marker quantification, and shape characterization'
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random forest pixel classification of cellular compartments, required by the
    MIRIAM pipeline
  repo: https://www.ilastik.org/download.html
- name: Python
  role: Alternative MIRIAM implementation; results largely similar to the Matlab implementation
    (Supplemental Figure 6)
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Primary MIRIAM implementation language per the repository README
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:a2092e37f42af4791544bc8e0e4b2b63af41127769115e8b77d4f8a8e0685c5f
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
  - Mesmer
  - Python
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# multiplexed-image-cell-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for segmenting single cells in multiplexed tissue imaging (e.g., multiplexed immunofluorescence TMA cores, MIBI) using the MIRIAM pipeline: random forest pixel classification to define cellular compartments, watershed seeding, and a novel resegmentation step that extends incomplete internal membranes.

## When to use

Use when segmenting individual cells from multiplexed immunofluorescence or MIBI tissue images where columnar/epithelial morphology causes dilation- or Voronoi-based methods to undersegment, and per-cell marker quantification in nucleus/membrane/cytoplasm compartments is needed.

## When NOT to use

- Input is already a per-cell feature/intensity table — this skill operates on raw multiplexed image stacks prior to quantification
- Cells are round and well-isolated such that nuclei-seeded Voronoi tessellation suffices — the article shows Voronoi undersegments columnar epithelial cells toward the lumen, but Voronoi is simpler when membranes are not informative
- Only nuclei detection is needed — the article notes deep learning nuclei detection was not implemented in MIRIAM

## Inputs

- Multiplexed immunofluorescence or MIBI image stacks (e.g., TMA cores with 15 markers plus DAPI)
- Trained Ilastik random forest pixel-classification probability maps
- Optional manual cell-border annotations for benchmarking (tiled into 512x512 sub-images)

## Outputs

- Full-image cell segmentation label mask (nucleus/cell borders per cell)
- Per-cell marker intensity table quantified over the entire cell and in nucleus, membrane, and cytoplasm compartments
- Optional autoencoder latent vectors for cell shape characterization
- DSC/JSC evaluation metrics per sub-image and per cell (e.g., segmentation_evaluation_metrics.csv)

## How to apply

Run MIRIAM (Matlab or Python implementation from github.com/Coffey-Lab/MIRIAM, with Ilastik required for some tasks): first perform random forest-based machine learning pixel classification on all markers (e.g., 15 markers plus DAPI) to define cellular compartments and generate probability maps. Apply the membrane-extension resegmentation step ('connects the dots'): identify cells whose internal membrane segments occupy more than 10% of the cell area (also including Mucin 2-expressing goblet cells), compute internal-membrane segment angles and endpoints, and iteratively extend segments until they intersect the cell border or another extension, replacing the initial watershed boundaries of flagged cells. Finally quantify each marker by image intensity over the entire cell and in nucleus, membrane, and cytoplasm compartments, and optionally characterize cell shapes with an autoencoder neural network. Validate against manually annotated cell borders using Dice (DSC) and Jaccard (JSC) similarity coefficients per 512x512 sub-image and per cell.

## Related tools

- **MIRIAM** (Core pipeline: pixel classification, internal-membrane resegmentation, per-cell marker quantification, and shape characterization) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random forest pixel classification of cellular compartments, required by the MIRIAM pipeline) — https://www.ilastik.org/download.html
- **Python** (Alternative MIRIAM implementation; results largely similar to the Matlab implementation (Supplemental Figure 6)) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Primary MIRIAM implementation language per the repository README) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Full-marker configuration (15 markers + DAPI) on the annotated TMA cores should reproduce image-level DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08 (cell-level 0.67 ± 0.14 / 0.52 ± 0.15) against manual annotations
- Segmentation robustness: leaving out a single marker or using only membrane markers should cause minimal DSC/JSC change; excluding membrane markers and DAPI, or using only NaKATPase and DAPI, should decrease similarity
- Output label mask matches input image dimensions and no flagged cell retains internal membrane segments covering >10% of its area after resegmentation
- Segmentation quality should exceed Mesmer (DSC 0.59 ± 0.10 image-level) and Voronoi (DSC 0.32 ± 0.11 image-level) benchmarks on the same sub-images

## Limitations

- Example data in the repository omits image stacks (too large); they must be regenerated or downloaded separately from the linked Dropbox share
- Deep learning methods for nuclei detection were not implemented; adding them may help improve initial seeds
- t-SNE analysis of autoencoder shape latent vectors did not reveal discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5)
- No software version numbers stated for the Matlab or Python implementations; Ilastik must be installed separately

## Evidence
