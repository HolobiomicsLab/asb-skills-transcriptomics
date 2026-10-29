---
name: watershed-segmentation-replacement
description: Use when multiplexed tissue images contain columnar epithelial cells
  whose watershed segmentation is fragmented by internal (aberrant) membrane segments,
  and cell objects must be remerged/split per MIRIAM's resegmentation logic.
when_to_use_negative:
- Images whose cells lack internal membrane segments exceeding the >10% cell-area
  threshold (no flagged candidates), where the initial watershed output is already
  adequate
- When only a per-cell feature/intensity table is available and no image label mask
  or probability maps exist to resegment
- When the goal is whole-image nuclei detection rather than replacing aberrant internal-membrane
  boundaries of existing watershed objects
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Provides the internal membrane identification and resegmentation ('extension
    of incomplete cell membranes') algorithm that replaces watershed boundaries
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Primary implementation language of the Coffey-Lab/MIRIAM repository used to
    run the resegmentation
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation for the same resegmentation behavior, with
    largely similar results to Matlab (Supplemental Figure 6)
- name: ilastik
  role: Required by the Matlab pipeline for pixel-classification tasks feeding probability
    maps into resegmentation
  repo: https://www.ilastik.org/download.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:a2b549cac2321a8ff147b082ee4f1c05f89124a6cbd0bfd7783c103331fc6b56
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
  - ilastik
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# watershed-segmentation-replacement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM's resegmentation method that replaces initial watershed boundaries for cells whose segmentation was split by internal membranes, by detecting internal membrane segment endpoints and extending them ('connecting the dots') until they intersect the cell border or another extension.

## When to use

Use when multiplexed tissue images contain columnar epithelial cells whose watershed segmentation is fragmented by internal (aberrant) membrane segments, and cell objects must be remerged/split per MIRIAM's resegmentation logic.

## When NOT to use

- Images whose cells lack internal membrane segments exceeding the >10% cell-area threshold (no flagged candidates), where the initial watershed output is already adequate
- When only a per-cell feature/intensity table is available and no image label mask or probability maps exist to resegment
- When the goal is whole-image nuclei detection rather than replacing aberrant internal-membrane boundaries of existing watershed objects

## Inputs

- Random forest pixel-classification probability maps (from ilastik-assisted MIRIAM stage)
- Initial watershed cell label mask (upstream MIRIAM segmentation output)
- Multiplexed immunofluorescence image stacks (e.g., colorectal adenoma IF or MIBI breast carcinoma data)

## Outputs

- Updated full-image cell label mask with internal-membrane-split cells resegmented
- Per-cell segmentation labels suitable for downstream compartment quantification (nucleus, membrane, cytoplasm)

## How to apply

Load the random forest pixel-classification probability maps and the initial watershed label mask from the upstream MIRIAM segmentation stage. Flag candidate cells whose internal membrane segments comprise more than 10% of the cell area, and additionally include Mucin 2-expressing goblet cells. For each candidate, compute the angles of internal membrane segments, locate their endpoints, and iteratively extend each segment from its endpoints along the computed angles until it intersects either the cell border or an extension from another internal membrane. Merge the resegmented cells back into the full-image label mask, replacing the initial watershed segmentation only for the flagged cells; the pipeline exists in Matlab and Python implementations (Matlab requires ilastik for pixel classification) with largely similar results per Supplemental Figure 6.

## Related tools

- **MIRIAM** (Provides the internal membrane identification and resegmentation ('extension of incomplete cell membranes') algorithm that replaces watershed boundaries) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Primary implementation language of the Coffey-Lab/MIRIAM repository used to run the resegmentation) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation for the same resegmentation behavior, with largely similar results to Matlab (Supplemental Figure 6))
- **ilastik** (Required by the Matlab pipeline for pixel-classification tasks feeding probability maps into resegmentation) — https://www.ilastik.org/download.html

## Evaluation signals

- Output label mask exists and matches the input image dimensions
- No flagged cell retains internal membrane segments covering more than 10% of its area after resegmentation
- Total cell count is preserved for unflagged regions (only flagged cells' watershed labels are replaced)
- Cell count and objects match the initial watershed mask outside candidate cells; behavior checked against the Supplemental Fig. 2 flowchart since no standalone benchmark is reported
- Matlab and Python implementations yield largely similar resegmentation results (Supplemental Figure 6)

## Limitations

- Deep learning methods for nuclei detection were not implemented and could otherwise improve initial seeds feeding the segmentation
- No software version numbers are stated for the Matlab or Python implementations
- No standalone benchmark is reported for the resegmentation step itself; validation relies on the Supplemental Fig. 2 flowchart and downstream DSC/JSC comparisons

## Evidence
