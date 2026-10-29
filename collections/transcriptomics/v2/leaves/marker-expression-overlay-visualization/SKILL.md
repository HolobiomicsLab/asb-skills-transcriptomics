---
name: marker-expression-overlay-visualization
description: Use when a spatial single-cell dataset (e.g., CODEX-exported .csv with
  x,y coordinates) has already been unrolled onto a linear structure and you need
  to visualize marker expression along organ length and wall thickness.
when_to_use_negative:
- Input is raw rolled intestinal tissue imaging data that has not yet been uploaded,
  ROI-selected, backbone-traced, and unrolled in IntestLine — the overlay requires
  a completed linear mapping.
- Data lacks xy-coordinates — IntestLine is suitable only for images providing xy-coordinates
  (e.g. CODEX or 10x spatial transcriptomics), not other imaging modalities.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: IntestLine
  role: Shiny-based application that performs the marker overlay on the stretched
    (unrolled) image of rolled intestinal tissue
  repo: https://github.com/SchlitzerLab/IntestLine
- name: Shiny
  role: Framework in which the IntestLine app, including the overlay panel, runs locally
    or in FASTGenomics
- name: ggplot2
  role: Required R library for rendering the overlay visualizations in the IntestLine
    app
- name: plotly
  role: Required R library for interactive plotting in the IntestLine app
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:4a9b7ddc4e99382edd88210f61acfa729ce731359467e2a04e32b4de256ea713
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
  - tidyr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# marker-expression-overlay-visualization

## Summary

A visualization step in the IntestLine Shiny app that overlays marker expression onto the linearly unrolled intestinal tissue, enabling assessment of location-specific cell types and markers along the proximal–distal and serosa-luminal axes.

## When to use

Use when a spatial single-cell dataset (e.g., CODEX-exported .csv with x,y coordinates) has already been unrolled onto a linear structure and you need to visualize marker expression along organ length and wall thickness.

## When NOT to use

- Input is raw rolled intestinal tissue imaging data that has not yet been uploaded, ROI-selected, backbone-traced, and unrolled in IntestLine — the overlay requires a completed linear mapping.
- Data lacks xy-coordinates — IntestLine is suitable only for images providing xy-coordinates (e.g. CODEX or 10x spatial transcriptomics), not other imaging modalities.

## Inputs

- CODEX-exported .csv file containing x and y coordinates (or any spatial image providing xy-coordinates, e.g. 10x spatial transcriptomics images)
- Filtered, mapped linear-coordinate cell table produced by the IntestLine unroll step
- Backbone selection file (previously created for the same ROI of the same image)

## Outputs

- Unrolled marker-overlay visualization (e.g. unrolled_overlay.png) showing expression along organ length and thickness
- Linear-coordinate cell table overlaid with marker expression parameters

## How to apply

After completing the unroll step in the IntestLine Shiny app (with angleCBS filter tuned to remove misprojected points and Z-score filtering applied to eliminate noisy points or contaminants), use the overlay panel (Step 4, bottom/green panel in the FASTGenomics layout) to overlay expressed markers over the stretched image. Choose the markers of interest and inspect their signal along the linearized proximal–distal axis and the serosa-luminal thickness axis of the organ. Examine the loss report from the preceding filters so you know how many cells and from which regions were dropped before interpreting marker patterns. Save the resulting unrolled marker-overlay visualization (e.g., unrolled_overlay.png) together with the filtered, mapped linear-coordinate cell table.

## Related tools

- **IntestLine** (Shiny-based application that performs the marker overlay on the stretched (unrolled) image of rolled intestinal tissue) — https://github.com/SchlitzerLab/IntestLine
- **Shiny** (Framework in which the IntestLine app, including the overlay panel, runs locally or in FASTGenomics)
- **ggplot2** (Required R library for rendering the overlay visualizations in the IntestLine app)
- **plotly** (Required R library for interactive plotting in the IntestLine app)

## Examples

```
/usr/bin/run-shiny.sh shiny  # then in the browser, use the IntestLine overlay panel on the unrolled image
```

## Evaluation signals

- The backbone preview shows a high-quality 3D spiral shape before unrolling, so the overlaid linear coordinates are trustworthy.
- The overlay shows marker expression patterns that vary along the proximal–distal and serosa-luminal axes, consistent with location-specific cell types.
- The number and regions of cells lost via angleCBS and Z-score filtering have been examined and are acceptable before interpretation.
- Output artifacts include the filtered, mapped linear-coordinate cell table (e.g. intestline_mapped_output.csv) and the unrolled overlay image (e.g. unrolled_overlay.png).

## Limitations

- The FASTGenomics server configuration is very limited (2 vCPUs @2.1GHz, 16 GB RAM), so unrolling large datasets is slow — 26s for the 10k-cell demo but much longer for ~150k cells.
- The docker image might not work on Mac with the M1-chip.
- Cells misprojected or flagged as contaminants are removed by the angleCBS and Z-score filters, so overlay patterns are based on a filtered subset; examine how many cells and from which regions are lost.

## Evidence

- [readme] overlay markers on unrolled image: "You can overlay expressed markers over the stretched image."
- [readme] marker expression along length and thickness: "This can be used to examine marker expression along the length and thickness of the organ."
- [abstract] filtered visualization findings: "The visualization of the mapped data facilitates the assessment of the whole intestine in both proximal–distal and serosa-luminal axis"
- [readme] filtering before overlay: "Misprojected points can be filtered out by manipulating the angleCBS filter."
