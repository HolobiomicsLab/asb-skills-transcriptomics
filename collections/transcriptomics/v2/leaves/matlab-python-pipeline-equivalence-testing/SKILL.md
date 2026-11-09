---
name: matlab-python-pipeline-equivalence-testing
description: Use when a pipeline has dual implementations (e.g., Matlab and Python)
  and you must verify they yield similar segmentations on identical image inputs before
  deploying either version for downstream quantification.
when_to_use_negative:
- Input is a single-implementation pipeline with no second codebase to compare against
  — there is nothing to test equivalence on.
- The goal is validating segmentation accuracy against ground-truth annotations (accuracy
  benchmarking, e.g., MIRIAM vs. Mesmer vs. Voronoi DSC of 0.65/0.59/0.32), not inter-implementation
  agreement.
- Image stacks differ between the two runs (different TMA cores or markers) — equivalence
  testing requires identical inputs for both implementations.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3365
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Segmentation pipeline under test; both implementations are executed on identical
    TMA core image stacks to produce comparable membrane masks
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Reference implementation runtime (R2018b) used to generate one side of the
    agreement comparison
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternate implementation runtime (Python 3.7) used to generate the other side
    of the agreement comparison
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:b18e2d9127556183f5f1698566f9113740aa35ce4b20bc3017e352cff42f1bf6
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
  - Python
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3365
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# matlab-python-pipeline-equivalence-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-implementation equivalence testing of a segmentation pipeline: running both the Matlab and Python implementations of MIRIAM on identical manually annotated multiplexed immunofluorescence TMA core image stacks and quantifying membrane segmentation agreement with Dice (DSC) and Jaccard (JSC) similarity coefficients.

## When to use

Use when a pipeline has dual implementations (e.g., Matlab and Python) and you must verify they yield similar segmentations on identical image inputs before deploying either version for downstream quantification.

## When NOT to use

- Input is a single-implementation pipeline with no second codebase to compare against — there is nothing to test equivalence on.
- The goal is validating segmentation accuracy against ground-truth annotations (accuracy benchmarking, e.g., MIRIAM vs. Mesmer vs. Voronoi DSC of 0.65/0.59/0.32), not inter-implementation agreement.
- Image stacks differ between the two runs (different TMA cores or markers) — equivalence testing requires identical inputs for both implementations.

## Inputs

- Manually annotated multiplexed immunofluorescence TMA core image stacks (240 sub-images across three TMA cores)
- MIRIAM pipeline source containing both Matlab and Python implementations (github.com/Coffey-Lab/MIRIAM)

## Outputs

- Per-sub-image image-correlation, DSC, and JSC values between Matlab- and Python-derived membrane segmentations
- Summary agreement statistics (DSC 0.84 ± 0.06, JSC 0.74 ± 0.09) in segmentation_agreement_metrics.csv
- Supplemental-Figure-6-style comparison of Matlab vs. Python membrane segmentation results

## How to apply

Obtain both pipeline implementations from the same source repo (github.com/Coffey-Lab/MIRIAM) and run each on the exact same image stacks — here 240 sub-images across three manually annotated TMA cores — generating membrane and cell segmentation masks with the Matlab (R2018b) build and the Python (3.7) build independently. For each sub-image, compare the two membrane segmentations pairwise using image correlation, Dice similarity coefficient, and Jaccard similarity coefficient rather than qualitative inspection, since pixel-level overlap is the equivalence criterion. Aggregate per-image metrics into mean ± standard deviation and check that the computed values fall within the reported agreement bands (DSC 0.84 ± 0.06, JSC 0.74 ± 0.09) to confirm 'largely similar results' as shown in Supplemental Figure 6. Write per-image and summary metrics to a CSV (e.g., segmentation_agreement_metrics.csv) so both image-level variability and the aggregate band are auditable.

## Related tools

- **MIRIAM** (Segmentation pipeline under test; both implementations are executed on identical TMA core image stacks to produce comparable membrane masks) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Reference implementation runtime (R2018b) used to generate one side of the agreement comparison) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternate implementation runtime (Python 3.7) used to generate the other side of the agreement comparison) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Computed membrane DSC mean falls within the reported band 0.84 ± 0.06 across the 240 sub-images
- Computed membrane JSC mean falls within the reported band 0.74 ± 0.09
- Agreement is characterized as 'largely similar results', consistent with Supplemental Figure 6 of the MIRIAM paper
- Per-image metrics (correlation, DSC, JSC) are present for all 240 sub-images, not a subset
- Both implementations ran on the exact same manually annotated image stacks — provenance of inputs is identical

## Limitations

- Agreement is membrane-level overlap, not quantification equivalence; cell-level marker intensity outputs are not compared
- No changelog found in the document, so implementation-version drift between repo commits is not tracked
- Matlab R2018b and Python 3.7 versions come from the task context, not the article itself; the paper does not state software version numbers
- Equivalence was established on multiplexed immunofluorescence TMA cores only; transferability to other platforms (e.g., the MIBI breast carcinoma dataset) is not demonstrated by this comparison

## Evidence
