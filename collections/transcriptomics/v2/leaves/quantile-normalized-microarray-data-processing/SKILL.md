---
name: quantile-normalized-microarray-data-processing
description: Use when microarray intensity data has already been quantile-normalized
  via PIMENTo's Preprocess Data command and background noise must be separated from
  signal by choosing a log2 cutoff from the intensity histogram before downstream
  analysis.
when_to_use_negative:
- Input is raw, un-normalized array intensities — run PIMENTo's Preprocess Data (quantile
  or LOESS normalization) first, since this skill operates on preprocess.data.obj$quantile
- The goal is choosing between normalization methods — quantile vs LOESS selection
  is made on the MA plot before background subtraction
- Data is RNA-seq count data rather than microarray intensities — the histogram-based
  log2 cutoff logic targets microarray background distributions
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PIMENTo
  role: Provides the Preprocess Data command producing the quantile-normalized matrix
    and the Background Cutoff histogram / background subtraction routine
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Language in which PIMENTo is implemented and in which the background subtraction
    commands are executed
- name: RStudio
  role: IDE used to run the PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:6b7f5be6fc9d687f3ebc01b5ae6f13e363a12c90df2936a27b7c196f76ec7a4e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# quantile-normalized-microarray-data-processing

## Summary

Background subtraction of quantile-normalized microarray data in the PIMENTo pipeline: a log2 intensity cutoff (e.g., 6.1 for the example dataset) is identified from the Background Cutoff histogram valley and applied to remove background genes from the data matrix.

## When to use

Use when microarray intensity data has already been quantile-normalized via PIMENTo's Preprocess Data command and background noise must be separated from signal by choosing a log2 cutoff from the intensity histogram before downstream analysis.

## When NOT to use

- Input is raw, un-normalized array intensities — run PIMENTo's Preprocess Data (quantile or LOESS normalization) first, since this skill operates on preprocess.data.obj$quantile
- The goal is choosing between normalization methods — quantile vs LOESS selection is made on the MA plot before background subtraction
- Data is RNA-seq count data rather than microarray intensities — the histogram-based log2 cutoff logic targets microarray background distributions

## Inputs

- Quantile-normalized microarray intensity matrix from PIMENTo Preprocess Data (preprocess.data.obj$quantile)
- Tab-separated raw expression file, e.g. example_unnorm.txt

## Outputs

- Background Cutoff histogram over the full log2 intensity range
- Zoomed Background Cutoff histogram over the [5,7] log-intensity range
- Background-subtracted matrix stored in Background.Subtraction.obj$data (genes with maximal intensity <= cutoff removed)

## How to apply

Load the quantile-normalized matrix (preprocess.data.obj$quantile) produced by PIMENTo's Preprocess Data command from the tab-separated input file (e.g., example_unnorm.txt). Run the PIMENTo background subtraction routine to render the Background Cutoff histogram over the full intensity range, then render the zoomed histogram over the [5,7] log2 range to resolve the valley separating background from signal. Inspect the histogram inflection to select the optimal cutoff — 6.1 was determined for the example dataset. Apply the cutoff by removing all genes whose maximal normalized intensity is not greater than 6.1, storing the result in Background.Subtraction.obj$data for downstream analysis. The visual inspection step is the key decision point, since the cutoff is dataset-specific rather than a fixed universal threshold.

## Related tools

- **PIMENTo** (Provides the Preprocess Data command producing the quantile-normalized matrix and the Background Cutoff histogram / background subtraction routine) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Language in which PIMENTo is implemented and in which the background subtraction commands are executed)
- **RStudio** (IDE used to run the PIMENTo commands with autocomplete capability)

## Examples

```
install.packages("devtools"); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- The inflection/valley of the zoomed Background Cutoff histogram (over log2 range [5,7]) matches the reported optimal cutoff of 6.1 for the example dataset
- The background-subtracted matrix (Background.Subtraction.obj$data) contains no genes with normalized intensity at or below 6.1
- The output matrix has fewer genes than the input quantile-normalized matrix, with removed genes being exactly those whose maximal intensity does not exceed the cutoff
- The full-range and zoomed histograms are mutually consistent in locating the background-signal separation point
- The input matrix is confirmed to be the quantile-normalized output (preprocess.data.obj$quantile), which the paper found superior to LOESS for this dataset

## Limitations

- The cutoff (6.1) is dataset-specific and determined by visual inspection of the histogram, so it must be re-derived per dataset rather than hardcoded
- The chapter deposits the microarray data in Array Express (accession E-MEXP-E-MTAB-2400) but gives no software version numbers for PIMENTo, R, limma, or affy, affecting reproducibility
- PIMENTo at the time of writing was command-line only; the R Shiny user interface was promised in a future release

## Evidence
