---
name: per-plate-linear-model-swapping-fraction-estimation
description: Use when quantifying sample-mislabelling (barcode swapping) rates in
  plate-based scRNA-seq libraries where the same or paired libraries were sequenced
  on both a non-patterned (HiSeq 2500) and a patterned flow-cell machine (HiSeq 4000/X/NovaSeq).
when_to_use_negative:
- Data come from a single sequencing run with no paired non-patterned (HiSeq 2500)
  counterpart, so no per-plate source-vs-destination regression can be fitted
- Goal is to remove swapped molecules in 10x Genomics droplet data — use the DropletUtils
  molecule-exclusion algorithm (UMI + cell barcode + aligned gene combination) instead
  of plate-level estimation
- You need gene-specific (not whole-plate) swapping estimates; the per-plate model
  yields plate-level fractions only (gene-wise models are a distinct variant in Supplementary
  Note 4)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Used to fit the per-plate linear models and take the gradient of each fitted
    line as the swapping fraction
  repo: https://github.com/MarioniLab/BarcodeSwapping2017
- name: HiSeq 2500
  role: Non-patterned-flow-cell sequencing of the same libraries serves as the swapping-free
    control/source-cell data
- name: HiSeq 4000
  role: Patterned flow-cell sequencer on which the ~2.5% swapped-read fraction is
    estimated for each plate
- name: BioAnalyzer Expert software
  role: Quantifies free barcode (45–70 bp trace area) and sequencable cDNA (400–800
    bp area) used as regression covariates across plates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:cb1c98db6494081162d6838eb3416c2a502480b190d86ff1d86a0c2ae0f8b201
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/BarcodeSwapping2017
  tools:
  - BioAnalyzer Expert software
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-plate-linear-model-swapping-fraction-estimation

## Summary

Estimates the fraction of barcode-swapped reads per plate in plate-based scRNA-seq data by fitting, in R, a linear model per plate that quantifies the contribution of HiSeq 2500 source-cell libraries to swapping-affected HiSeq 4000 transcriptomes, taking the regression gradient as the swapping fraction. Applied to the Nestorowa et al. dataset (GSE81682) it yielded a mean swapping fraction of 2.275 ± 0.359% (~2.5%).

## When to use

Use when quantifying sample-mislabelling (barcode swapping) rates in plate-based scRNA-seq libraries where the same or paired libraries were sequenced on both a non-patterned (HiSeq 2500) and a patterned flow-cell machine (HiSeq 4000/X/NovaSeq).

## When NOT to use

- Data come from a single sequencing run with no paired non-patterned (HiSeq 2500) counterpart, so no per-plate source-vs-destination regression can be fitted
- Goal is to remove swapped molecules in 10x Genomics droplet data — use the DropletUtils molecule-exclusion algorithm (UMI + cell barcode + aligned gene combination) instead of plate-level estimation
- You need gene-specific (not whole-plate) swapping estimates; the per-plate model yields plate-level fractions only (gene-wise models are a distinct variant in Supplementary Note 4)

## Inputs

- Gene-level count matrices for plate-based Smart-seq2 scRNA-seq libraries sequenced on both HiSeq 2500 and HiSeq 4000 (e.g. Nestorowa et al., GSE81682; reads mapped to mm10 with exonic read-pair counts)
- Per-plate cell barcode assignments for HiSeq 2500 and HiSeq 4000 samples
- Optionally, BioAnalyzer Expert traces from library preparation (for free-barcode/cDNA covariate analysis)

## Outputs

- Per-plate swapping fraction estimates (regression gradients)
- Mean and standard deviation of swapping fractions across plates (e.g. 2.275 ± 0.359%)
- Regression coefficients and p-values if covariates (e.g. free-barcode-to-cDNA ratio) are tested
- Tabular results file (e.g. swapping_fraction_vs_free_barcode.csv)

## How to apply

Obtain count matrices (e.g. Nestorowa et al. from NCBI GEO GSE81682, plate libraries sequenced on both HiSeq 2500 and HiSeq 4000) and apply cell exclusion by removing all cell barcodes shared between samples so genuine cells are not confused with swapped libraries. For each of the 16 plates, fit a linear model in R quantifying the contributions of different HiSeq 2500 cells to the swapping-affected HiSeq 4000 transcriptomes, and take the gradient of each fitted model as the per-plate swapping fraction. Compute the mean and standard deviation across plates as the overall swapping estimate; the paper's result was approximately 2.5% of reads mislabelled on the HiSeq 4000, lower than previous reports. The rationale is that swapped reads for an 'impossible' barcode combination grow linearly with the number of reads available for swapping, so the fitted regression gradient estimates the swapped fraction. Optionally regress the per-plate swapping fractions against a covariate such as the free-barcode-to-cDNA ratio (BioAnalyzer Expert trace areas 45–70 bp vs 400–800 bp) to test drivers of swapping; in the paper this slope was not significantly different from zero (p = 0.129).

## Related tools

- **R** (Used to fit the per-plate linear models and take the gradient of each fitted line as the swapping fraction) — https://github.com/MarioniLab/BarcodeSwapping2017
- **HiSeq 2500** (Non-patterned-flow-cell sequencing of the same libraries serves as the swapping-free control/source-cell data)
- **HiSeq 4000** (Patterned flow-cell sequencer on which the ~2.5% swapped-read fraction is estimated for each plate)
- **BioAnalyzer Expert software** (Quantifies free barcode (45–70 bp trace area) and sequencable cDNA (400–800 bp area) used as regression covariates across plates)

## Examples

```
Rscript -e 'fit <- lm(n_swapped ~ n_available, data = plate_dat); swap_fraction <- coef(fit)[2]'
```

## Evaluation signals

- Per-plate gradients across the 16 Nestorowa plates should average approximately 2.275% with standard deviation about 0.359% (i.e. ~2.5% of reads mislabelled on the HiSeq 4000)
- Repeating the procedure on the same libraries sequenced on the HiSeq 2500 (control) should yield near-zero swapping fractions
- Per-plate swapping fraction must be the gradient (second coefficient) of the fitted linear model, not an intercept or per-read count
- Regression of swapping fractions against free-barcode-to-cDNA ratios should give a slope not significantly different from zero (paper: p = 0.129)
- Cell exclusion must have removed all barcodes shared between samples before model fitting

## Limitations

- Cell exclusion (removing all barcodes shared between samples) can incorrectly discard genuine cell libraries; the simulation counted the fraction incorrectly discarded
- Full model specification (including the gene-wise variant) is deferred to Supplementary Note 4, which may be unavailable in the main-text-only source
- The ~2.5% estimate is specific to patterned flow-cell instruments (HiSeq 4000/X/NovaSeq); no excessive swapping is expected on HiSeq 2500
- No correlation between swapped fraction and free barcode concentration was observed, so free barcode amount is not a validated predictor of swapping rate

## Evidence

- [discussion] Per-plate linear model definition: "For each plate, we fitted a linear model to quantify contributions of different cells in the HiSeq 2500 data to the swapping-affected transcriptomes of the HiSeq 4000 data"
- [discussion] Mean and SD across plates: "The mean and standard deviation of swapping fractions were then computed across plates."
- [abstract] Overall swapped-fraction finding: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [discussion] Cell exclusion filter step: "We applied cell exclusion by removing all barcodes that were shared between samples"
- [discussion] Regression rationale (gradient as fraction): "We performed a simple linear regression on the number of observed swapped reads for each impossible combination, with respect to the number of reads available for swapping."
- [readme] Supplementary report repository: [evidence span withheld — anchor status: external_doc]
