---
name: gene-expression-matrix-filtering
description: Use when a raw or quantile-normalized microarray matrix (e.g. the PIMENTo
  example_unnorm.txt tab-separated file) contains a large low-intensity background
  population that must be separated from signal before differential analysis.
when_to_use_negative:
- The input is already a filtered/background-subtracted matrix — reapplying the cutoff
  would remove real signal.
- The data are not microarray intensities (e.g. RNA-seq counts); PIMENTo is built
  for Illumina Bead array / DNA microarray data and histogram-based intensity cutoffs
  do not transfer to count data.
- The quantile-normalization step (PIMENTo Preprocess Data) has not yet been run,
  since the cutoff histogram is defined on the quantile-normalized intensities.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: R package providing the Background Cutoff histograms and background subtraction
    routine applied to the quantile-normalized matrix
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Programming language in which PIMENTo is built and run
- name: RStudio
  role: IDE used to run PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:c4d2df1ea4d930f57836542dd0c577b7c17586841bef293e05d8212a03f38b63
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-expression-matrix-filtering

## Summary

A PIMENTo-based background subtraction step for microarray expression matrices: a background cutoff is identified from the quantile-normalized data via intensity histograms (log2 cutoff of 6.1 in the worked example), and all genes whose maximal intensity does not exceed that cutoff are removed to yield a background-subtracted matrix for downstream analysis.

## When to use

Use when a raw or quantile-normalized microarray matrix (e.g. the PIMENTo example_unnorm.txt tab-separated file) contains a large low-intensity background population that must be separated from signal before differential analysis.

## When NOT to use

- The input is already a filtered/background-subtracted matrix — reapplying the cutoff would remove real signal.
- The data are not microarray intensities (e.g. RNA-seq counts); PIMENTo is built for Illumina Bead array / DNA microarray data and histogram-based intensity cutoffs do not transfer to count data.
- The quantile-normalization step (PIMENTo Preprocess Data) has not yet been run, since the cutoff histogram is defined on the quantile-normalized intensities.

## Inputs

- Quantile-normalized expression matrix from PIMENTo Preprocess Data (preprocess.data.obj$quantile)
- Tab-separated raw input file (e.g. example_unnorm.txt)

## Outputs

- Background Cutoff histogram over the full intensity range
- Zoomed Background Cutoff histogram over the [5,7] log2-intensity range
- Background-subtracted data matrix (Background.Subtraction.obj$data) containing only genes whose maximal intensity exceeds the 6.1 log2 cutoff

## How to apply

Load the quantile-normalized matrix produced by the PIMENTo Preprocess Data command (preprocess.data.obj$quantile) from the tab-separated input file example_unnorm.txt. Run the PIMENTo background subtraction routine to render the Background Cutoff histogram over the full intensity range, then render the zoomed histogram over the [5,7] log2-intensity range to resolve the valley separating background from signal. Inspect the histograms to determine the optimal cutoff (6.1 for the mouse wild-type vs villin-gp130Act intestinal crypt dataset) and confirm the histogram inflection matches the reported value. Apply the cutoff by removing all genes whose maximal intensity is not greater than 6.1, storing the background-subtracted matrix in Background.Subtraction.obj$data for downstream analysis.

## Related tools

- **PIMENTo** (R package providing the Background Cutoff histograms and background subtraction routine applied to the quantile-normalized matrix) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Programming language in which PIMENTo is built and run)
- **RStudio** (IDE used to run PIMENTo commands with autocomplete capability)

## Examples

```
install.packages("devtools"); source("https://bioconductor.org/biocLite.R"); biocLite(c("affy","limma","DESeq2")); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- The histogram inflection in the zoomed [5,7] log2-intensity plot must match the reported optimal background cutoff of 6.1.
- The background-subtracted matrix (Background.Subtraction.obj$data) must contain no genes with normalized intensity below 6.1.
- Every retained gene must have a maximal intensity strictly greater than the cutoff — no gene whose max is <= 6.1 survives subtraction.
- The output matrix is a strict subset of the input quantile-normalized matrix (same columns/samples, equal or fewer genes).
- The downstream QC evidence chain must hold: quantile normalization chosen via MA plots (linear on the MA scale) and no batch effect confirmed by clustering before filtering.

## Limitations

- The optimal cutoff (6.1 here) is dataset-specific and must be re-derived from the histogram valley for each new experiment; 6.1 is not a universal constant.
- Cutoff selection depends on visual inspection of the zoomed histogram, introducing operator subjectivity.
- No version numbers are stated for PIMENTo or its dependencies (limma, affy, samr, R, Bioconductor), and a GUI (R Shiny) was not yet available at publication time, so only R-command usage is supported.
- The analysis dataset (wild-type vs villin-gp130Act small intestinal crypt expression profiles) has no independent accession given beyond Array Express E-MEXP-E-MTAB-2400.

## Evidence

- [other] After examining the zoomed log2-scale histogram, a cutoff point of 6.1 was determined as optimal, and Background Subtraction removes all genes whose maximal intensity is not greater than this value: [evidence span withheld — anchor status: not_found]
- [other] Background-subtracted matrix validation: [evidence span withheld — anchor status: not_found]
- [methods] Tab-separated input file format: "a tab-separated sample input file “example_unnorm.txt” is used (see Fig. 1)."
- [readme] PIMENTo is an R package for microarray data: "PIMENTo is an R package to normalize, analyze, and visualize microarray data"
- [methods] Quantile normalization determined best for this dataset: [evidence span withheld — anchor status: not_found]
