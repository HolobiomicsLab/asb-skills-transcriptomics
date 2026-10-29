---
name: microarray-background-subtraction
description: Use when a quantile-normalized microarray intensity matrix (e.g. PIMENTo's
  preprocess.data.obj$quantile from a tab-separated file like example_unnorm.txt)
  shows a distinct low-intensity background peak in its log2 histogram that must be
  filtered out before downstream expression analysis.
when_to_use_negative:
- Input is RNA-seq count data rather than microarray intensity values — the cutoff
  is defined on normalized array log-intensities, not read counts
- Data has not yet been normalized — run the PIMENTo Preprocess Data step (quantile
  or LOESS normalization) first, since the cutoff is chosen on the quantile-normalized
  histogram
- The intensity distribution shows no separable background population in the [5,7]
  log2 zoomed histogram, so no defensible valley/inflection point exists to set the
  cutoff
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: R package providing the Preprocess Data command (quantile normalization) and
    the Background Subtraction routine that renders the cutoff histograms and removes
    genes with maximal intensity below the cutoff
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Runtime environment in which PIMENTo's background subtraction commands are
    executed (installable via devtools from GitHub)
- name: RStudio
  role: Open-source IDE used to run the PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:b118f08ec43f5e47f73716abcee35d491cdf76be6f560c6012659ab435735d01
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# microarray-background-subtraction

## Summary

Identifies a background-vs-signal intensity cutoff from a PIMENTo-rendered Background Cutoff histogram of quantile-normalized microarray data, then removes all genes whose maximal intensity does not exceed that cutoff (6.1 on the log2 scale for the example data) to yield a background-subtracted matrix for downstream analysis.

## When to use

Use when a quantile-normalized microarray intensity matrix (e.g. PIMENTo's preprocess.data.obj$quantile from a tab-separated file like example_unnorm.txt) shows a distinct low-intensity background peak in its log2 histogram that must be filtered out before downstream expression analysis.

## When NOT to use

- Input is RNA-seq count data rather than microarray intensity values — the cutoff is defined on normalized array log-intensities, not read counts
- Data has not yet been normalized — run the PIMENTo Preprocess Data step (quantile or LOESS normalization) first, since the cutoff is chosen on the quantile-normalized histogram
- The intensity distribution shows no separable background population in the [5,7] log2 zoomed histogram, so no defensible valley/inflection point exists to set the cutoff

## Inputs

- Tab-separated unnormalized microarray intensity file (e.g. example_unnorm.txt)
- Quantile-normalized matrix from PIMENTo Preprocess Data (preprocess.data.obj$quantile)
- Background cutoff value on the log2 intensity scale (e.g. 6.1, chosen from the zoomed histogram)

## Outputs

- Background-subtracted data matrix stored in Background.Subtraction.obj$data
- Background Cutoff histogram over the full intensity range
- Zoomed Background Cutoff histogram over the [5,7] log-intensity range

## How to apply

Load the quantile-normalized matrix produced by the PIMENTo Preprocess Data command (stored as preprocess.data.obj$quantile) from the tab-separated input file (e.g. example_unnorm.txt). Run the PIMENTo background subtraction routine to render the Background Cutoff histogram over the full intensity range, then render the zoomed histogram over the [5,7] log-intensity range to resolve the valley separating background from signal. Inspect the zoomed histogram and select the cutoff at the valley/inflection point — 6.1 for the reported example data set. Apply the cutoff by removing all genes whose maximal intensity is not greater than this value, storing the filtered matrix in Background.Subtraction.obj$data. The rationale is that genes whose maximal normalized intensity across samples falls at or below the background cutoff carry no reliable signal, so removing them prevents background noise from confounding downstream QC, dendrogram clustering, and significance testing (samr).

## Related tools

- **PIMENTo** (R package providing the Preprocess Data command (quantile normalization) and the Background Subtraction routine that renders the cutoff histograms and removes genes with maximal intensity below the cutoff) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Runtime environment in which PIMENTo's background subtraction commands are executed (installable via devtools from GitHub))
- **RStudio** (Open-source IDE used to run the PIMENTo commands with autocomplete capability)

## Evaluation signals

- The inflection/valley observed in the zoomed [5,7] log-intensity Background Cutoff histogram matches the selected optimal cutoff (6.1 for the example data)
- Background.Subtraction.obj$data contains no genes whose maximal normalized intensity is not greater than the chosen cutoff (no genes remaining at or below log2 = 6.1)
- The number of genes in Background.Subtraction.obj$data is smaller than or equal to the number in preprocess.data.obj$quantile, and all retained genes exceed the cutoff
- The input matrix is confirmed to be the quantile-normalized output of PIMENTo's Preprocess Data command before cutoff selection
- The full-range and zoomed Background Cutoff histograms are rendered successfully as visual QC artifacts

## Limitations

- The optimal cutoff (6.1 in the example) is data-dependent and must be re-inspected from the zoomed histogram for each new data set; it should not be applied blindly
- Background removal discards genes whose maximal intensity is not greater than the cutoff, so genuinely low-expressing genes are permanently removed from downstream analysis
- PIMENTo is only available as an R package via GitHub at the time described (no R Shiny UI yet — a future release was promised); no version numbers are stated for its dependencies (limma, affy, samr, Bioconductor)

## Evidence
