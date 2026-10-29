---
name: membrane-extension-geometry
description: Use when watershed or pixel-classification-based cell segmentation has
  split columnar epithelial cells along internal membrane segments, and cells need
  re-segmentation by extending incomplete membranes geometrically.
when_to_use_negative:
- Cells have no internal membrane segments exceeding 10% of cell area — the initial
  watershed segmentation is retained as-is
- Segmentation targets round, regular cells (e.g., isolated nuclei) where Voronoi
  or dilation from nuclei suffices; MIRIAM targets irregular columnar epithelial morphologies
- No pixel-classification probability maps or watershed label mask are available —
  this skill is a downstream resegmentation step, not a stand-alone segmenter
edam_operation: http://edamontology.org/operation_3684
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Implements the membrane-extension resegmentation algorithm ('connecting the
    dots') on watershed label masks and pixel-classification maps
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Primary implementation language of the Coffey-Lab/MIRIAM membrane-extension
    code
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation producing largely similar resegmentation
    results (Supplemental Figure 6)
- name: Ilastik
  role: Required upstream for random forest pixel classification whose probability
    maps feed the membrane extension step
  repo: https://www.ilastik.org/download.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:6128bfbbd0cdc6bbfc299449ee86cfb632d414f175b41c73850a16ab9b541451
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
  - Ilastik
  - MIRIAM
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_3684
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# membrane-extension-geometry

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM's geometric membrane-extension routine that re-segments cells whose initial watershed segmentation was split by internal (aberrant) membranes, by extending internal membrane segments from their endpoints until they meet the cell border or another extension.

## When to use

Use when watershed or pixel-classification-based cell segmentation has split columnar epithelial cells along internal membrane segments, and cells need re-segmentation by extending incomplete membranes geometrically.

## When NOT to use

- Cells have no internal membrane segments exceeding 10% of cell area — the initial watershed segmentation is retained as-is
- Segmentation targets round, regular cells (e.g., isolated nuclei) where Voronoi or dilation from nuclei suffices; MIRIAM targets irregular columnar epithelial morphologies
- No pixel-classification probability maps or watershed label mask are available — this skill is a downstream resegmentation step, not a stand-alone segmenter

## Inputs

- Random forest pixel-classification probability maps (from ilastik in the MIRIAM pipeline)
- Initial watershed label mask from upstream MIRIAM segmentation
- Multiplexed immunofluorescence image stacks (e.g., colorectal adenoma or MIBI breast carcinoma data)
- Goblet cell marker channel (Mucin 2) for candidate cell selection

## Outputs

- Corrected full-image cell label mask with re-segmented (merged) cells replacing the initial watershed labels for flagged cells
- Per-cell marker quantification over nucleus, membrane, and cytoplasm compartments downstream

## How to apply

Load the random forest pixel-classification probability maps (produced with ilastik in the MIRIAM pipeline) together with the initial watershed label mask. Flag candidate cells whose internal membrane segments occupy more than 10% of the cell area, and additionally include Mucin 2-expressing goblet cells. For each flagged cell, compute the angles of the internal membrane segments and locate their endpoints to derive extension directions. Iteratively extend each segment from its endpoints along the computed angles until it intersects either the cell border or an extension from another internal membrane, thereby 'connecting the dots' and replacing the initial watershed boundaries. Merge the re-segmented cells back into the full-image label mask; the workflow is implemented in both Matlab and Python per the Coffey-Lab MIRIAM repository.

## Related tools

- **MIRIAM** (Implements the membrane-extension resegmentation algorithm ('connecting the dots') on watershed label masks and pixel-classification maps) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Primary implementation language of the Coffey-Lab/MIRIAM membrane-extension code) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation producing largely similar resegmentation results (Supplemental Figure 6))
- **Ilastik** (Required upstream for random forest pixel classification whose probability maps feed the membrane extension step) — https://www.ilastik.org/download.html

## Evaluation signals

- Output label mask exists and matches the input image dimensions
- No flagged cell retains internal membrane segments occupying more than 10% of the cell area after extension
- Total cell count in unflagged regions is preserved relative to the initial watershed mask
- Behavioral check against the Supplemental Fig. 2 flowchart (no standalone numeric benchmark is reported for this step)
- Matlab and Python implementations should show largely similar resegmentation results (per Supplemental Figure 6)

## Limitations

- Extension terminates only at a cell border or another membrane extension, so extension direction errors from noisy angle estimates can produce incorrect splits
- Deep learning methods for nuclei detection that might improve initial seeds were not implemented
- README notes image stacks are too large to ship with the example data and must be re-generated or downloaded externally by the user

## Evidence

- [full_text] The resegmentation algorithm 'connects the dots' by locating the endpoints of internal membrane segments within a cell and extending them until they intersect either the cell border or an extension from another internal membrane: [evidence span withheld — anchor status: not_found]
