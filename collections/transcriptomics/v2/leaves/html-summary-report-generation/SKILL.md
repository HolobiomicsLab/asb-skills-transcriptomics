---
name: html-summary-report-generation
description: Use when a COPILOT quality-filtering run on scKB spliced/unspliced gene-by-cell
  matrices must deliver a documented HTML summary (cell statistics, UMI histograms)
  for QC inspection or validation.
when_to_use_negative:
- No summary/QC report is needed and only the filtered matrices are required — the
  Seurat/annotation steps (do.seurat=TRUE) still write a summary but the overhead
  is unnecessary for matrix-only output
- Running in an R environment without an X11 device (e.g. some conda-installed R),
  where COPILOT plotting for the summary file fails
- Input is not raw gene-by-cell UMI count matrices (e.g. already quality-filtered
  or Seurat-processed objects), which COPILOT is not designed to summarize
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COPILOT
  role: Quality-filtering tool that generates the HTML summary report with cell statistics
    and UMI histogram plots
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: R2HTML
  role: R dependency used by COPILOT to write the HTML summary file
- name: scKB
  role: Upstream pipeline producing the raw spliced/unspliced count matrices fed into
    COPILOT for summary generation
- name: DropletUtils
  role: R dependency of COPILOT used in the quality-filtering run that the summary
    reports on
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:22913009d64641c6b76240d9673996e101862e45fd01cd0281ecfbb18474b6db
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - COPILOT
  - DropletUtils
  - R2HTML
  - scKB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# html-summary-report-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates an HTML summary report from the COPILOT scRNA-seq quality-filtering run, containing cell statistics and UMI histogram plots comparable to CellRanger 3.1's summary output, using the built-in R2HTML reporting shipped with COPILOT.

## When to use

Use when a COPILOT quality-filtering run on scKB spliced/unspliced gene-by-cell matrices must deliver a documented HTML summary (cell statistics, UMI histograms) for QC inspection or validation.

## When NOT to use

- No summary/QC report is needed and only the filtered matrices are required — the Seurat/annotation steps (do.seurat=TRUE) still write a summary but the overhead is unnecessary for matrix-only output
- Running in an R environment without an X11 device (e.g. some conda-installed R), where COPILOT plotting for the summary file fails
- Input is not raw gene-by-cell UMI count matrices (e.g. already quality-filtered or Seurat-processed objects), which COPILOT is not designed to summarize

## Inputs

- Raw spliced gene-by-cell count matrix (with row and column names) from scKB
- Raw unspliced gene-by-cell count matrix (with row and column names) from scKB
- COPILOT argument values: sample.name, mt.pattern (e.g. 'ATMG'), filtered.mtx.output.dir, mt.threshold, top.percent, filtering.ratio, min.UMI.low.quality, min.UMI.high.quality

## Outputs

- COPILOT HTML summary file with cell statistics and UMI histogram plots (CellRanger 3.1-comparable format)
- Quality-filtered spliced and unspliced gene-by-cell matrices written to filtered.mtx.output.dir in 10x format

## How to apply

Run the exported copilot() function from the COPILOT R package (installed via devtools::install_github('Hsu-Che-Wei/COPILOT'), requiring the R2HTML dependency) on the raw spliced and unspliced gene-by-cell matrices produced by scKB, passing mt.pattern (e.g. 'ATMG' for Arabidopsis), filtered.mtx.output.dir, and QC parameters such as mt.threshold=5, top.percent=1, filtering.ratio=1, min.UMI.low.quality=100, min.UMI.high.quality=300. As part of the run, COPILOT automatically builds the HTML summary via R2HTML, embedding cell-filtering statistics and UMI histogram plots; note the summary includes feature plots, which require an X11 device. The summary file's utility is directly comparable to CellRanger 3.1, so validate it by confirming the HTML file exists in the output directory and contains the expected cell statistics. Avoid running in conda-installed R environments without X11 support, a documented failure mode for summary-file feature plots.

## Related tools

- **COPILOT** (Quality-filtering tool that generates the HTML summary report with cell statistics and UMI histogram plots) — https://github.com/Hsu-Che-Wei/COPILOT
- **R2HTML** (R dependency used by COPILOT to write the HTML summary file)
- **scKB** (Upstream pipeline producing the raw spliced/unspliced count matrices fed into COPILOT for summary generation)
- **DropletUtils** (R dependency of COPILOT used in the quality-filtering run that the summary reports on)

## Examples

```
copilot(sample.name="col0", spliced.mtx=spliced, unspliced.mtx=unspliced, mt.pattern="ATMG", filtering.ratio=1, remove.doublet=FALSE, do.seurat=FALSE, filtered.mtx.output.dir="copilot_out/")
```

## Evaluation signals

- HTML summary file exists in the filtered.mtx.output.dir output directory after the run completes
- Summary contains the expected cell-filtering statistics (number of cells before/after filtering, UMI distributions) consistent with the filtering.ratio, mt.threshold, and min.UMI parameter values used
- Summary content and utility are directly comparable to CellRanger 3.1 summary output
- Quality-filtered spliced and unspliced matrices in 10x format are present alongside the summary in the output directory
- No plotting errors related to X11 device unavailability occurred during summary feature-plot generation

## Limitations

- Summary feature plots require an X11 device, which is unavailable in some conda-installed R environments and causes COPILOT plotting failures
- The CellRanger-comparable summary reflects COPILOT's filtering decisions (e.g. dying cells above mt.threshold, low-UMI barcodes below min.UMI.low.quality), so statistics are only meaningful for the given parameter stringency
- COPILOT's annotation-related output is limited to root of Arabidopsis thaliana

## Evidence

- [other] CellRanger 3.1-comparable summary output: [evidence span withheld — anchor status: not_found]
- [readme] R2HTML dependency: [evidence span withheld — anchor status: not_found]
