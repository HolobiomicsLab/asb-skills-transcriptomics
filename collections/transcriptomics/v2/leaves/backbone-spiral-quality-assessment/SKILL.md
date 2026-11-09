---
name: backbone-spiral-quality-assessment
description: Use when you are drawing or reusing a backbone over a rolled-intestine
  ROI (CODEX or 10x xy-coordinate CSV) and must confirm the selection traces a high-quality
  3D spiral, starting from the spiral center and going outward, before running the
  Unroll step.
when_to_use_negative:
- The image is not a Swiss-rolled intestinal tissue with a spiral structure — there
  is no center-outward backbone to trace and the linear mapping assumptions do not
  hold
- The input provides no per-point xy-coordinates (IntestLine requires 'basically any
  images providing xy-coordinates')
- You only need marker-expression visualization along an already validated, downloaded
  backbone — reuse the existing backbone file instead of redrawing and re-assessing
  it
edam_operation: http://edamontology.org/operation_0337
tools:
- name: IntestLine
  role: Shiny application whose backbone panel provides mouse-click backbone creation,
    the quality preview, and backbone download for the spiral ROI
  repo: https://github.com/SchlitzerLab/IntestLine
- name: Shiny
  role: R web framework hosting the interactive backbone-selection and preview panel
    used for this quality assessment
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:3a8af3ec34296cee0a513161e1563d357238898b19ced884949cfecee71d7a39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchlitzerLab/IntestLine
  tools:
  - IntestLine
  - Shiny
  - dplyr
  - ggplot2
  - igraph
  - plotly
  - readr
  - tidyr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# backbone-spiral-quality-assessment

## Summary

Quality-control step in the IntestLine workflow that previews whether the user-selected backbone points for a Swiss-rolled intestinal ROI form a high-quality 3D spiral before unrolling. Because cells are assigned to backbone points by a directionally biased outward nearest-neighbor search, a wrongly ordered or deformed backbone corrupts the proximal-distal and serosa-luminal mapping.

## When to use

Use when you are drawing or reusing a backbone over a rolled-intestine ROI (CODEX or 10x xy-coordinate CSV) and must confirm the selection traces a high-quality 3D spiral, starting from the spiral center and going outward, before running the Unroll step.

## When NOT to use

- The image is not a Swiss-rolled intestinal tissue with a spiral structure — there is no center-outward backbone to trace and the linear mapping assumptions do not hold
- The input provides no per-point xy-coordinates (IntestLine requires 'basically any images providing xy-coordinates')
- You only need marker-expression visualization along an already validated, downloaded backbone — reuse the existing backbone file instead of redrawing and re-assessing it

## Inputs

- CODEX-exported .csv file containing per-cell x and y coordinates
- User-defined Region of Interest (center point plus parameters a and b defining the elliptical area of interest)
- Backbone point coordinates entered by mouse clicks on the edge of the structures (or a previously created backbone file)

## Outputs

- Backbone preview plot used to judge quality (a high-quality backbone displays a 3D spiral shape)
- Downloaded backbone selection file for reuse with the same ROI of the same image
- Ordered center-outward backbone point list that feeds the Unroll step

## How to apply

In the IntestLine Shiny app (blue panel, Step 2), create the backbone by clicking with the mouse on the edge of the structures, strictly starting from the spiral center and proceeding outward — the ordering is what the outward-restricted nearest-neighbor assignment relies on. Preview the backbone selection quality in the app: a high-quality backbone has a 3D spiral shape; if the preview is degenerate or out of order, redraw the selection rather than proceeding to Unroll. Download the accepted backbone file so it can be uploaded again for later runs on the same ROI of the same image (or supply a previously created backbone file at upload time). Only after the spiral-shape check passes should you move to Step 3 (Unroll with angleCBS and Z-score filters).

## Related tools

- **IntestLine** (Shiny application whose backbone panel provides mouse-click backbone creation, the quality preview, and backbone download for the spiral ROI) — https://github.com/SchlitzerLab/IntestLine
- **Shiny** (R web framework hosting the interactive backbone-selection and preview panel used for this quality assessment)

## Examples

```
docker run --name IntestLine -it -p 3838:3838 altayyuzeir/intestline bash && /usr/bin/run-shiny.sh shiny
```

## Evaluation signals

- The backbone preview shows a 3D spiral shape — the article's explicit criterion for a high-quality backbone selection
- The backbone point sequence starts at the spiral center and proceeds outward; any inward jumps or disorder should trigger redrawing
- The backbone file downloads successfully and can be re-uploaded for the same ROI of the same image without changing the ROI definition
- Downstream check: after Unroll, verify row count equals input cells minus misprojected points removed by the angleCBS filter, and that along-line and thickness coordinate fields are present
- Ordering invariant: Spearman rank correlation between cell order in the mapped linear coordinate and original order along the spiral backbone approaches 1 (monotone preservation)

## Limitations

- Backbone selection is manual (mouse clicks on structure edges), so quality depends on the user; the article prescribes the 3D spiral shape as the visual acceptance criterion but no automated score
- A downloaded backbone is only valid for the same ROI of the same image; changing ROI or parameters a/b invalidates reuse
- The FASTGenomics deployment is constrained (2 vCPUs @2.1 GHz, 16 GB RAM) and large datasets unroll slowly (26 s for a 10k-cell demo; much longer for 150k cells)
- The Docker image may not work on Macs with the M1 chip, and FASTGenomics availability was noted only until mid-2024

## Evidence

- [methods] Create a backbone by pressing with the mouse on the edge of the structures.: "Create a backbone by pressing with the mouse on the edge of the structures."
- [methods] You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape.: "You can preview the quality of the backbone selection. High quality backbone has a 3D spiral shape."
- [readme] You must start from the center and go outward !: "You must start from the center and go outward !"
- [readme] You can download the backbone selection for later use with the same ROI of the same image.: "You can download the backbone selection for later use with the same ROI of the same image."
- [readme] If available, you can upload previously created backbone file.: "If available, you can upload previously created backbone file."
