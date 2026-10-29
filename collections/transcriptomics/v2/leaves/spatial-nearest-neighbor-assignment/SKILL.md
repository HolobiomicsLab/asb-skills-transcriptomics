---
name: spatial-nearest-neighbor-assignment
description: Use when cells with x/y coordinates from a rolled (Swiss-rolled) intestinal
  tissue image must be mapped onto an ordered, user-selected backbone so that along-line
  arc position and radial offset (wall thickness) are retained.
when_to_use_negative:
- Input images provide no x/y per-cell coordinates (e.g. only bulk or spot-level data)
  — IntestLine processes 'basically any images providing xy-coordinates'
- The tissue image is not a rolled/Swiss-rolled intestinal preparation, so no spiral
  backbone from center outward can be defined
- Data are already an unrolled linear map or a feature/abundance table with no spatial
  coordinates
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: IntestLine
  role: Shiny-based application implementing the directionally biased nearest-neighbor
    assignment of cells to base-layer points during unrolling of the rolled intestinal
    tissue image
  repo: https://github.com/SchlitzerLab/IntestLine
- name: igraph
  role: R library used to build the nearest-neighbor matching between cells and outward-direction
    candidate backbone points
- name: tidyr
  role: R library used to shape the per-cell x/y coordinate and backbone point data
    frames for the matching
- name: dplyr
  role: R library used to load and manipulate the CODEX-exported CSV and ordered backbone
    point data frames
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:d02cc5f8840e18609c3f2d2d2419d4c385fca71c5c5bde4683c3032f419ab540
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchlitzerLab/IntestLine
  tools:
  - IntestLine
  - dplyr
  - igraph
  - tidyr
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spatial-nearest-neighbor-assignment

## Summary

Assigns individual cells from a CODEX-exported x/y coordinate table to user-selected base-layer (backbone) points on a Swiss-rolled intestinal tissue image using a directionally biased nearest-neighbor matching, preserving intestinal-wall thickness and each cell's serosa-luminal position in the linear map.

## When to use

Use when cells with x/y coordinates from a rolled (Swiss-rolled) intestinal tissue image must be mapped onto an ordered, user-selected backbone so that along-line arc position and radial offset (wall thickness) are retained.

## When NOT to use

- Input images provide no x/y per-cell coordinates (e.g. only bulk or spot-level data) — IntestLine processes 'basically any images providing xy-coordinates'
- The tissue image is not a rolled/Swiss-rolled intestinal preparation, so no spiral backbone from center outward can be defined
- Data are already an unrolled linear map or a feature/abundance table with no spatial coordinates

## Inputs

- CODEX-exported .csv file containing per-cell x and y coordinates
- Ordered backbone point coordinates (user-selected by mouse clicks from the spiral center outward, or a previously created backbone file)

## Outputs

- Per-cell assignment of each input cell to its outward-facing nearest base-layer (backbone) point
- Along-line (arc) position coordinate for each cell in the linear map
- Residual radial offset (thickness) coordinate for each cell, retaining serosa-luminal position within the wall
- Filtered cell table after angleCBS misprojection removal, with counts of lost cells and their regions

## How to apply

Load the CODEX-exported CSV of per-cell x/y coordinates together with the ordered backbone point coordinates — selected by clicking on the edge of the structures starting from the spiral center and proceeding outward — into data frames using dplyr and tidyr. Build a nearest-neighbor matching with igraph, restricting candidate base-layer points to the outward direction so each cell is assigned to the backbone point lying beneath it in the spiral; this directional bias is what preserves wall thickness and serosa-luminal position in the linear map. For every cell, record its arc position along the linear structure and its residual radial offset from the backbone. Remove misprojected assignments with the angleCBS filter (optionally applying Z-score filtering for noisy points or contaminants), and track how many cells are lost and from which regions.

## Related tools

- **IntestLine** (Shiny-based application implementing the directionally biased nearest-neighbor assignment of cells to base-layer points during unrolling of the rolled intestinal tissue image) — https://github.com/SchlitzerLab/IntestLine
- **igraph** (R library used to build the nearest-neighbor matching between cells and outward-direction candidate backbone points)
- **tidyr** (R library used to shape the per-cell x/y coordinate and backbone point data frames for the matching)
- **dplyr** (R library used to load and manipulate the CODEX-exported CSV and ordered backbone point data frames)

## Examples

```
docker run --name IntestLine -it -p 3838:3838 altayyuzeir/intestline bash && /usr/bin/run-shiny.sh shiny
```

## Evaluation signals

- Backbone selection preview shows a high-quality 3D spiral shape, and selection started from the center and went outward
- Output row count equals the number of input cells minus the misprojected points removed by the angleCBS filter
- Output contains fields for both along-line (arc position) and thickness (radial offset) coordinates
- Counts of lost cells and the regions they came from are tracked after angleCBS and Z-score filtering
- Mapped data resolves cells along both proximal–distal and serosa-luminal axes when overlaid on the linear structure

## Limitations

- Backbone selection must start from the center and go outward; selecting in the wrong direction breaks the outward-directional nearest-neighbor assignment
- angleCBS filtering discards misprojected points, so cells are lost and the loss must be examined by region
- Z-score filtering may remove noisy points or contaminants along with genuine cells
- The Docker image might not work on Mac with the M1 chip, and the FASTGenomics server (2 vCPUs @2.1GHz, 16 GB RAM) is slow for large datasets, e.g. minutes for 150k cells

## Evidence

- [readme] Backbone must be traced from the spiral center outward so the nearest-neighbor matching can restrict candidates to the outward direction.: "2.2 **You must start from the center and go outward !**"
- [methods] Misprojected assignments are removed via the angleCBS filter, which requires tracking lost cells.: "Misprojected points can be filtered out by manipulating the angleCBS filter."
- [readme] Backbone quality is validated by previewing for a 3D spiral shape.: "You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape."
- [intro] Scope: the assignment applies to any imaging modality providing per-cell x/y coordinates such as CODEX or 10x spatial transcriptomics.: "It is suitable to process CODEX images or 10x spatial transcriptomics images, basically any images providing xy-coordinates"
- [methods] Required R libraries for running the matching and filtering pipeline locally.: "Following libraries are needed to run the shiny app: igraph,tidyr,dplyr,ggplot2,plotly,ggtext,data.table,readr."
