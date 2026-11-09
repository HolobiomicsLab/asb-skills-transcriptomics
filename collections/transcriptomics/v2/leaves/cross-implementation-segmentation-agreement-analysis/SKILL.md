---
name: cross-implementation-segmentation-agreement-analysis
description: Use when the same segmentation pipeline exists in two implementations
  (e.g. Matlab vs Python) and you need to demonstrate the outputs are largely similar
  before distributing or adopting either version on multiplexed image data.
when_to_use_negative:
- Comparing two different segmentation algorithms (e.g. MIRIAM vs Mesmer vs Voronoi)
  against manual annotation — that is a benchmarking task with different metrics and
  reference ground truth, not cross-implementation agreement of the same pipeline
- Input data is not image segmentation masks from the same pipeline run on identical
  stacks (e.g. raw mzML, FASTQ, or single-cell feature tables) — mask-overlap metrics
  like DSC/JSC do not apply
- Only one implementation is available, so no pairwise mask comparison can be made
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3384
tools:
- name: MIRIAM
  role: The segmentation pipeline whose Matlab and Python implementations are run
    on identical TMA core image stacks to generate membrane and cell segmentation
    masks for pairwise comparison
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Execution environment for the Matlab (R2018b) MIRIAM implementation producing
    one set of membrane segmentation masks
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Execution environment for the Python (3.7) MIRIAM implementation producing
    the corresponding membrane segmentation masks on the identical image stacks
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:32c694e0b4eb29432186d154565fa3149ba6860df6f5b94df261e5d709daccb6
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-implementation-segmentation-agreement-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies whether two independent implementations of the same segmentation pipeline (here MIRIAM's Matlab R2018b and Python 3.7 versions) produce concordant cell/membrane segmentation masks on identical multiplexed immunofluorescence TMA core images, using image-correlation, Dice similarity coefficient (DSC), and Jaccard similarity coefficient (JSC).

## When to use

Use when the same segmentation pipeline exists in two implementations (e.g. Matlab vs Python) and you need to demonstrate the outputs are largely similar before distributing or adopting either version on multiplexed image data.

## When NOT to use

- Comparing two different segmentation algorithms (e.g. MIRIAM vs Mesmer vs Voronoi) against manual annotation — that is a benchmarking task with different metrics and reference ground truth, not cross-implementation agreement of the same pipeline
- Input data is not image segmentation masks from the same pipeline run on identical stacks (e.g. raw mzML, FASTQ, or single-cell feature tables) — mask-overlap metrics like DSC/JSC do not apply
- Only one implementation is available, so no pairwise mask comparison can be made

## Inputs

- Manually annotated multiplexed immunofluorescence TMA core image stacks (240 sub-images across three TMA cores in the reference study)
- Matlab-derived membrane and cell segmentation masks from MIRIAM (Matlab R2018b implementation)
- Python-derived membrane and cell segmentation masks from MIRIAM (Python 3.7 implementation, same input stacks)

## Outputs

- Per-sub-image image-correlation, DSC, and JSC values between Matlab- and Python-derived membrane segmentations
- Aggregated mean ± standard deviation agreement metrics (reference: DSC 0.84 ± 0.06, JSC 0.74 ± 0.09)
- Summary table of agreement metrics (e.g. segmentation_agreement_metrics.csv)

## How to apply

Obtain both implementations from the same source repository (github.com/Coffey-Lab/MIRIAM contains the Matlab and Python MIRIAM versions). Run the Matlab (R2018b) implementation on the manually annotated TMA core image stacks — in the reference study, 240 sub-images across three TMA cores — then run the Python (3.7) implementation on the identical stacks. For each sub-image, compute image-correlation, Dice similarity coefficient (DSC), and Jaccard similarity coefficient (JSC) between the Matlab- and Python-derived membrane segmentations, then aggregate per-image metrics into mean ± standard deviation. In the reference run, membrane agreement was DSC 0.84 ± 0.06 and JSC 0.74 ± 0.09 (Supplemental Figure 6), supporting the 'largely similar results' claim. Write per-image and summary metrics (e.g. segmentation_agreement_metrics.csv) and validate that computed means fall within the reported mean ± SD bands.

## Related tools

- **MIRIAM** (The segmentation pipeline whose Matlab and Python implementations are run on identical TMA core image stacks to generate membrane and cell segmentation masks for pairwise comparison) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Execution environment for the Matlab (R2018b) MIRIAM implementation producing one set of membrane segmentation masks) — github.com/Coffey-Lab/MIRIAM
- **Python** (Execution environment for the Python (3.7) MIRIAM implementation producing the corresponding membrane segmentation masks on the identical image stacks) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Computed mean membrane DSC falls within the reported mean ± standard deviation band (0.84 ± 0.06)
- Computed mean membrane JSC falls within the reported mean ± standard deviation band (0.74 ± 0.09)
- Both implementations are run on the identical input image stacks (same 240 sub-images across three TMA cores), so differences are attributable to implementation, not inputs
- Per-image agreement metrics are aggregated into mean ± standard deviation and written to a summary CSV for traceability
- Agreement is consistent with the reported qualitative claim of 'largely similar results' (Supplemental Figure 6)

## Limitations

- The article does not state software version numbers for the Matlab or Python implementations in the discussion; version details must come from the repository or study protocol
- Agreement was evaluated at the membrane-segmentation level on TMA cores; other compartments or tissue types (e.g. the breast carcinoma MIBI dataset) were not the basis of this comparison
- Results depend on the quality of the input image stacks; the README notes image stacks are not included in the example data and must be regenerated by the user (via Dropbox link)

## Evidence

- [results] Cross-implementation agreement claim: "Results of MIRIAM using Matlab or Python implementations show largely similar results (Supplemental Figure 6)."
- [results] Reference agreement metrics: "On an image level, 240 sub-images were analyzed across the three manually annotated TMA cores (Fig. 2H)."
- [readme] Repository source: "This pipeline requires Iltastik for some tasks. Download Ilastik here: https://www.ilastik.org/download.html"
