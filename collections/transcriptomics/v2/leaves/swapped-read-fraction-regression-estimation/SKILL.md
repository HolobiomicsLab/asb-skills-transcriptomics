---
name: swapped-read-fraction-regression-estimation
description: Use when raw demultiplexed scRNA-seq FASTQs from patterned flow-cell
  machines (HiSeq 4000/X/NovaSeq) are available with per-sample index barcodes, and
  you need to quantify the fraction of barcode-swapped (mislabelled) reads or validate
  that swapping is machine-specific.
when_to_use_negative:
- The data come from non-patterned flow-cell instruments (e.g. HiSeq 2500 alone) where
  swapping is expected to be near background, so the regression primarily serves as
  a control rather than a correction target
- Reads lack sample-index barcodes that permit assignment to impossible combinations
  (e.g. uniquely barcoded molecules or UMI-only designs), so no swapped-read counts
  per impossible combination can be computed
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: R
  role: Environment for fitting the simple linear regression of observed swapped reads
    against reads available for swapping and extracting the gradient
- name: HiSeq 4000
  role: Patterned flow-cell sequencer on which the swapped fraction (~2.5%) is estimated
- name: HiSeq 2500
  role: Non-patterned control sequencer used to repeat the procedure on the same libraries
    and confirm machine specificity
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:ce9465e454d8ed4f989f18ad672a03c659b3f2586720c3da75af36abe205b8c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# swapped-read-fraction-regression-estimation

## Summary

Estimates the fraction of barcode-swapped reads in Illumina single-cell RNA-seq experiments by regressing observed swapped reads for each impossible sample-index combination against the reads available for swapping, and taking the gradient as the swapping fraction; serves both to quantify HiSeq 4000 swapping (~2.5%) and as a control on HiSeq 2500 runs.

## When to use

Use when raw demultiplexed scRNA-seq FASTQs from patterned flow-cell machines (HiSeq 4000/X/NovaSeq) are available with per-sample index barcodes, and you need to quantify the fraction of barcode-swapped (mislabelled) reads or validate that swapping is machine-specific.

## When NOT to use

- The data come from non-patterned flow-cell instruments (e.g. HiSeq 2500 alone) where swapping is expected to be near background, so the regression primarily serves as a control rather than a correction target
- Reads lack sample-index barcodes that permit assignment to impossible combinations (e.g. uniquely barcoded molecules or UMI-only designs), so no swapped-read counts per impossible combination can be computed

## Inputs

- Raw demultiplexable FASTQ reads from the same libraries sequenced on HiSeq 2500 and HiSeq 4000 (e.g. ArrayExpress E-MTAB-6854 droplet-prepared mouse epithelial cells)
- Sample-index barcode sequences for all libraries, to define expected and impossible index combinations
- Reference genome (mm10) for read mapping

## Outputs

- Swapped-read fraction estimate (regression gradient) with uncertainty, e.g. 2.18 ± 0.08% (HiSeq 4000) and 0.22 ± 0.01% (HiSeq 2500 control)
- Per-impossible-combination counts of observed swapped reads and reads available for swapping
- Fitted linear regression of swapped reads vs available reads per impossible combination

## How to apply

Download the raw libraries (e.g. droplet-prepared mouse epithelial cells from ArrayExpress E-MTAB-6854, sequenced on both HiSeq 2500 and HiSeq 4000) and demultiplex allowing assignment to both expected and 'impossible' sample-index barcode combinations. Map reads to the mm10 genome and count exonic read pairs per gene for every library combination. In R, fit a simple linear regression of the number of observed swapped reads for each impossible combination against the number of reads available for swapping; the gradient of the fitted line is the swapped-read fraction estimate. Run the identical procedure on the same libraries sequenced on the HiSeq 2500 as a control, since swapping is expected to be specific to patterned flow cells.

## Related tools

- **R** (Environment for fitting the simple linear regression of observed swapped reads against reads available for swapping and extracting the gradient)
- **HiSeq 4000** (Patterned flow-cell sequencer on which the swapped fraction (~2.5%) is estimated)
- **HiSeq 2500** (Non-patterned control sequencer used to repeat the procedure on the same libraries and confirm machine specificity)

## Examples

```
Rscript -e 'fit <- lm(swapped_reads ~ available_reads, data=impossible_combos); coef(fit)[[2]]'
```

## Evaluation signals

- The regression gradient for the HiSeq 4000 run should reproduce the previously estimated swapped fraction of approximately 2.5% (2.18 ± 0.08%)
- The same procedure on the HiSeq 2500 control must yield a substantially lower swapped fraction (0.22 ± 0.01%), consistent with swapping being specific to patterned flow cells
- Mean and standard deviation of swapping fractions computed across plates should be reported, mirroring the paper's per-plate aggregation
- Impossible barcode combinations should show swapped-read counts that scale linearly with the number of reads available for swapping (goodness-of-fit of the linear model)
- Demultiplexing must have permitted assignment to both expected and impossible barcode combinations; if only expected combinations appear, the pipeline was run incorrectly

## Limitations

- Key details of the swapping-fraction estimation via the regression gradient are deferred to Supplementary Note 3, which is unavailable from the article's electronic supplementary material
- The cell exclusion strategy used downstream (removing all barcodes shared between samples) can incorrectly discard genuine cell libraries; the fraction incorrectly discarded was counted in the authors' simulation
- Estimates are specific to the instrument and run; the HiSeq 2500 control shows the regression gradient is near-background on non-patterned flow cells, so small absolute values carry limited information

## Evidence

- [discussion] Demultiplexing allowing assignment to expected and impossible barcode combinations: "Demultiplexing was performed allowing for reads to be assigned to libraries with both expected and “impossible” barcode combinations."
- [discussion] Linear regression of observed swapped reads vs reads available for swapping: "We performed a simple linear regression on the number of observed swapped reads for each impossible combination, with respect to the number of reads available for swapping."
- [discussion] Gradient of fitted line as swapped fraction estimate: "We used the gradient of the fitted line to obtain an estimate of the fraction of swapped reads for this experiment"
- [discussion] HiSeq 2500 control repeats the procedure: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [discussion] Mapping and counting workflow: "Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted."
- [discussion] Reproducible ~2.5% HiSeq 4000 estimate: "we have reproducibly estimated the fraction of barcode-swapped reads on the HiSeq 4000 to be approximately 2.5%, which is lower than previously reported1"
- [discussion] Libraries sequenced on both machines: "Libraries were sequenced on both the HiSeq 2500 and the HiSeq 4000."
