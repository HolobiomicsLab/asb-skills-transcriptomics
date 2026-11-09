---
name: impossible-barcode-combination-quantification
description: Use when a plate-prepared scRNA-seq library was sequenced on a patterned
  flow-cell instrument (HiSeq 4000, HiSeq X, NovaSeq) and you need to estimate the
  sample index swapping fraction from the raw FASTQ/BCL data.
when_to_use_negative:
- Data were generated on non-patterned flow-cell instruments (e.g., HiSeq 2500), where
  index swapping is negligible and the regression would mostly measure noise
- Droplet-based (e.g., 10x Genomics) data where swapped molecules should instead be
  removed by the UMI/cell-barcode/gene molecule-exclusion algorithm rather than quantified
  by impossible-combination regression
- Only processed count matrices are available; the method requires raw reads and sample
  barcodes to detect impossible combinations during demultiplexing
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: R
  role: Used to fit the simple linear regression of observed swapped reads on reads
    available for swapping and to compute mean and standard deviation of swapping
    fractions across plates
- name: HiSeq 4000
  role: Patterned flow-cell sequencer whose index swapping generates the impossible
    barcode combinations being quantified
- name: BarcodeSwapping2017
  role: Repository containing the supplementary report and code that expands on the
    models and results for estimating swapping fractions
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:75b86f6eef8eed27ec836bfdbd444c0d53e2cdd361e54be65567afaf96a25c0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - BarcodeSwapping2017
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# impossible-barcode-combination-quantification

## Summary

Quantifies the fraction of barcode-swapped reads in plate-based scRNA-seq data (e.g., the Richard et al. mouse T-cell plates sequenced on HiSeq 4000) by demultiplexing to retain 'impossible' sample-barcode combinations and regressing their library sizes on the reads available for swapping.

## When to use

Use when a plate-prepared scRNA-seq library was sequenced on a patterned flow-cell instrument (HiSeq 4000, HiSeq X, NovaSeq) and you need to estimate the sample index swapping fraction from the raw FASTQ/BCL data.

## When NOT to use

- Data were generated on non-patterned flow-cell instruments (e.g., HiSeq 2500), where index swapping is negligible and the regression would mostly measure noise
- Droplet-based (e.g., 10x Genomics) data where swapped molecules should instead be removed by the UMI/cell-barcode/gene molecule-exclusion algorithm rather than quantified by impossible-combination regression
- Only processed count matrices are available; the method requires raw reads and sample barcodes to detect impossible combinations during demultiplexing

## Inputs

- Raw FASTQ/BCL data from plate-prepared Smart-seq2 scRNA-seq libraries (e.g., ArrayExpress E-MTAB-6843 mouse T-cell plates)
- Sample index barcode sequences and plate/sample metadata defining expected barcode combinations
- Optional control FASTQ data of the same libraries sequenced on HiSeq 2500

## Outputs

- Estimated swapped-read fraction (regression gradient) with across-plate standard deviation, e.g., 2.18 ± 0.08%
- swapping_fraction_estimate.txt containing the fraction and uncertainty
- Per-impossible-combination table of observed swapped reads vs. reads available for swapping

## How to apply

Obtain raw plate-prepared data (e.g., ArrayExpress E-MTAB-6843) and demultiplex the HiSeq 4000 libraries allowing reads to be assigned to both expected and 'impossible' sample-barcode combinations. For each impossible combination, tabulate the observed swapped-read count together with the number of reads available for swapping (reads from the two real cells sharing exactly one barcode). Fit a simple linear regression of observed swapped reads on reads available for swapping across the impossible combinations, and use the gradient of the fitted line as the swapped-read fraction estimate; compute the mean and standard deviation of swapping fractions across the two plates as the uncertainty. As a negative control, repeat the identical procedure on the same libraries sequenced on a non-patterned instrument such as the HiSeq 2500, where swapping should be negligible. Write the estimate (e.g., 2.18 ± 0.08%) to swapping_fraction_estimate.txt and report the deviation from the reference ~2.5% HiSeq 4000 swapping rate.

## Related tools

- **R** (Used to fit the simple linear regression of observed swapped reads on reads available for swapping and to compute mean and standard deviation of swapping fractions across plates)
- **HiSeq 4000** (Patterned flow-cell sequencer whose index swapping generates the impossible barcode combinations being quantified)
- **BarcodeSwapping2017** (Repository containing the supplementary report and code that expands on the models and results for estimating swapping fractions) — https://github.com/MarioniLab/BarcodeSwapping2017

## Examples

```
Rscript -e 'fit <- lm(swapped_reads ~ available_reads, data=impossible_combos); writeLines(sprintf("%f", coef(fit)[2]), "swapping_fraction_estimate.txt")'
```

## Evaluation signals

- Estimated swapped-read fraction should be approximately 0.025 (2.5%), consistent with the reported HiSeq 4000 mislabelling rate; report the deviation from this reference
- The task's own reproduction target is 2.18 ± 0.08% for the Richard et al. HiSeq 4000 dataset
- Control run on the same libraries sequenced on HiSeq 2500 should yield a near-zero swapping fraction
- Regression of impossible-combination library sizes on available-to-swap reads should show a clear positive linear trend across combinations
- swapping_fraction_estimate.txt exists and contains both the fraction estimate and its uncertainty

## Limitations

- The swapping-fraction estimation details via regression gradient are deferred to Supplementary Note 3 of the paper, which may not be available in the main text
- Approximately 2.5% of reads were mislabelled on the HiSeq 4000, lower than previous reports, so older higher estimates should not be used as validation targets
- No correlation was observed between the swapped fraction of reads and the concentration of free barcode across plates, so free-barcode concentration cannot predict swapping severity

## Evidence

- [discussion] Demultiplexing allowing assignment to expected and impossible barcode combinations: "Demultiplexing was performed allowing for reads to be assigned to libraries with both expected and “impossible” barcode combinations."
- [discussion] Linear regression of observed swapped reads vs reads available for swapping: "We performed a simple linear regression on the number of observed swapped reads for each impossible combination, with respect to the number of reads available for swapping."
- [discussion] Swapping fraction from regression gradient: "We used the gradient of the fitted line to obtain an estimate of the fraction of swapped reads for this experiment"
- [discussion] Across-plate mean and SD: "The mean and standard deviation of swapping fractions were then computed across plates."
- [discussion] HiSeq 2500 negative control: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [abstract] ~2.5% reference swapping rate on HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
