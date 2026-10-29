---
name: signature-autocorrelation-report-rendering
description: Use when a completed VISION/PhyloVision analysis object is available
  and the goal is to deliver or share results as an interactive web report, locally
  or on a remote server.
when_to_use_negative:
- The analysis object has not yet been processed (signature autocorrelations, Hotspot
  modules, and projections not computed) — build the analysis first, then render
- The goal is static publication figures rather than interactive exploration; the
  report is designed for interactive, shareable exploration
- Only lineage visualization without expression integration is needed — tools like
  iTOL or CeLaVi handle tree-only viewing
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: PhyloVision
  role: Integrated pipeline whose viewResults step builds and serves the interactive
    web report of lineage and expression analysis
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Provides the R package and front-end on which the interactive web-based report
    is built; supplies viewResults, load_de_cache, and load_selections
  repo: https://github.com/YosefLab/VISION
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:cbf24c8df2123b2b9a3c7931a59ae3a2b2b510e532bcf37c6440c08b8c3ef4a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - PhyloVision
  - R
  - VISION
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# signature-autocorrelation-report-rendering

## Summary

Generate and host the interactive, feature-rich web-based report that PhyloVision produces from an analyzed single-cell lineage-tracing dataset, exposing signature/Hotspot module tables (FDR < 0.05), phylogeny displays (PhyloPlot.js), and UMAP/tSNE overlays built on the VISION front-end.

## When to use

Use when a completed VISION/PhyloVision analysis object is available and the goal is to deliver or share results as an interactive web report, locally or on a remote server.

## When NOT to use

- The analysis object has not yet been processed (signature autocorrelations, Hotspot modules, and projections not computed) — build the analysis first, then render
- The goal is static publication figures rather than interactive exploration; the report is designed for interactive, shareable exploration
- Only lineage visualization without expression integration is needed — tools like iTOL or CeLaVi handle tree-only viewing

## Inputs

- Analyzed VISION/PhyloVision R object (with signature autocorrelations, Hotspot modules, and projections computed)
- Optional saved report state: DE cache and selections files loaded via load_de_cache and load_selections
- Port number and host configuration for serving (e.g., host='0.0.0.0' on a remote server)

## Outputs

- Interactive web-based report served over HTTP (phylovision_report.html)
- Accompanying JSON state files backing the report UI
- Report UI components: collapsable signature/Hotspot module tables, radial/linear phylogeny display (PhyloPlot.js), UMAP/tSNE overlay panels

## How to apply

Load the analyzed VISION/PhyloVision object into an R session with the VISION package installed from github.com/YosefLab/VISION. Optionally restore previously saved report state with load_de_cache and load_selections so prior differential-expression and selection results reappear in the report. Invoke viewResults on the processed object to build and serve the dynamic web report on a chosen port; when hosting on a remote server, use host='0.0.0.0' and browser=FALSE. Capture the served report assets (HTML plus JSON state) as phylovision_report.html with accompanying JSON files. The report front-end is VISION's, presenting signature Geary's C autocorrelation results, collapsable signature/Hotspot module tables, and phylogeny plus projection panels for interactive exploration.

## Related tools

- **PhyloVision** (Integrated pipeline whose viewResults step builds and serves the interactive web report of lineage and expression analysis) — https://github.com/YosefLab/VISION
- **VISION** (Provides the R package and front-end on which the interactive web-based report is built; supplies viewResults, load_de_cache, and load_selections) — https://github.com/YosefLab/VISION

## Examples

```
viewResults(vio, port=8788, host='0.0.0.0', browser=FALSE)
```

## Evaluation signals

- Report files exist on disk after viewResults (file_exists check on phylovision_report.html and accompanying JSON files)
- phylovision_report.html is parseable HTML (format_is check)
- Report contains the expected structural elements: collapsable signature and Hotspot module tables with FDR < 0.05 entries
- Report embeds the radial/linear phylogeny display backed by PhyloPlot.js
- Report includes UMAP/tSNE overlay panels and loads restored DE-cache/selection state correctly (expert_review)

## Limitations

- PhyloVision is implemented in R, which may pose challenges for using separate Python software (e.g., Cassiopeia) alongside it
- The dynamic phylogeny viewer was tested on trees of up to approximately 4,000 leaves; much larger trees may not render with low latency
- The analysis presented in the study still lacks experimental validation of the generated hypotheses; the report presents exploratory, hypothesis-generating results

## Evidence

- [abstract] PhyloVision provides a feature-rich, interactive, and shareable web-based report for investigating these modules while also supporting several other data and meta-data exploration capabilities: "PhyloVision provides a feature-rich, interactive, and shareable web-based report for investigating these modules while also supporting several other data and meta-data exploration capabilities"
- [results] Our interactive web-based report is built on our VISION front-end (DeTomaso et al., 2019): "Our interactive web-based report is built on our VISION front-end (DeTomaso et al., 2019)"
- [results] allowing low-latency selections and subtree collapsing for large trees (we tested trees of up to ∼4,000 leaves): "allowing low-latency selections and subtree collapsing for large trees (we tested trees of up to ∼4,000 leaves)"
- [discussion] our implementation of PhyloVision in the R programming language may provide challenges for utilizing separate software available in Python: "our implementation of PhyloVision in the R programming language may provide challenges for utilizing separate software available in Python, like Cassiopeia"
- [intro] PhyloVision is distributed publicly on Github at https://github.com/YosefLab/VISION: "PhyloVision is distributed publicly on Github at https://github.com/YosefLab/VISION"
