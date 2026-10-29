---
name: linear-regression-swapping-fraction-estimation
description: Use when quantifying the swapped-read fraction in plate-based scRNA-seq
  libraries sequenced on the HiSeq 4000/HiSeq X/NovaSeq, where demultiplexing exposes
  impossible sample-barcode combinations to regress against.
when_to_use_negative:
- Libraries sequenced exclusively on non-patterned-flowcell instruments such as the
  HiSeq 2500, where barcode swapping is negligible and the regression signal would
  be near zero
- Droplet-based 10x Genomics data where the goal is molecule-level exclusion (shared
  UMI, cell barcode, gene) rather than estimating a plate-level swapping fraction
  — use the DropletUtils exclusion algorithm instead
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Fitting the simple linear regression of observed swapped reads on reads available
    for swapping and computing per-plate means/standard deviations
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
- name: HiSeq 4000
  role: Patterned-flowcell sequencer producing the libraries in which barcode swapping
    is quantified
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:4044fba7fff01f30991e532b649759adadb67ae1402ccf99ad54640feacc5b7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - HiSeq 4000
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# linear-regression-swapping-fraction-estimation

## Summary

Estimates the fraction of barcode-swapped reads in plate-prepared scRNA-seq data sequenced on patterned-flowcell instruments (HiSeq 4000) by regressing observed swapped reads in 'impossible' barcode combinations against reads available for swapping; the fitted gradient is the swapping fraction.

## When to use

Use when quantifying the swapped-read fraction in plate-based scRNA-seq libraries sequenced on the HiSeq 4000/HiSeq X/NovaSeq, where demultiplexing exposes impossible sample-barcode combinations to regress against.

## When NOT to use

- Libraries sequenced exclusively on non-patterned-flowcell instruments such as the HiSeq 2500, where barcode swapping is negligible and the regression signal would be near zero
- Droplet-based 10x Genomics data where the goal is molecule-level exclusion (shared UMI, cell barcode, gene) rather than estimating a plate-level swapping fraction — use the DropletUtils exclusion algorithm instead

## Inputs

- Raw FASTQ/BCL data from plate-prepared scRNA-seq libraries (e.g. ArrayExpress E-MTAB-6843, Richard et al. mouse T-cell plates)
- Sample barcode definitions for each plate/library, including expected and impossible combinations
- Optional: matched HiSeq 2500 sequencing runs of the same libraries as a negative control

## Outputs

- Estimated swapped-read fraction (regression gradient) per dataset
- Mean and standard deviation of swapping fractions across plates (e.g. swapping_fraction_estimate.txt with 2.18 ± 0.08%)
- Per-combination table of observed swapped reads vs. reads available for swapping

## How to apply

Obtain raw plate-prepared scRNA-seq data (e.g. ArrayExpress E-MTAB-6843, Richard et al. mouse T-cell plates) and demultiplex HiSeq 4000 libraries allowing assignment to both expected and 'impossible' barcode combinations. For each impossible combination, tabulate observed swapped-read counts and the corresponding number of reads available for swapping. Fit a simple linear regression of observed swapped reads on reads available for swapping; use the gradient of the fitted line as the swapped-read fraction estimate. Compute the mean and standard deviation of swapping fractions across the two plates for the uncertainty. Validate against the reported reference of approximately 2.5% mislabelled reads on the HiSeq 4000 (the task-001 replication recovered 2.18 ± 0.08%). A key control is repeating the same procedure on the same libraries sequenced on the HiSeq 2500, where swapping should be negligible.

## Related tools

- **R** (Fitting the simple linear regression of observed swapped reads on reads available for swapping and computing per-plate means/standard deviations) — https://github.com/MarioniLab/BarcodeSwapping2017
- **HiSeq 4000** (Patterned-flowcell sequencer producing the libraries in which barcode swapping is quantified)

## Examples

```
bash get_data.sh && Rscript -e 'rmarkdown::render("bcswap_supp.Rmd")'
```

## Evaluation signals

- Estimated gradient should fall near the reported ~2.5% mislabelling rate on the HiSeq 4000 (task-001 replication: 0.0218 ± 0.0008); report deviation from the 0.025 reference value
- The same procedure applied to HiSeq 2500-sequenced versions of the same libraries should yield a substantially lower swapping fraction
- Per-combination swapped-read counts should scale approximately linearly with reads available for swapping (check regression fit/residuals)
- The output file must record both the estimated fraction and its uncertainty (standard deviation across plates)
- Impossible barcode combinations used in the regression must be genuinely absent from the experimental design (never expected sample pairs)

## Limitations

- Details of the swapping-fraction estimation via the regression gradient are deferred to Supplementary Note 3, and the per-plate linear model to Supplementary Note 4; the electronic supplementary material may be unavailable in some renditions of the article
- The estimate is an experimental/instrument-level rate (~2.5%) and varies across instruments and runs; the observed no-correlation between swapped fraction and free barcode concentration limits mechanism-based adjustment
- The result depends on demultiplexing that tolerates assignment to impossible barcode combinations; overly strict demultiplexing will hide swapped reads and bias the gradient downward

## Evidence

- [discussion] Demultiplexing allowing assignment to expected and impossible barcode combinations: "Demultiplexing was performed allowing for reads to be assigned to libraries with both expected and “impossible” barcode combinations."
- [discussion] Linear regression of swapped reads on available reads: "We performed a simple linear regression on the number of observed swapped reads for each impossible combination, with respect to the number of reads available for swapping."
- [abstract] Reference swapping fraction of ~2.5% on HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [discussion] Per-plate mean and standard deviation: "The mean and standard deviation of swapping fractions were then computed across plates."
- [discussion] HiSeq 2500 negative control: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [readme] Report regeneration with R/rmarkdown: [evidence span withheld — anchor status: not_found]
