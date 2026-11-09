---
name: cell-radial-offset-preservation
description: Use when mapping per-cell x/y coordinates from a CODEX (or 10x spatial
  transcriptomics) image of Swiss-rolled intestinal tissue onto a linear backbone,
  and you must retain radial (thickness) position, not just along-line position.
when_to_use_negative:
- The image data provide no xy-coordinates — IntestLine is suitable for CODEX, 10x
  spatial transcriptomics, or any images providing xy-coordinates, but nothing else
- The tissue is already unrolled/linear so there is no spiral backbone and no radial
  offset to preserve
- The backbone selection was not drawn from the center outward (e.g. an arbitrary
  or reversed point order), which would break the directional bias of the nearest-neighbor
  assignment
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: igraph
  role: Builds the directionally biased nearest-neighbor matching between cells and
    outward-restricted base-layer points
- name: dplyr
  role: Loads and manipulates the per-cell coordinate and backbone point data frames
- name: tidyr
  role: Reshapes the CODEX coordinate and backbone data frames for the matching step
- name: IntestLine
  role: Shiny application hosting the unroll step, backbone selection, and angleCBS/Z-score
    filters that wrap this assignment
  repo: https://github.com/SchlitzerLab/IntestLine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:c126f757c16921a41f7ded67f5d44373e1ada6b8d462dac3fbb75bd94b91fdae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchlitzerLab/IntestLine
  tools:
  - IntestLine
  - dplyr
  - igraph
  - tidyr
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cell-radial-offset-preservation

## Summary

Assigns each cell from a Swiss-rolled intestinal tissue image to a base-layer (backbone) point using a directionally biased nearest-neighbor match restricted to outward base-layer points, then records each cell's arc position along the line and its residual radial offset. This preserves intestinal-wall thickness and each cell's serosa-luminal position within the wall in the linear map.

## When to use

Use when mapping per-cell x/y coordinates from a CODEX (or 10x spatial transcriptomics) image of Swiss-rolled intestinal tissue onto a linear backbone, and you must retain radial (thickness) position, not just along-line position.

## When NOT to use

- The image data provide no xy-coordinates — IntestLine is suitable for CODEX, 10x spatial transcriptomics, or any images providing xy-coordinates, but nothing else
- The tissue is already unrolled/linear so there is no spiral backbone and no radial offset to preserve
- The backbone selection was not drawn from the center outward (e.g. an arbitrary or reversed point order), which would break the directional bias of the nearest-neighbor assignment

## Inputs

- CODEX-exported .csv file containing per-cell x and y coordinates
- Ordered backbone point coordinates selected from the spiral center outward (optionally a previously created backbone file for the same ROI of the same image)

## Outputs

- Per-cell table with along-line (arc) position for each cell
- Per-cell residual radial offset (thickness/serosa-luminal coordinate) from the backbone
- Filtered output with misprojected points removed by the angleCBS filter, plus counts of lost cells by region

## How to apply

Load the CODEX-exported .csv of per-cell x/y coordinates together with the ordered backbone point coordinates — the backbone must be selected starting from the spiral center and proceeding outward — into data frames using dplyr and tidyr. Build the nearest-neighbor matching with igraph, restricting candidate base-layer points to the outward direction so each cell is assigned to the backbone point lying beneath it in the spiral. For every cell, record its arc position along the linear structure and its residual radial offset from the backbone, so wall thickness and serosa-luminal position are retained in the linear map. Remove misprojected assignments by manipulating the angleCBS filter, and examine how many cells and from which regions are lost.

## Related tools

- **igraph** (Builds the directionally biased nearest-neighbor matching between cells and outward-restricted base-layer points)
- **dplyr** (Loads and manipulates the per-cell coordinate and backbone point data frames)
- **tidyr** (Reshapes the CODEX coordinate and backbone data frames for the matching step)
- **IntestLine** (Shiny application hosting the unroll step, backbone selection, and angleCBS/Z-score filters that wrap this assignment) — https://github.com/SchlitzerLab/IntestLine

## Examples

```
/usr/bin/run-shiny.sh shiny
```

## Evaluation signals

- Output row count equals the number of input cells minus the cells removed by the angleCBS filter for misprojected points
- Output fields for along-line (arc) position and thickness/radial-offset coordinates are present for every retained cell
- Backbone preview shows the high-quality 3D spiral shape described for a valid selection
- Intestinal-wall thickness is preserved: each cell's serosa-luminal position within the wall is retained in the linear map
- The number of lost cells and the regions they came from after angleCBS and Z-score filtering are inspected and documented

## Limitations

- Backbone must be drawn starting from the center and going outward, otherwise the directional assignment is invalid
- Misprojected points must be filtered with angleCBS (and noisy points/contaminants with Z-score filtering), so some cells are necessarily lost
- The Docker image may not work on Macs with the M1 chip
- The FASTGenomics deployment is resource-limited (2 vCPUs @2.1GHz, 16 GB RAM; 26s for a 10k-cell demo, much longer for ~150k cells) and was only available until mid-2024

## Evidence

- [readme] Upload CODEX-exported .csv file, containing x and y coordinates.: "Upload CODEX-exported .csv file, containing x and y coordinates."
- [readme] You must start from the center and go outward !: "2.2 **You must start from the center and go outward !**"
- [methods] Following libraries are needed to run the shiny app: igraph,tidyr,dplyr,ggplot2,plotly,ggtext,data.table,readr.: "Following libraries are needed to run the shiny app: igraph,tidyr,dplyr,ggplot2,plotly,ggtext,data.table,readr."
- [methods] Misprojected points can be filtered out by manipulating the angleCBS filter.: "Misprojected points can be filtered out by manipulating the angleCBS filter."
- [methods] High quality backbone has a 3D spiral shape.: "You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape."
