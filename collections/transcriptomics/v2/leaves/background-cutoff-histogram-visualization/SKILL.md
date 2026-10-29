---
name: background-cutoff-histogram-visualization
description: Use when a quantile-normalized microarray intensity matrix (e.g. PIMENTo's
  preprocess.data.obj$quantile) shows a bimodal log2 intensity distribution and a
  background/signal valley must be identified to define a background-subtracted gene
  set.
when_to_use_negative:
- The input matrix is raw (not yet quantile-normalized by PIMENTo's Preprocess Data
  command) — normalization must come first.
- The goal is normalization-method comparison (quantile vs LOESS via MA plots) rather
  than removing genes below a background threshold.
- Data are RNA-seq count data rather than microarray intensities from platforms such
  as Illumina Bead arrays; PIMENTo targets microarray QC and analysis.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0092
tools:
- name: PIMENTo
  role: R package providing the background subtraction routine that renders the Background
    Cutoff histograms and applies the cutoff to remove background genes
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: language in which PIMENTo is built and in which the histograms and cutoff
    filtering are executed
- name: RStudio
  role: open-source IDE used to run PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:0964d49c93a0ecf34993a05960be960d31fc6fb7fb46a8ebc56e297b3bf75ea9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# background-cutoff-histogram-visualization

## Summary

A PIMENTo (R) routine that renders full-range and zoomed log2-scale Background Cutoff histograms of quantile-normalized microarray intensities so the valley separating background from signal can be visually located and used to set an intensity cutoff (here 6.1) for background subtraction.

## When to use

Use when a quantile-normalized microarray intensity matrix (e.g. PIMENTo's preprocess.data.obj$quantile) shows a bimodal log2 intensity distribution and a background/signal valley must be identified to define a background-subtracted gene set.

## When NOT to use

- The input matrix is raw (not yet quantile-normalized by PIMENTo's Preprocess Data command) — normalization must come first.
- The goal is normalization-method comparison (quantile vs LOESS via MA plots) rather than removing genes below a background threshold.
- Data are RNA-seq count data rather than microarray intensities from platforms such as Illumina Bead arrays; PIMENTo targets microarray QC and analysis.

## Inputs

- quantile-normalized intensity matrix (PIMENTo preprocess.data.obj$quantile) produced by the Preprocess Data command
- tab-separated raw intensity input file (e.g. example_unnorm.txt)

## Outputs

- full-range Background Cutoff histogram (log2 intensity vs frequency)
- zoomed Background Cutoff histogram over the [5,7] log-intensity range
- determined background cutoff value (log2 = 6.1 for the example data)
- background-subtracted matrix stored in Background.Subtraction.obj$data containing only genes with maximal intensity above the cutoff

## How to apply

Load the quantile-normalized matrix from the PIMENTo Preprocess Data step (stored in preprocess.data.obj$quantile after reading the tab-separated file example_unnorm.txt), then run the PIMENTo background subtraction routine to render the Background Cutoff histogram over the full intensity range. Next render the zoomed histogram restricted to the [5,7] log2-intensity range, which resolves the valley between the background mode and the signal peak where the full-range plot is ambiguous. Inspect the inflection in the zoomed plot and record the optimal cutoff — for the villin-gp130Act vs WT small intestinal crypt microarray data this was log2 = 6.1. Apply the cutoff so that all genes whose maximal intensity is not greater than 6.1 are removed, yielding Background.Subtraction.obj$data for downstream analysis.

## Related tools

- **PIMENTo** (R package providing the background subtraction routine that renders the Background Cutoff histograms and applies the cutoff to remove background genes) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (language in which PIMENTo is built and in which the histograms and cutoff filtering are executed)
- **RStudio** (open-source IDE used to run PIMENTo commands with autocomplete capability)

## Examples

```
install.packages("devtools"); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- The inflection/valley visible in the zoomed [5,7] log2 histogram must match the reported optimal cutoff of 6.1.
- Background.Subtraction.obj$data must contain no genes with normalized intensity at or below the 6.1 cutoff (i.e. every retained gene's maximal intensity exceeds 6.1).
- The input to the routine must be exactly the quantile-normalized matrix from PIMENTo's Preprocess Data command, not the raw matrix.
- The zoomed histogram must resolve a clear valley separating the background mode from the signal peak; if no valley is visible in [5,7], the cutoff choice is unreliable and should not be applied blindly.
- Gene row counts must drop between the normalized matrix and Background.Subtraction.obj$data, reflecting removal of background-only genes.

## Limitations

- The cutoff (6.1) is data-dependent: it was determined by visual inspection of the zoomed histogram for this specific wild-type vs villin-gp130Act microarray dataset and must be re-derived for other datasets.
- The rule removes a gene if its maximal intensity is not greater than the cutoff, so low-expression but biologically relevant genes in a subset of samples can be discarded.
- No version numbers are stated for PIMENTo or its dependencies (affy, limma, DESeq2), and the promised R Shiny user interface was not yet available, so the routine must be driven from R commands.

## Evidence

- [readme] PIMENTo is an R package to normalize, analyze, and visualize microarray data: "PIMENTo is an R package to normalize, analyze, and visualize microarray data"
- [methods] PIMENTo Preprocess Data first step: "The first step in the PIMENTo pipeline is normalization and visualization through the command Preprocess Data."
- [methods] example input file: "a tab-separated sample input file “example_unnorm.txt” is used (see Fig. 1)."
- [other] cutoff finding: [evidence span withheld — anchor status: not_found]
- [readme] PIMENTo installation: "devtools::install_github("MUSC-CGM/PIMENTo")"
