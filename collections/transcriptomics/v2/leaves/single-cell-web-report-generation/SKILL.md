---
name: single-cell-web-report-generation
description: Use when a completed VISION/PhyloVision analysis object (expression +
  lineage + signatures) must be rendered as an interactive HTML web report for sharing
  and exploration.
when_to_use_negative:
- Only a raw expression matrix and lineage tree are available — the PhyloVision pipeline
  (filterGenesFano, runHotspot, compute_autocorrelations, clustering) must run first
  to produce an analyzed object
- A static publication figure is needed rather than an interactive, shareable web
  report
- The dataset has no phylogenetic/lineage structure — plain VISION scRNA-seq functional
  interpretation covers expression-only reports
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: PhyloVision
  role: Source of the analyzed object (signatures, Hotspot modules, phylogenetic autocorrelations)
    whose viewResults() call produces the web report
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Front-end on which the interactive web-based report is built; supplies load_de_cache,
    load_selections, addProjection, and viewResults functions
  repo: https://github.com/YosefLab/VISION
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:9c5e43626f4f996bd8f7a5ce9bc2aa9e79d63c4c60b14fc814754c9f76c17daa
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-web-report-generation

## Summary

Generates PhyloVision's feature-rich, interactive, shareable web-based report from a completed single-cell lineage-tracing analysis, so that gene signature enrichments, Hotspot heritable gene modules, and phylogeny/UMAP overlays can be explored in a browser.

## When to use

Use when a completed VISION/PhyloVision analysis object (expression + lineage + signatures) must be rendered as an interactive HTML web report for sharing and exploration.

## When NOT to use

- Only a raw expression matrix and lineage tree are available — the PhyloVision pipeline (filterGenesFano, runHotspot, compute_autocorrelations, clustering) must run first to produce an analyzed object
- A static publication figure is needed rather than an interactive, shareable web report
- The dataset has no phylogenetic/lineage structure — plain VISION scRNA-seq functional interpretation covers expression-only reports

## Inputs

- Analyzed VISION/PhyloVision R object (expression matrix, signatures, phylogeny, Hotspot modules, autocorrelation results)
- Previously saved report state (differential-expression cache and cell selections restored via load_de_cache / load_selections)
- Optional 2D projections added with addProjection() (e.g., UMAP or tSNE embeddings)

## Outputs

- phylovision_report.html — interactive web report with signature/Hotspot module tables, PhyloPlot.js radial/linear phylogeny view, and UMAP/tSNE overlay panels
- Accompanying JSON state files encoding the report contents and interactive selections

## How to apply

Load the analyzed VISION/PhyloVision object into R with the VISION package installed from github.com/YosefLab/VISION, optionally restoring saved report state via load_de_cache and load_selections before rendering. Invoke viewResults on the processed object to build and serve the dynamic report; when hosting on a remote server, pass host='0.0.0.0' and browser=FALSE and choose a port. Capture the resulting phylovision_report.html and accompanying JSON state files, which can be hosted locally or externally. The report front-end is VISION's, so any projection added with addProjection() appears as an overlay panel, and Hotspot module/signature tables render as collapsable tables with FDR-filtered (FDR < 0.05) entries.

## Related tools

- **PhyloVision** (Source of the analyzed object (signatures, Hotspot modules, phylogenetic autocorrelations) whose viewResults() call produces the web report) — https://github.com/YosefLab/VISION
- **VISION** (Front-end on which the interactive web-based report is built; supplies load_de_cache, load_selections, addProjection, and viewResults functions) — https://github.com/YosefLab/VISION

## Examples

```
viewResults(phylovision_object, port = 8118, host = '0.0.0.0', browser = FALSE)
```

## Evaluation signals

- Report files exist on disk (file_exists check on phylovision_report.html and its JSON state files)
- Output parses as valid HTML (format_is check)
- Report contains collapsable signature and Hotspot module tables with expected FDR < 0.05 entries
- Radial/linear phylogeny display is present and driven by PhyloPlot.js
- UMAP/tSNE overlay panels render and match projections added with addProjection()

## Limitations

- The dynamic phylogeny viewer was tested on trees of up to ~4,000 leaves; much larger trees may not support low-latency selections and subtree collapsing
- PhyloVision's R implementation may challenge integration with Python-only software (e.g., Cassiopeia), limiting upstream/downstream tool interop around the report

## Evidence

- [abstract] PhyloVision provides a feature-rich, interactive, and shareable web-based report for investigating these modules while also supporting several other data and meta-data exploration capabilities: "PhyloVision provides a feature-rich, interactive, and shareable web-based report for investigating these modules while also supporting several other data and meta-data exploration capabilities"
- [results] Our interactive web-based report is built on our VISION front-end (DeTomaso et al., 2019): "Our interactive web-based report is built on our VISION front-end (DeTomaso et al., 2019)"
- [methods] the `addProjection()` method can be used to add this view of the data to the output report: "the `addProjection()` method can be used to add this view of the data to the output report"
- [intro] PhyloVision is distributed publicly on Github at https://github.com/YosefLab/VISION: "PhyloVision is distributed publicly on Github at https://github.com/YosefLab/VISION"
- [results] allowing low-latency selections and subtree collapsing for large trees (we tested trees of up to ∼4,000 leaves): "allowing low-latency selections and subtree collapsing for large trees (we tested trees of up to ∼4,000 leaves)"
