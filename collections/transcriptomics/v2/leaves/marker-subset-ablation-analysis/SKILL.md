---
name: marker-subset-ablation-analysis
description: Use when you need to test how the marker composition of multiplexed immunofluorescence
  TMA data affects segmentation quality — e.g., to justify which markers are mandatory
  for MIRIAM's membrane-driven cell-border reconstruction.
when_to_use_negative:
- When the goal is absolute segmentation accuracy rather than relative robustness
  to marker composition — use a standard benchmarking skill instead.
- When the input lacks membrane markers entirely; the reference finding shows segmentation
  quality decreases when membrane markers and DAPI are excluded, so ablation subsets
  missing them are uninformative for the full pipeline's expected performance.
- When the imaging modality is not multiplexed tissue imaging (e.g., single-channel
  or single-cell suspensions), since the method's rationale is membrane-marker redundancy
  across a multiplexed panel.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Pipeline run on each subset's pixel-classification output to produce cell
    segmentations for comparison
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random forest pixel classifier retrained separately per marker subset to define
    cellular compartments
- name: Python
  role: Implementation used to run the MIRIAM pipeline and compute DSC/JSC metrics
    per 512x512 sub-image
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:d30c4eb6a8a3d845b766b0af95d3d8c540e462804f775ca8f292734944ab781f
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marker-subset-ablation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ablation workflow that retrains MIRIAM's Ilastik random-forest pixel classifier under different marker-subset configurations and quantifies segmentation fidelity against manual cell-border annotations using image-level Dice (DSC) and Jaccard (JSC) coefficients.

## When to use

Use when you need to test how the marker composition of multiplexed immunofluorescence TMA data affects segmentation quality — e.g., to justify which markers are mandatory for MIRIAM's membrane-driven cell-border reconstruction.

## When NOT to use

- When the goal is absolute segmentation accuracy rather than relative robustness to marker composition — use a standard benchmarking skill instead.
- When the input lacks membrane markers entirely; the reference finding shows segmentation quality decreases when membrane markers and DAPI are excluded, so ablation subsets missing them are uninformative for the full pipeline's expected performance.
- When the imaging modality is not multiplexed tissue imaging (e.g., single-channel or single-cell suspensions), since the method's rationale is membrane-marker redundancy across a multiplexed panel.

## Inputs

- Multiplexed immunofluorescence TMA core image stacks (multiple markers incl. membrane markers and DAPI)
- Manual cell-border annotations for the TMA cores
- Ilastik pixel-classification project files (retrained per marker subset)

## Outputs

- MIRIAM segmentations per marker-subset configuration
- Per-sub-image Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC)
- Aggregated per-subset DSC/JSC table (e.g., segmentation_ablation_metrics.csv)

## How to apply

Load the multiplexed immunofluorescence TMA core images with manual cell-border annotations and tile them into 512x512 sub-image evaluation sets (the reference study used 240 sub-images across three annotated cores). Retrain the Ilastik random forest pixel classifier separately for each marker subset: full 15-marker + DAPI configuration, membrane-marker-only subsets, each leave-one-out exclusion, NaKATPase+DAPI only, and no-membrane configurations. Run MIRIAM (Python implementation at github.com/Coffey-Lab/MIRIAM) on each subset's pixel-classification output to generate segmentations, then compute DSC and JSC against the manual annotations for every sub-image. Aggregate image-level DSC/JSC as mean ± standard deviation per subset and export the results table (e.g., segmentation_ablation_metrics.csv). Validate that the full-marker configuration reproduces the reported baseline (DSC 0.65 ± 0.08, JSC 0.48 ± 0.08) and that degradation is limited to subsets lacking membrane markers and DAPI.

## Related tools

- **MIRIAM** (Pipeline run on each subset's pixel-classification output to produce cell segmentations for comparison) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random forest pixel classifier retrained separately per marker subset to define cellular compartments)
- **Python** (Implementation used to run the MIRIAM pipeline and compute DSC/JSC metrics per 512x512 sub-image) — github.com/Coffey-Lab/MIRIAM

## Examples

```
python run_ablation.py --markers all,membrane,leave_one_out,NaKATPase+DAPI --tiles 512x512 --metrics dsc,jsc --out segmentation_ablation_metrics.csv
```

## Evaluation signals

- Full-marker (15 markers + DAPI) configuration must reproduce the reported baseline: image-level DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08.
- Leave-one-out exclusions and membrane-only subsets should show minimal degradation in DSC/JSC relative to the baseline.
- Subsets excluding membrane markers and DAPI, or using only NaKATPase+DAPI, should show a clear drop in similarity to manual cell-border annotations.
- Segmentation_ablation_metrics.csv should contain mean ± standard deviation image-level DSC and JSC for every marker-subset configuration evaluated.

## Limitations

- Baseline thresholds are specific to the 512x512 sub-image evaluation scheme used in the reference study (240 sub-images across three annotated TMA cores); different annotation density or tiling may shift the DSC/JSC range.
- Deep-learning nuclei detection was not incorporated, which the authors note may limit initial seed quality; ablation conclusions do not account for such augmentations.
- The reference implementation leaves software versions unstated for the Matlab and Python implementations, so exact reproducibility of pixel-classifier behavior across versions is not guaranteed.
- Ablation findings assume membrane-marker redundancy; panels with a single membrane marker may not show the same robustness pattern.

## Evidence

- [results] MIRIAM segmentation was robust to marker composition: [evidence span withheld — anchor status: not_found]
