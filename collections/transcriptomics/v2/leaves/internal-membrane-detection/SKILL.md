---
name: internal-membrane-detection
description: Use when multiplexed immunofluorescence or MIBI tissue images yield watershed
  cell labels that are bisected by internal membrane segments (e.g., columnar epithelial
  or Mucin 2+ goblet cells) and need resegmentation into single objects.
when_to_use_negative:
- Cells already correctly segmented by the initial watershed step — only cells with
  >10% internal membrane area (or goblet cells) are candidates, so applying extension
  to unflagged cells risks altering valid boundaries
- Segmentation problems caused by poor nuclei seeds rather than internal membranes
  — the article notes deep learning nuclei detection was not implemented here and
  may be a better remedy for seed errors
- Non-tissue, isolated single-cell images without tissue-scale membrane architecture
  where internal aberrant membranes do not occur
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Pipeline implementing internal membrane detection and resegmentation ('extension
    of incomplete cell membranes')
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Required upstream for pixel-classification tasks whose probability maps feed
    membrane detection
  repo: https://www.ilastik.org/download.html
- name: Matlab
  role: Primary implementation language of the README-described MIRIAM segmentation
    code; Matlab and Python implementations give largely similar results
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative implementation of MIRIAM segmentation used to run the resegmentation
    algorithm
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:f250bb0ab286ff6bdf826d51cdedf4c65d53e2730b9d327b496e22b1ae3cd524
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
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# internal-membrane-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM's method for detecting and resegmenting cells whose watershed segmentation was incorrectly split by internal (aberrant) membranes in multiplexed tissue images, by 'connecting the dots' — extending incomplete membrane segments from their endpoints until they meet the cell border or another extension.

## When to use

Use when multiplexed immunofluorescence or MIBI tissue images yield watershed cell labels that are bisected by internal membrane segments (e.g., columnar epithelial or Mucin 2+ goblet cells) and need resegmentation into single objects.

## When NOT to use

- Cells already correctly segmented by the initial watershed step — only cells with >10% internal membrane area (or goblet cells) are candidates, so applying extension to unflagged cells risks altering valid boundaries
- Segmentation problems caused by poor nuclei seeds rather than internal membranes — the article notes deep learning nuclei detection was not implemented here and may be a better remedy for seed errors
- Non-tissue, isolated single-cell images without tissue-scale membrane architecture where internal aberrant membranes do not occur

## Inputs

- Random forest pixel-classification probability maps (from upstream MIRIAM/Ilastik stage)
- Initial watershed cell label mask
- Marker channels enabling identification of internal membrane segments and Mucin 2-expressing goblet cells

## Outputs

- Corrected full-image cell label mask with flagged cells resegmented and internal membrane boundaries replaced
- Per-cell labels reflecting merged, single-object cells after membrane extension

## How to apply

Load the random forest pixel-classification probability maps (generated upstream, e.g., via Ilastik as required by the MIRIAM pipeline) and the initial watershed label mask. Flag as candidates any cell whose internal membrane segments cover more than 10% of the cell area, and always include Mucin 2-expressing goblet cells. For each candidate, compute the angles of its internal membrane segments and locate their endpoints to determine extension directions. Iteratively extend each segment from its endpoints along those angles until it intersects either the cell border or another membrane extension, splitting the cell; merge resegmented labels back into the full-image mask, replacing the watershed boundaries of flagged cells only.

## Related tools

- **MIRIAM** (Pipeline implementing internal membrane detection and resegmentation ('extension of incomplete cell membranes')) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Required upstream for pixel-classification tasks whose probability maps feed membrane detection) — https://www.ilastik.org/download.html
- **Matlab** (Primary implementation language of the README-described MIRIAM segmentation code; Matlab and Python implementations give largely similar results) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative implementation of MIRIAM segmentation used to run the resegmentation algorithm) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Output label mask exists and matches input image dimensions
- No flagged cell retains internal membrane segments covering >10% of its area after resegmentation
- Total cell count is preserved for unflagged regions (only watershed boundaries of flagged cells are replaced)
- Split/resegmented cells are separated objects bounded by extended membrane segments meeting the cell border or other extensions
- Cross-check Matlab vs Python implementation results (article reports largely similar results, Supplemental Figure 6)

## Limitations

- Detection criterion is an area threshold (>10% internal membrane area) plus goblet-cell inclusion, so cells below the threshold with genuine internal membranes may be missed
- No standalone benchmark for the resegmentation step alone; validation is behavioral against the Supplemental Fig. 2 flowchart
- Deep learning nuclei detection was not implemented, so seed errors upstream propagate into this stage
- No software version numbers stated for the Matlab or Python implementations; example data omits image stacks (too large), requiring user regeneration
- Pipeline requires Ilastik installation for some tasks

## Evidence
