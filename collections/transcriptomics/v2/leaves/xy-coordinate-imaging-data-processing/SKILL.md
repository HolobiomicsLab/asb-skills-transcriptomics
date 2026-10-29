---
name: xy-coordinate-imaging-data-processing
description: Use when the input is a CODEX-exported .csv (or similar) containing x
  and y coordinates of cells from rolled intestinal tissue and the goal is to map
  the spiral onto a linear proximal–distal / serosa–luminal axis.
when_to_use_negative:
- The input image format does not provide x and y cell coordinates (IntestLine is
  designed for CODEX, 10x spatial transcriptomics, or any xy-coordinate imagery).
- The tissue was not prepared by Swiss-rolling, so there is no spiral to unroll onto
  a line.
- A non-interactive, fully automated batch pipeline is required — backbone selection
  is manual, done by pressing with the mouse in the Shiny app.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: IntestLine
  role: Shiny-based application that performs the ROI selection, backbone clicking,
    unrolling, and marker overlay on xy-coordinate imaging data
  repo: https://github.com/SchlitzerLab/IntestLine
- name: Shiny
  role: Interactive web framework hosting the IntestLine app panels (data upload,
    backbone selection, unroll, overlay)
  repo: https://github.com/SchlitzerLab/IntestLine
- name: Docker (altayyuzeir/intestline image)
  role: Container used to run the IntestLine Shiny app locally via /usr/bin/run-shiny.sh
    shiny exposed on port 3838
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:d4dd16bea57b5f2c5cdeb37f74be51df967cfdaef4b7f14c2394f796694beb8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchlitzerLab/IntestLine
  tools:
  - Docker
  - Docker (altayyuzeir/intestline image)
  - IntestLine
  - Shiny
  - data.table
  - dplyr
  - ggplot2
  - ggtext
  - igraph
  - plotly
  - readr
  - tidyr
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# xy-coordinate-imaging-data-processing

## Summary

Process xy-coordinate imaging data from Swiss-rolled intestinal tissue (e.g., CODEX or 10x spatial transcriptomics exports) by interactively selecting a region of interest and backbone in the IntestLine Shiny app, then unrolling the spiral onto a line with angleCBS and Z-score filtering.

## When to use

Use when the input is a CODEX-exported .csv (or similar) containing x and y coordinates of cells from rolled intestinal tissue and the goal is to map the spiral onto a linear proximal–distal / serosa–luminal axis.

## When NOT to use

- The input image format does not provide x and y cell coordinates (IntestLine is designed for CODEX, 10x spatial transcriptomics, or any xy-coordinate imagery).
- The tissue was not prepared by Swiss-rolling, so there is no spiral to unroll onto a line.
- A non-interactive, fully automated batch pipeline is required — backbone selection is manual, done by pressing with the mouse in the Shiny app.

## Inputs

- CODEX-exported .csv file containing x and y coordinates of cells
- Optionally, a previously created backbone file for the same ROI of the same image
- 10x spatial transcriptomics images or any imagery providing xy-coordinates

## Outputs

- Unrolled (stretched) linear mapping of the rolled intestinal tissue
- Previewable 3D spiral backbone selection (downloadable for reuse)
- Marker expression overlaid along the length and thickness of the linear structure
- Counts of cells lost to angleCBS and Z-score filtering, with their regions

## How to apply

Upload the CODEX-exported .csv containing x and y coordinates, then select the Region of Interest by choosing an image center and setting parameters a and b to highlight the elliptical area of interest. Create the backbone by clicking on the edge of the structures with the mouse, strictly starting from the center and going outward, and preview it — a high-quality backbone has a 3D spiral shape. Run the Unroll step (bottom green panel), tuning the angleCBS filter to remove misprojected points and the Z-score filtering to eliminate noisy points or contaminants; in both cases inspect how many cells and from which regions are lost. Overlay expressed markers on the stretched image to examine expression along the length and thickness of the organ. Expect roughly 26 s to unroll a 10k-cell demo dataset on a 2 vCPU @2.1 GHz, 16 GB RAM server, and on the order of minutes for ~150k cells.

## Related tools

- **IntestLine** (Shiny-based application that performs the ROI selection, backbone clicking, unrolling, and marker overlay on xy-coordinate imaging data) — https://github.com/SchlitzerLab/IntestLine
- **Shiny** (Interactive web framework hosting the IntestLine app panels (data upload, backbone selection, unroll, overlay)) — https://github.com/SchlitzerLab/IntestLine
- **Docker (altayyuzeir/intestline image)** (Container used to run the IntestLine Shiny app locally via /usr/bin/run-shiny.sh shiny exposed on port 3838)

## Examples

```
docker run --name IntestLine -it -p 3838:3838 altayyuzeir/intestline bash
```

## Evaluation signals

- The backbone preview shows a 3D spiral shape, the README's criterion for a high-quality backbone selection.
- The unroll step produces a stretched linear image on which markers can be overlaid along length and thickness.
- The number of cells (and regions) lost to the angleCBS filter and the Z-score filtering can be examined after unrolling.
- Runtime for a 10k-cell demo dataset is on the order of ~26 s on a 2 vCPU @2.1 GHz, 16 GB RAM server; a 150k-cell dataset takes minutes.
- The output preserves both proximal–distal and serosa–luminal axes, allowing location-specific cell types and markers to be observed.

## Limitations

- The FASTGenomics server configuration is very limited (2 vCPUs @2.1GHz, 16 GB RAM), so unrolling a large dataset (e.g., 150k cells) takes minutes.
- The Docker image might not work on Mac with the M1 chip.
- The FASTGenomics hosted option is only available until mid-2024, per the JiangyanYu/IntestLine README.
- No software or library versions are stated for the required R libraries (igraph, tidyr, dplyr, ggplot2, plotly, ggtext, data.table, readr).

## Evidence

- [abstract] Here, we present IntestLine, a Shiny-based open-source application for processing imaging data of (rolled) intestinal tissues: "Here, we present IntestLine, a Shiny-based open-source application for processing imaging data of (rolled) intestinal tissues"
- [readme] Upload CODEX-exported .csv file, containing x and y coordinates.: "Upload CODEX-exported .csv file, containing x and y coordinates."
- [readme] Create a backbone by pressing with the mouse on the edge of the structures.: "Create a backbone by pressing with the mouse on the edge of the structures."
- [readme] Misprojected points can be filtered out by manipulating the angleCBS filter.: "Misprojected points can be filtered out by manipulating the angleCBS filter."
- [readme] For example, 26s for the demo dataset containing 10k cells: "For example, 26s for the demo dataset containing 10k cells"
- [methods] Following libraries are needed to run the shiny app: igraph,tidyr,dplyr,ggplot2,plotly,ggtext,data.table,readr.: "Following libraries are needed to run the shiny app: igraph,tidyr,dplyr,ggplot2,plotly,ggtext,data.table,readr."
