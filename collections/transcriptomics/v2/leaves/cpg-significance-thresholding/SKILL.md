---
name: cpg-significance-thresholding
description: Use when a DMC results table from methylR's ChAMP- or minfi-based differential
  analysis must be thresholded on adjusted P-value and logFC to identify significant
  CpGs and visualize them on a volcano plot.
when_to_use_negative:
- Input is raw IDAT or a normalized beta-value table (testDataFile2) intended for
  MDS/PCA — the volcano module requires the DMC results file, not beta values
- 'Differential analysis has not yet been run: no DMC results exist to threshold,
  so the workflow should start with the ChAMP/minfi step instead'
- The goal is set intersection across multiple CpG lists rather than single-comparison
  thresholding — the intervene tool in the context addresses intersection/visualization
  of multiple sets
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: methylR
  role: Provides the shiny-based Feature Analysis Section in which adjusted P-value
    and logFC cutoffs are applied to DMC results to generate the volcano plot and
    significant CpG list
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: One of the two upstream differential analysis backends whose DMC output is
    consumed by the volcano plot thresholding step
- name: minfi
  role: Alternative upstream differential analysis backend producing the DMC results
    that are thresholded
- name: Docker
  role: Distributes methylR as a container (jd21/methylr) from which the Feature Analysis
    module is run locally
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:61efd18caa01163d928f67a16f4cb81099de975b29d4362c9d615fb37ba517c8
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
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cpg-significance-thresholding

## Summary

Filters CpG probes from differential methylation (DMC) results by user-selected adjusted P-value and log-fold-change cutoffs within methylR's Feature Analysis Section, rendering an interactive volcano plot and exporting the significant CpG list from the same filtered set.

## When to use

Use when a DMC results table from methylR's ChAMP- or minfi-based differential analysis must be thresholded on adjusted P-value and logFC to identify significant CpGs and visualize them on a volcano plot.

## When NOT to use

- Input is raw IDAT or a normalized beta-value table (testDataFile2) intended for MDS/PCA — the volcano module requires the DMC results file, not beta values
- Differential analysis has not yet been run: no DMC results exist to threshold, so the workflow should start with the ChAMP/minfi step instead
- The goal is set intersection across multiple CpG lists rather than single-comparison thresholding — the intervene tool in the context addresses intersection/visualization of multiple sets

## Inputs

- Differentially methylated CpG (DMC) results file from methylR's ChAMP- or minfi-based differential analysis (e.g. testDataFile1 with per-CpG adjusted P-value and logFC columns)
- User-selected adjusted P-value cutoff
- User-selected log-fold-change cutoff

## Outputs

- Interactive volcano plot of -log10(adjusted P-value) versus logFC for all tested CpGs, with significant CpGs highlighted
- Exported list of significant CpG IDs passing both cutoffs

## How to apply

Load the DMC results file produced by a complete ChAMP or minfi analysis inside methylR (a differentially methylated data file, e.g. testDataFile1). In the Feature Analysis Section of the shiny-based interface, set the adjusted P-value cutoff and filter the tested CpG probes, then apply the log-fold-change cutoff to the survivors. Render an interactive volcano plot of -log10(adjusted P-value) versus logFC for all tested CpGs, highlighting the CpGs that pass both thresholds. Export the significant CpG ID list alongside the plot so the two artifacts remain mutually consistent. The rationale is that visual and tabular significance calls derive from one shared filtering step, avoiding divergent gene lists between the figure and downstream modules such as GO and pathway enrichment.

## Related tools

- **methylR** (Provides the shiny-based Feature Analysis Section in which adjusted P-value and logFC cutoffs are applied to DMC results to generate the volcano plot and significant CpG list) — https://github.com/JD2112/methylr
- **ChAMP** (One of the two upstream differential analysis backends whose DMC output is consumed by the volcano plot thresholding step)
- **minfi** (Alternative upstream differential analysis backend producing the DMC results that are thresholded)
- **Docker** (Distributes methylR as a container (jd21/methylr) from which the Feature Analysis module is run locally)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Every CpG in the exported significant list must lie beyond both the adjusted P-value and logFC cutoffs displayed on the volcano plot (plot/list mutual consistency)
- The x-axis of the rendered volcano plot corresponds to logFC and the y-axis to -log10(adjusted P-value) for all tested CpGs from the loaded DMC file
- Changing either cutoff in the shiny interface visibly changes which points are highlighted and updates the exported CpG list accordingly
- The consumed input file is the differentially methylated data file (DMC results), matching the role of testDataFile1 in the methylR test data

## Limitations

- The ARM64 chipset architecture is not supported by methylR; the Docker container is built for AMD64 only, so thresholding must run on compatible hosts (Linux, Intel macOS, or Windows with WSL2)
- Only one complete ChAMP or minfi analysis can run at a time: the browser must be closed between analyses to clear temporary memory, so repeated thresholding runs require restarting the session
- Cutoff values are user-adjusted with no fixed defaults specified in the article, so significance definitions are not standardized across runs

## Evidence

- [readme] DMC results provenance: [evidence span withheld — anchor status: not_found]
- [methods] upstream analysis backend: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [abstract] downstream module context: "We implemented eight downstream analysis modules that can perform multidimensional analyses to pathway enrichment"
