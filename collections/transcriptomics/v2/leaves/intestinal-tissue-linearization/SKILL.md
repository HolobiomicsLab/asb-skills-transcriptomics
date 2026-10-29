---
name: intestinal-tissue-linearization
description: Use when a CODEX-exported .csv of per-cell x/y coordinates from rolled
  (spiral/Swiss-roll) intestinal tissue needs to be unrolled onto a line so cell types
  and markers can be assessed along proximal–distal and serosa-luminal axes.
when_to_use_negative:
- The input image is an unrolled/flat tissue section rather than a Swiss-rolled spiral,
  so there is no spiral backbone to trace from center outward
- The data provide no x/y coordinates (e.g. an expression matrix without spatial positions)
  — IntestLine requires images providing xy-coordinates
- A large dataset must be processed quickly on the FASTGenomics deployment, whose
  server is limited (2 vCPUs @2.1GHz, 16 GB RAM) — unrolling 150k cells takes on the
  order of minutes
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: IntestLine
  role: Shiny application that performs ROI selection, backbone tracing, nearest-neighbor
    assignment of cells to backbone points, filtering, and marker overlay
  repo: https://github.com/SchlitzerLab/IntestLine
- name: igraph
  role: R library used to build the directionally biased nearest-neighbor matching
    that assigns each cell to its outward base-layer point
- name: tidyr
  role: Required library for reshaping the CODEX-exported per-cell coordinate table
    and backbone points during unrolling
- name: dplyr
  role: Required library for data-frame manipulation of cell coordinates and backbone
    assignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:586d7727b1d72f184a5ee99b1cc43e793bc0cff74daf143f50587008663260c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchlitzerLab/IntestLine
  tools:
  - IntestLine
  - dplyr
  - igraph
  - tidyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# intestinal-tissue-linearization

## Summary

Maps per-cell x/y coordinates from Swiss-rolled intestinal tissue images (e.g. CODEX or 10x spatial transcriptomics exports) onto a one-dimensional line, preserving proximal–distal position along the organ and serosa-luminal position within the intestinal wall. Implemented as the IntestLine Shiny application.

## When to use

Use when a CODEX-exported .csv of per-cell x/y coordinates from rolled (spiral/Swiss-roll) intestinal tissue needs to be unrolled onto a line so cell types and markers can be assessed along proximal–distal and serosa-luminal axes.

## When NOT to use

- The input image is an unrolled/flat tissue section rather than a Swiss-rolled spiral, so there is no spiral backbone to trace from center outward
- The data provide no x/y coordinates (e.g. an expression matrix without spatial positions) — IntestLine requires images providing xy-coordinates
- A large dataset must be processed quickly on the FASTGenomics deployment, whose server is limited (2 vCPUs @2.1GHz, 16 GB RAM) — unrolling 150k cells takes on the order of minutes

## Inputs

- CODEX-exported .csv file containing per-cell x and y coordinates
- Optionally, a previously created backbone file for the same ROI of the same image

## Outputs

- Per-cell linear map with an arc/along-line position and a residual radial offset (thickness coordinate) for each cell
- Downloadable backbone point selection (ordered from spiral center outward)
- Filtered cell table after angleCBS and Z-score filtering, with counts of lost cells per region
- Overlay of expressed markers along the length and thickness of the organ on the linear structure

## How to apply

Upload the CODEX-exported .csv containing per-cell x and y coordinates into IntestLine, then select the Region of Interest by choosing an image center and setting parameters a and b that highlight the elliptical area of interest. Create a backbone by clicking on the edge of the structures, starting from the spiral center and proceeding outward — ordering is essential because candidate base-layer points are restricted to the outward direction. Preview the backbone: a high-quality selection has a 3D spiral shape; download it for reuse with the same ROI. During unrolling, each cell is assigned to a backbone point by a directionally biased nearest-neighbor analysis (built with igraph), recording each cell's arc position along the linear structure and its residual radial offset so wall thickness and serosa-luminal position are retained. Finally, remove misprojected assignments with the angleCBS filter and noisy points or contaminants with Z-score filtering, examining in both cases how many cells are lost and from which regions.

## Related tools

- **IntestLine** (Shiny application that performs ROI selection, backbone tracing, nearest-neighbor assignment of cells to backbone points, filtering, and marker overlay) — https://github.com/SchlitzerLab/IntestLine
- **igraph** (R library used to build the directionally biased nearest-neighbor matching that assigns each cell to its outward base-layer point)
- **tidyr** (Required library for reshaping the CODEX-exported per-cell coordinate table and backbone points during unrolling)
- **dplyr** (Required library for data-frame manipulation of cell coordinates and backbone assignments)

## Examples

```
docker pull altayyuzeir/intestline && docker run --name IntestLine -it -p 3838:3838 altayyuzeir/intestline bash
```

## Evaluation signals

- Backbone preview shows a 3D spiral shape, confirming a high-quality selection
- Output row count equals the number of input cells minus the filtered misprojected points
- Every output cell has both an along-line (arc) coordinate and a thickness (radial offset) coordinate
- Number of cells lost to angleCBS and Z-score filtering is recorded along with the regions they came from
- Backbone point ordering starts at the spiral center and proceeds outward

## Limitations

- The Docker image might not work on a Mac with the M1 chip
- The FASTGenomics server is limited to 2 vCPUs @2.1GHz and 16 GB RAM; a 10k-cell demo dataset took 26 s to unroll, while a 150k-cell dataset takes on the order of minutes
- Misprojected points must be removed by manually manipulating the angleCBS filter, and noisy points or contaminants via Z-score filtering — both cause cell loss that must be inspected by region

## Evidence

- [abstract] Here, we present IntestLine, a Shiny-based open-source application for processing imaging data of (rolled) intestinal tissues and subsequent mapping onto a line.: "Here, we present IntestLine, a Shiny-based open-source application for processing imaging data of (rolled) intestinal tissues and subsequent mapping onto a line."
- [methods] Following libraries are needed to run the shiny app: igraph,tidyr,dplyr,ggplot2,plotly,ggtext,data.table,readr.: "Following libraries are needed to run the shiny app: igraph,tidyr,dplyr,ggplot2,plotly,ggtext,data.table,readr."
- [readme] Misprojected points can be filtered out by manipulating the angleCBS filter.: "Misprojected points can be filtered out by manipulating the angleCBS filter."
- [readme] You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape.: "You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape."
