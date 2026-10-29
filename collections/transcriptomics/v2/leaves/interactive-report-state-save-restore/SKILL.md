---
name: interactive-report-state-save-restore
description: Use when an analyzed PhyloVision/VISION object must be delivered as an
  interactive HTML+JSON web report, or when a previously rendered report's saved state
  (differential expression cache, selections) must be restored before serving.
when_to_use_negative:
- No analysis has been run yet — the object must already contain computed signature/Hotspot
  module results before a report can be viewed
- Static figures or publication plots are the goal rather than interactive exploration
  of phylogeny, module tables, and UMAP/tSNE overlays
- The analysis lives in Python-only tooling; the article notes the R implementation
  may challenge use of separate Python software like Cassiopeia at report-serving
  time
tools:
- name: PhyloVision
  role: R pipeline whose viewResults call builds and serves the interactive web report;
    supports load_de_cache/load_selections state restoration
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Front-end framework on which the interactive web-based report is built
  repo: https://github.com/YosefLab/VISION
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:324990ceb68f7da00b3706b1ddf194efed139d854c0bcfda6455413049197a63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - PhyloVision
  - R
  - VISION
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# interactive-report-state-save-restore

## Summary

Generate and re-serve the feature-rich, interactive web-based report of an analyzed PhyloVision (VISION) single-cell lineage-tracing object, optionally restoring previously saved differential-expression and cell-selection state so that hosted reports remain fully explorable locally or on a remote server.

## When to use

Use when an analyzed PhyloVision/VISION object must be delivered as an interactive HTML+JSON web report, or when a previously rendered report's saved state (differential expression cache, selections) must be restored before serving.

## When NOT to use

- No analysis has been run yet — the object must already contain computed signature/Hotspot module results before a report can be viewed
- Static figures or publication plots are the goal rather than interactive exploration of phylogeny, module tables, and UMAP/tSNE overlays
- The analysis lives in Python-only tooling; the article notes the R implementation may challenge use of separate Python software like Cassiopeia at report-serving time

## Inputs

- Analyzed VISION/PhyloVision R object (single-cell expression plus lineage/phylogeny, with Hotspot modules and signature scores computed)
- Saved report state files: differential-expression cache and selections JSON
- Optional local port number and host configuration for serving

## Outputs

- Interactive web-based report served over HTTP (HTML with embedded VISION front-end)
- phylovision_report.html plus accompanying JSON state files capturing the report

## How to apply

Load the analyzed VISION/PhyloVision object in R with the VISION package installed from github.com/YosefLab/VISION. If saved report state exists, restore it into the object with load_de_cache and load_selections before rendering, so prior differential-expression analyses and user cell selections appear in the regenerated report. Invoke viewResults on the object to build and serve the dynamic web report; when hosting remotely, pass host='0.0.0.0' and browser=FALSE so the server binds externally without launching a local browser. The report is built on the VISION front-end and exposes collapsable signature/Hotspot module tables (FDR < 0.05 entries), a radial/linear phylogeny via PhyloPlot.js, and UMAP/tSNE overlay panels. Capture the resulting HTML plus accompanying JSON state files (e.g., phylovision_report.html) as the shareable artifact.

## Related tools

- **PhyloVision** (R pipeline whose viewResults call builds and serves the interactive web report; supports load_de_cache/load_selections state restoration) — https://github.com/YosefLab/VISION
- **VISION** (Front-end framework on which the interactive web-based report is built) — https://github.com/YosefLab/VISION

## Examples

```
v <- load_de_cache(v, "de_cache."); v <- load_selections(v, "selections."); viewResults(v, port = 8123, host = '0.0.0.0', browser = FALSE)
```

## Evaluation signals

- Report files exist after rendering (file_exists check on phylovision_report.html and companion JSON files)
- Report HTML is parseable (format_is HTML) and contains expected structural elements
- Collapsable signature/Hotspot module tables are present with FDR < 0.05 entries populated
- Phylogeny display (PhyloPlot.js radial/linear view) and UMAP/tSNE overlay panels render in the served report
- Restored state is visible: previously saved differential-expression results and cell selections appear in the regenerated report

## Limitations

- Report serving requires an R session with the VISION package installed; the R implementation may limit interoperability with separate Python software such as Cassiopeia
- State restoration depends on having previously saved de-cache and selections files; absent these, regenerated reports lose prior analyses
- Remote hosting requires binding to host='0.0.0.0' with browser=FALSE, so report access depends on network/port configuration

## Evidence

- [abstract] abstract: "PhyloVision provides a feature-rich, interactive, and shareable web-based report for investigating these modules"
- [results] results: "Our interactive web-based report is built on our VISION front-end (DeTomaso et al., 2019)"
- [intro] intro: "PhyloVision is distributed publicly on Github at https://github.com/YosefLab/VISION"
