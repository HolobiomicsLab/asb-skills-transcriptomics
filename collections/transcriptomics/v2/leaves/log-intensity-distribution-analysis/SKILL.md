---
name: log-intensity-distribution-analysis
description: Use when a quantile-normalized microarray intensity matrix contains a
  mixed background/signal distribution and a log2-scale histogram inflection must
  be identified to set a background cutoff (e.g. 6.1 in the PIMENTo pipeline).
when_to_use_negative:
- The matrix is already background-subtracted or contains only signal-range intensities
  (no background mixture to resolve).
- The data are not microarray intensity distributions (e.g. RNA-seq count tables),
  where PIMENTo's histogram-based background cutoff does not apply.
- The zoomed [5,7] log-intensity histogram shows no resolvable valley, so the cutoff
  cannot be defensibly read from an inflection.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: R package whose background subtraction routine renders the Background Cutoff
    histograms and applies the cutoff to remove genes
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Programming language in which the PIMENTo background cutoff and subtraction
    commands are run
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: RStudio
  role: Open-source IDE used to run the PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:ed47878f0ddb65da1a11f4a0723e9fbaabe57833316114e779e6357648064e16
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# log-intensity-distribution-analysis

## Summary

Analysis of the log2-scale intensity distribution of quantile-normalized microarray data to locate the valley separating background from signal, yielding a background cutoff (e.g. 6.1) used to strip unexpressed genes from the data matrix before downstream testing.

## When to use

Use when a quantile-normalized microarray intensity matrix contains a mixed background/signal distribution and a log2-scale histogram inflection must be identified to set a background cutoff (e.g. 6.1 in the PIMENTo pipeline).

## When NOT to use

- The matrix is already background-subtracted or contains only signal-range intensities (no background mixture to resolve).
- The data are not microarray intensity distributions (e.g. RNA-seq count tables), where PIMENTo's histogram-based background cutoff does not apply.
- The zoomed [5,7] log-intensity histogram shows no resolvable valley, so the cutoff cannot be defensibly read from an inflection.

## Inputs

- Quantile-normalized microarray intensity matrix (preprocess.data.obj$quantile) from PIMENTo Preprocess Data
- Tab-separated raw input file (e.g. example_unnorm.txt)

## Outputs

- Background Cutoff histogram over the full intensity range
- Zoomed Background Cutoff histogram over the [5,7] log-intensity range
- Determined background cutoff value (e.g. 6.1)
- Background-subtracted data matrix (Background.Subtraction.obj$data) with low-intensity genes removed

## How to apply

Load the quantile-normalized matrix (preprocess.data.obj$quantile) produced by the PIMENTo Preprocess Data command from the tab-separated input file example_unnorm.txt. Render the Background Cutoff histogram over the full intensity range, then render a zoomed histogram restricted to the [5,7] log-intensity range so the valley separating background noise from signal is resolvable. Inspect the zoomed plot to read the optimal cutoff at the inflection (reported as 6.1 for this dataset). Apply the cutoff by removing all genes whose maximal intensity is not greater than 6.1, storing the result in Background.Subtraction.obj$data for downstream significance testing (samr). The rationale is that genes whose maximum normalized log2 intensity never exceeds the background valley carry no signal and would otherwise dilute differential-expression testing.

## Related tools

- **PIMENTo** (R package whose background subtraction routine renders the Background Cutoff histograms and applies the cutoff to remove genes) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Programming language in which the PIMENTo background cutoff and subtraction commands are run) — https://www.github.com/MUSC-CGM/PIMENTo
- **RStudio** (Open-source IDE used to run the PIMENTo commands with autocomplete capability)

## Examples

```
devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- The inflection of the zoomed [5,7] log-intensity histogram matches the reported optimal cutoff of 6.1.
- The background-subtracted matrix (Background.Subtraction.obj$data) contains no genes with normalized intensity below 6.1.
- Every retained gene has a maximal intensity strictly greater than the determined cutoff, per the 'not greater than this value' removal rule.
- The full-range and zoomed histograms are both rendered from the same preprocess.data.obj$quantile matrix, ensuring the cutoff is read from the quantile-normalized data, not the raw file.
- Downstream significance testing (samr) operates only on genes surviving the 6.1 cutoff.

## Limitations

- The cutoff is dataset-specific: 6.1 applies to this example dataset and must be re-read from the histogram inflection for other experiments.
- Cutoff selection relies on visual inspection of the zoomed Background Cutoff histogram, introducing operator judgment.
- No version numbers are stated for the underlying dependencies (limma, affy, samr, R, Bioconductor), affecting reproducibility.

## Evidence
