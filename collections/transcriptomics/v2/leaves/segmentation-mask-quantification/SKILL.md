---
name: segmentation-mask-quantification
description: Use when two segmentation implementations (or a pipeline and ground-truth
  annotations) have produced membrane/cell masks over the same multiplexed immunofluorescence
  TMA sub-images and per-image agreement statistics (DSC/JSC) must be computed and
  aggregated.
when_to_use_negative:
- Input is raw multiplexed image data rather than already-generated segmentation masks
  — use the MIRIAM segmentation pipeline itself first
- The goal is measuring accuracy against manual ground truth of cell segmentation
  quality (MIRIAM DSC 0.65 ± 0.08, Mesmer 0.59 ± 0.10, Voronoi 0.32 ± 0.11 vs annotations)
  rather than cross-implementation reproducibility between Matlab and Python masks
- Masks come from different image stacks or differently registered images — pixel-wise
  DSC/JSC is meaningless without identical sub-image alignment
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Source of the paired membrane segmentation masks being compared; pipeline
    with Matlab and Python implementations executed on identical TMA sub-image stacks
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Matlab (R2018b per source card) implementation of MIRIAM executed to generate
    the reference-side segmentation masks
- name: Python
  role: Python (3.7 per source card) implementation of MIRIAM executed to generate
    the comparison-side segmentation masks and to compute per-image DSC/JSC/correlation
    metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:32a94ba228245c35aeb89571074c5c2f4cb3320dd8c6a62847b805497a867055
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
  - Python
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# segmentation-mask-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between cell/membrane segmentation masks produced by alternative implementations of the MIRIAM pipeline (Matlab vs Python) using image-correlation, Dice similarity coefficient (DSC), and Jaccard similarity coefficient (JSC) computed per sub-image of manually annotated TMA cores.

## When to use

Use when two segmentation implementations (or a pipeline and ground-truth annotations) have produced membrane/cell masks over the same multiplexed immunofluorescence TMA sub-images and per-image agreement statistics (DSC/JSC) must be computed and aggregated.

## When NOT to use

- Input is raw multiplexed image data rather than already-generated segmentation masks — use the MIRIAM segmentation pipeline itself first
- The goal is measuring accuracy against manual ground truth of cell segmentation quality (MIRIAM DSC 0.65 ± 0.08, Mesmer 0.59 ± 0.10, Voronoi 0.32 ± 0.11 vs annotations) rather than cross-implementation reproducibility between Matlab and Python masks
- Masks come from different image stacks or differently registered images — pixel-wise DSC/JSC is meaningless without identical sub-image alignment

## Inputs

- Membrane segmentation masks from the Matlab implementation of MIRIAM (Matlab R2018b)
- Membrane segmentation masks from the Python implementation of MIRIAM (Python 3.7)
- Manually annotated TMA core image stacks (240 sub-images across three cores) from multiplexed immunofluorescence data

## Outputs

- Per-image image-correlation, DSC, and JSC values for each of the 240 sub-image mask pairs
- Summary agreement statistics: mean ± SD membrane DSC (expected 0.84 ± 0.06) and JSC (expected 0.74 ± 0.09)
- segmentation_agreement_metrics.csv containing per-image and summary agreement metrics

## How to apply

Run each implementation of MIRIAM — the Matlab and the Python versions from github.com/Coffey-Lab/MIRIAM — on the identical manually annotated TMA core image stacks (240 sub-images across three TMA cores) to obtain paired membrane segmentation masks. For each sub-image, treat one mask as reference and compute three metrics between mask pairs: image-correlation, Dice similarity coefficient, and Jaccard similarity coefficient (JSC = intersection over union of membrane pixels; DSC = 2·|A∩B|/(|A|+|B|), equivalent to F1 on binary membrane labels). Aggregate the 240 per-image values into mean ± standard deviation and compare against the reported band: membrane DSC 0.84 ± 0.06 and JSC 0.74 ± 0.09 (Supplemental Figure 6). A mean DSC within roughly one standard deviation of 0.84 confirms the 'largely similar results' conclusion between Matlab and Python implementations. Write per-image and summary values to segmentation_agreement_metrics.csv for auditability.

## Related tools

- **MIRIAM** (Source of the paired membrane segmentation masks being compared; pipeline with Matlab and Python implementations executed on identical TMA sub-image stacks) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Matlab (R2018b per source card) implementation of MIRIAM executed to generate the reference-side segmentation masks)
- **Python** (Python (3.7 per source card) implementation of MIRIAM executed to generate the comparison-side segmentation masks and to compute per-image DSC/JSC/correlation metrics)

## Examples

```
from sklearn.metrics import f1_score, jaccard_score; dsc = f1_score(matlab_mask.ravel(), python_mask.ravel()); jsc = jaccard_score(matlab_mask.ravel(), python_mask.ravel())
```

## Evaluation signals

- Computed mean membrane DSC across all 240 sub-images falls within the reported band 0.84 ± 0.06
- Computed mean membrane JSC falls within the reported band 0.74 ± 0.09
- Per-image metrics are computed on exactly the 240 sub-images from the three manually annotated TMA cores, with one metric triple (correlation, DSC, JSC) per sub-image
- Matlab and Python implementations were run on the identical image stacks, so mask dimensions and pixel counts match before comparison
- Output file segmentation_agreement_metrics.csv contains both per-image rows and a summary mean ± SD row

## Limitations

- Agreement is at the membrane-mask level; the paper's 'largely similar results' claim (Supplemental Figure 6) covers implementation equivalence, not segmentation accuracy versus ground truth
- The article itself does not state software version numbers for the Matlab or Python implementations (Matlab R2018b and Python 3.7 come from the source card, not the article text)
- The GitHub repo README provides Matlab code requiring Ilastik for some tasks and example data without image stacks (too large, must be regenerated), so masks must be produced by the user before comparison

## Evidence
