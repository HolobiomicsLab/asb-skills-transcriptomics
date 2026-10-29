---
name: backbone-point-selection-center-outward
description: Use when processing imaging data of rolled (Swiss-roll) intestinal tissues
  — e.g. CODEX-exported .csv files or 10x spatial transcriptomics images providing
  x and y coordinates — where a curved spiral backbone must be manually defined before
  unrolling cells onto a line.
when_to_use_negative:
- The input image is of flat, non-rolled tissue with no spiral topology — the center-outward
  backbone and downstream unrolling are designed for Swiss-rolled intestines
- The data source does not provide x and y coordinates (IntestLine is only stated
  to suit CODEX, 10x spatial transcriptomics, or any images providing xy-coordinates)
- You only need to re-analyze an existing mapped output without redefining the geometry
  — reuse the previously downloaded backbone file instead of re-selecting points
tools:
- name: IntestLine
  role: Shiny application whose backbone-selection panel is used to click the center-outward
    spiral backbone on the ROI of the rolled tissue image
  repo: https://github.com/SchlitzerLab/IntestLine
- name: Shiny
  role: Framework hosting the interactive IntestLine app that captures mouse clicks
    as backbone points
  repo: https://github.com/SchlitzerLab/IntestLine
- name: FASTGenomics
  role: Hosted implementation of IntestLine (https://beta.fastgenomics.org/a/intestline)
    offering the same backbone-selection workflow with limited compute (2 vCPUs, 16
    GB RAM)
  repo: https://github.com/SchlitzerLab/IntestLine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:0ac2220378a4482b24896577c63a6b72093b8ef640fdae761340a2cf602bbac3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchlitzerLab/IntestLine
  tools:
  - FASTGenomics
  - IntestLine
  - Shiny
  - dplyr
  - ggplot2
  - igraph
  - tidyr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# backbone-point-selection-center-outward

## Summary

Interactive step in IntestLine (Bioinformatics 2023, btad140) in which the user manually traces a spiral backbone along the edge of a Swiss-rolled intestinal tissue image, starting from the spiral center and moving outward, to serve as the reference curve for subsequently mapping xy-coordinate cell data onto a linear axis.

## When to use

Use when processing imaging data of rolled (Swiss-roll) intestinal tissues — e.g. CODEX-exported .csv files or 10x spatial transcriptomics images providing x and y coordinates — where a curved spiral backbone must be manually defined before unrolling cells onto a line.

## When NOT to use

- The input image is of flat, non-rolled tissue with no spiral topology — the center-outward backbone and downstream unrolling are designed for Swiss-rolled intestines
- The data source does not provide x and y coordinates (IntestLine is only stated to suit CODEX, 10x spatial transcriptomics, or any images providing xy-coordinates)
- You only need to re-analyze an existing mapped output without redefining the geometry — reuse the previously downloaded backbone file instead of re-selecting points

## Inputs

- CODEX-exported .csv file containing x and y cell coordinates
- ROI definition: chosen image center plus elliptical parameters a and b
- Optionally, a previously created backbone file for the same ROI of the same image

## Outputs

- Backbone selection file (downloadable, reusable with the same ROI of the same image)
- Backbone preview plot used to judge quality (expected 3D spiral shape)

## How to apply

After uploading a CODEX-exported .csv containing x and y coordinates, first define the Region of Interest by choosing an image center and setting the elliptical parameters a and b to highlight the area of interest. In the backbone panel, create the backbone by mouse-clicking on the edge of the structures — the click order matters and you must start from the center and go outward, since the backbone encodes the spiral winding of the Swiss-rolled tissue. Download the backbone selection for later reuse with the same ROI of the same image, or upload a previously created backbone file if one exists. Preview the selection and accept it only if the backbone shows a high-quality 3D spiral shape; a broken or non-spiral preview indicates misordered clicks and requires re-selection before proceeding to the unroll step with the angleCBS and Z-score filters.

## Related tools

- **IntestLine** (Shiny application whose backbone-selection panel is used to click the center-outward spiral backbone on the ROI of the rolled tissue image) — https://github.com/SchlitzerLab/IntestLine
- **Shiny** (Framework hosting the interactive IntestLine app that captures mouse clicks as backbone points) — https://github.com/SchlitzerLab/IntestLine
- **FASTGenomics** (Hosted implementation of IntestLine (https://beta.fastgenomics.org/a/intestline) offering the same backbone-selection workflow with limited compute (2 vCPUs, 16 GB RAM)) — https://github.com/SchlitzerLab/IntestLine

## Examples

```
docker run --name IntestLine -it -p 3838:3838 altayyuzeir/intestline bash
```

## Evaluation signals

- The backbone preview plot forms a 3D spiral shape, which the README defines as the criterion for high-quality backbone selection
- The first clicked point lies at the spiral center and subsequent points progress outward along the edge of the structures (click order preserved)
- The downloaded backbone file reloads successfully for the same ROI of the same image (uploadable in Step 1.3)
- The subsequent unroll step retains the bulk of cells, with only misprojected points removed via the angleCBS filter and noisy points/contaminants via Z-score filtering, and the count/regions of lost cells are examinable
- ROI was defined before backbone selection (center plus elliptical parameters a and b), since the backbone is only reusable with the same ROI

## Limitations

- Backbone selection is manual and click-order-sensitive — starting anywhere but the spiral center invalidates the geometry
- A backbone file is only reusable with the same ROI of the same image, not across images or changed ROIs
- The Docker deployment of IntestLine might not work on Mac with the M1-chip, and the FASTGenomics server is compute-limited (2 vCPUs @2.1GHz, 16 GB RAM; ~26 s for the 10k-cell demo dataset, much longer for 150k cells)

## Evidence

- [methods] Create a backbone by pressing with the mouse on the edge of the structures.: "Create a backbone by pressing with the mouse on the edge of the structures."
- [methods] Select Region of Interest (ROI) by first choosing a center for the image.: "Select Region of Interest (ROI) by first choosing a center for the image."
- [readme] You must start from the center and go outward !: "You must start from the center and go outward !"
- [readme] You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape.: "You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape."
- [readme] You can download the backbone selection for later use with the same ROI of the same image.: "You can download the backbone selection for later use with the same ROI of the same image."
