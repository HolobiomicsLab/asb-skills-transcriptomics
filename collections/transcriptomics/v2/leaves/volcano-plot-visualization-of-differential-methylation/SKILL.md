---
name: volcano-plot-visualization-of-differential-methylation
description: Use when you already have a differential methylation (DMC) results table
  from Illumina 450K/EPIC array analysis and need to visualize and extract significant
  CpGs by adjusted-P and logFC thresholds.
when_to_use_negative:
- Input is raw IDAT or normalized beta values rather than a completed DMC results
  table — run the methylysis step (ChAMP/minfi) first
- The goal is RNA-seq or sequencing-based differential methylation; methylR is built
  for 450K and EPIC Illumina arrays
- Running on ARM64 architecture, which methylR does not support
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: methylR
  role: Shiny-based graphical tool whose Feature Analysis module filters DMC results
    by adjusted-P and logFC cutoffs and renders the volcano plot
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: Upstream differential methylation analysis engine within methylR producing
    the DMC results consumed by the volcano plot module
- name: minfi
  role: Alternative upstream differential methylation analysis engine within methylR
    producing the DMC results
- name: Docker
  role: Container (jd21/methylr) used to run methylR locally on AMD64 systems to access
    the volcano plot module
  repo: https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:afb241da5cb3aac606dc5d18297722481df4c3a47c3cc40bf36809bf4ff9d9bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Volcano plot visualization of differential methylation

## Summary

Generates an interactive volcano plot of -log10(adjusted P-value) versus log-fold-change from ChAMP- or minfi-based differential methylation results within methylR's Feature Analysis Section, and exports the list of significant CpG IDs passing both user-adjusted cutoffs.

## When to use

Use when you already have a differential methylation (DMC) results table from Illumina 450K/EPIC array analysis and need to visualize and extract significant CpGs by adjusted-P and logFC thresholds.

## When NOT to use

- Input is raw IDAT or normalized beta values rather than a completed DMC results table — run the methylysis step (ChAMP/minfi) first
- The goal is RNA-seq or sequencing-based differential methylation; methylR is built for 450K and EPIC Illumina arrays
- Running on ARM64 architecture, which methylR does not support

## Inputs

- DMC results file from ChAMP- or minfi-based differential methylation analysis (e.g., methylR testDataFile1)
- User-adjusted adjusted-P-value cutoff
- User-adjusted log-fold-change (logFC) cutoff

## Outputs

- Interactive volcano plot of -log10(adjusted P-value) vs logFC with threshold-highlighted CpGs
- Exported list of significant CpG IDs passing both cutoffs

## How to apply

Load the DMC results file produced by methylR's methylysis step (ChAMP- or minfi-based differential analysis) into the shiny-based Feature Analysis module; the test data file testDataFile1 is a differentially methylated data file intended exactly for this module. Apply the user-adjusted adjusted-P-value cutoff to filter CpG probes, then apply the user-adjusted log-fold-change cutoff to the remaining probes. Render an interactive volcano plot of -log10(adjusted P-value) versus logFC for all tested CpGs, highlighting probes that pass both thresholds. Export the significant CpG ID list alongside the plot, ensuring every exported CpG lies beyond both displayed cutoff lines.

## Related tools

- **methylR** (Shiny-based graphical tool whose Feature Analysis module filters DMC results by adjusted-P and logFC cutoffs and renders the volcano plot) — https://github.com/JD2112/methylr
- **ChAMP** (Upstream differential methylation analysis engine within methylR producing the DMC results consumed by the volcano plot module)
- **minfi** (Alternative upstream differential methylation analysis engine within methylR producing the DMC results)
- **Docker** (Container (jd21/methylr) used to run methylR locally on AMD64 systems to access the volcano plot module) — https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # then open http://localhost:3838, load testDataFile1 (DMC results), set adjusted-P and logFC cutoffs in Feature Analysis > Volcano plot
```

## Evaluation signals

- Every CpG in the exported significant list lies beyond both the adjusted-P and logFC cutoff lines shown on the rendered volcano plot
- The plot axes match -log10(adjusted P-value) on one axis and logFC on the other for all tested CpGs
- CpGs highlighted in the plot are exactly those passing both user-specified thresholds
- The exported CpG ID list and the highlighted points derive from the same filtered subset of the input DMC table

## Limitations

- ARM64 chipset architecture is not supported by methylR; the Docker container is built on AMD64
- After one complete analysis (ChAMP or minfi), the browser must be closed to clear temporary memory before starting again
- The module expects a differentially methylated data file (DMC results), not raw or beta-value-only data; beta tables are used by MDS/PCA modules instead

## Evidence

- [results] user selects adjusted P-value and logFC cutoffs; interactive volcano plot and significant CpG list generated from DMC data: [evidence span withheld — anchor status: not_found]
- [readme] testDataFile1 is the DMC file for the Volcano plot module: [evidence span withheld — anchor status: not_found]
- [methods] methylR runs ChAMP or minfi differential analysis: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] methylR presented as a shiny-based web server approach: "Here, in *methylR*, we presented a shiny-based web server approach to minimize the above-mentioned difficulties."
