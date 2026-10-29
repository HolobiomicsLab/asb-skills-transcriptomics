---
name: linear-regression-correlation-analysis
description: Use when you need to estimate a per-sample swapped-read fraction from
  count data via regression gradients, or test whether a swapping rate correlates
  with a covariate such as free library barcode concentration across plates.
when_to_use_negative:
- The data are droplet-based (10x Genomics) and the goal is to remove swapped molecules
  rather than estimate plate-level swapping rates — use the DropletUtils molecule-exclusion
  algorithm instead
- The sequencing instrument is not a patterned flow-cell platform (e.g. libraries
  were run only on HiSeq 2500), where swapping at the ~2.5% HiSeq 4000 level is not
  expected
- The design is a simple two-group comparison where a correlation/regression of swapping
  fraction against a covariate is not the question being asked
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Fits the per-plate linear models and the swapping-fraction vs. free-barcode
    ratio regression
- name: BioAnalyzer Expert software
  role: Computes AUC from library-preparation traces to derive free barcode (45–70
    bp) and sequencable cDNA (400–800 bp) concentrations
- name: HiSeq 4000
  role: Patterned flow-cell sequencer whose swapping rate (~2.5%) is being estimated
    by the regression
- name: HiSeq 2500
  role: Non-patterned sequencer used as the control and as the source-cell reference
    for the per-plate linear models
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:cebc5757666ca4732cec53085d6d6cd38db70aa63fe104d412947f0901797f7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioAnalyzer Expert software
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# linear-regression-correlation-analysis

## Summary

Fits simple linear models in R to quantify barcode swapping: regressing observed swapped reads on reads available for swapping (taking the gradient as the swapping fraction), and regressing per-plate swapping fractions against free-barcode-to-cDNA ratios to test for correlation.

## When to use

Use when you need to estimate a per-sample swapped-read fraction from count data via regression gradients, or test whether a swapping rate correlates with a covariate such as free library barcode concentration across plates.

## When NOT to use

- The data are droplet-based (10x Genomics) and the goal is to remove swapped molecules rather than estimate plate-level swapping rates — use the DropletUtils molecule-exclusion algorithm instead
- The sequencing instrument is not a patterned flow-cell platform (e.g. libraries were run only on HiSeq 2500), where swapping at the ~2.5% HiSeq 4000 level is not expected
- The design is a simple two-group comparison where a correlation/regression of swapping fraction against a covariate is not the question being asked

## Inputs

- Count matrices from the Nestorowa et al. plate-based scRNA-seq dataset (NCBI GEO GSE81682), sequenced on both HiSeq 2500 and HiSeq 4000
- Demultiplexed read counts for 'impossible' barcode combinations and reads available for swapping
- BioAnalyzer Expert trace data from library preparation (AUC between 45–70 bp and 400–800 bp per plate)
- R environment (packages such as scran and Rtsne used for downstream checks)

## Outputs

- Per-plate swapping fractions with mean and standard deviation across the 16 Nestorowa plates
- Free-barcode-to-cDNA ratios per plate
- Regression coefficients and p-values for swapping fraction vs. free-barcode ratio
- Table swapping_fraction_vs_free_barcode.csv

## How to apply

After demultiplexing Nestorowa et al. (GSE81682) HiSeq 2500/HiSeq 4000 plate libraries and applying cell exclusion of shared barcodes, fit a simple linear regression in R of observed swapped reads for each impossible barcode combination against the number of reads available for swapping; the gradient of the fitted line estimates the swapping fraction (~2.5% on the HiSeq 4000). For the plate-wise analysis, fit a per-plate linear model quantifying contributions of HiSeq 2500 cells to the swapping-affected HiSeq 4000 transcriptomes, then compute the mean and standard deviation of the 16 per-plate gradients (mean 2.275 ± 0.359%). Separately, quantify free barcode from BioAnalyzer Expert traces (area under the curve 45–70 bp) and sequencable cDNA (area 400–800 bp), form the free-barcode-to-cDNA ratio per plate, and regress per-plate swapping fractions against these ratios; a slope not significantly different from zero (p = 0.129) indicates no correlation.

## Related tools

- **R** (Fits the per-plate linear models and the swapping-fraction vs. free-barcode ratio regression)
- **BioAnalyzer Expert software** (Computes AUC from library-preparation traces to derive free barcode (45–70 bp) and sequencable cDNA (400–800 bp) concentrations)
- **HiSeq 4000** (Patterned flow-cell sequencer whose swapping rate (~2.5%) is being estimated by the regression)
- **HiSeq 2500** (Non-patterned sequencer used as the control and as the source-cell reference for the per-plate linear models)

## Examples

```
Rscript -e 'fit <- lm(swapped_reads ~ available_reads, data=plate); coef(fit)[2]'  # gradient = per-plate swapping fraction; then summary(lm(swap_frac ~ free_bc_ratio))
```

## Evaluation signals

- The gradient of the fitted regression line should give a swapping fraction of approximately 2.5% on the HiSeq 4000, reproducibly lower than previously reported
- Mean per-plate swapping fraction across the 16 Nestorowa plates should be ~2.275% with standard deviation ~0.359%
- The same regression on HiSeq 2500-sequenced libraries (control) should not show comparable swapping
- The slope of swapping fraction vs. free-barcode-to-cDNA ratio should not be significantly different from zero (reported p = 0.129)
- Output CSV swapping_fraction_vs_free_barcode.csv should contain per-plate fractions, ratios, and regression coefficients/p-values

## Limitations

- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries; the fraction incorrectly discarded must be counted
- The regression-gradient approach is a whole-plate (or whole-experiment) estimate; gene-specific estimates require a different per-gene linear model (Supplementary Note 4)
- No correlation with free barcode concentration was observed, so the extent of swapping is not primarily determined by free barcode amount
- Key method details (regression gradient estimation, per-plate model) are deferred to Supplementary Notes 3–4, which may not be available in the indexed full text

## Evidence

- [discussion] gradient estimates swapping fraction: "We used the gradient of the fitted line to obtain an estimate of the fraction of swapped reads for this experiment"
- [discussion] per-plate linear model and cross-plate statistics: "For each plate, we fitted a linear model to quantify contributions of different cells in the HiSeq 2500 data to the swapping-affected transcriptomes of the HiSeq 4000 data"
- [abstract] no correlation finding: "We observed no correlation between the swapped fraction of reads and the concentration of free barcode across plates."
- [discussion] free barcode quantification from BioAnalyzer traces: "we used the BioAnalyzer Expert software on the traces collected during library preparation, considering the area under the curve between DNA lengths 45 to 70 base pairs"
- [readme] repo workflow support: "A Github repository (https://github.com/MarioniLab/BarcodeSwapping2017) contains a detailed report that expands on the analyses described herein, describing models and showing results."
