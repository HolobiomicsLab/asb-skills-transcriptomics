---
name: ma-plot-quality-assessment
description: 'Use when microarray intensity data (e.g., Illumina Bead array exports
  in a tab-separated file like example_unnorm.txt) need normalization-method selection
  and QC: compare quantile vs LOESS normalization via MA plots and check for batch
  effects via sample clustering.'
when_to_use_negative:
- Input is RNA-seq read count data or another non-microarray expression modality where
  MA-plot-based microarray normalization QC does not apply (PIMENTo targets microarray
  intensity arrays)
- Data are already normalized and only downstream significance testing (e.g., samr)
  is needed — MA-plot QC is a preprocessing/normalization-stage decision tool
- The goal is automated batch-effect removal with limma rather than visual normalization-method
  comparison and QC
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: PIMENTo
  role: R package whose Preprocess Data command normalizes the data and generates
    the MA plots and dendrogram
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Programming language in which PIMENTo is built and run
- name: RStudio
  role: Open-source IDE recommended for running PIMENTo commands with autocomplete
    capability
- name: limma
  role: Provides the quantile normalization (and batch-effect removal if necessary)
    whose output is compared on the MA scale
- name: affy
  role: Provides the LOESS normalization whose output is compared on the MA scale
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:ceb370b48d69a757b2aeda4e2b5ea6b9b8054fdf4d5dec0b6f7600bb87206ea0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - affy
  - limma
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ma-plot-quality-assessment

## Summary

A PIMENTo-based QC skill that generates MA plots (log ratio M vs mean average A) for raw, quantile-normalized, and LOESS-normalized microarray data, plus a Ward's-method dendrogram, so the user can visually judge which normalization yields linear data and whether batch effects are present.

## When to use

Use when microarray intensity data (e.g., Illumina Bead array exports in a tab-separated file like example_unnorm.txt) need normalization-method selection and QC: compare quantile vs LOESS normalization via MA plots and check for batch effects via sample clustering.

## When NOT to use

- Input is RNA-seq read count data or another non-microarray expression modality where MA-plot-based microarray normalization QC does not apply (PIMENTo targets microarray intensity arrays)
- Data are already normalized and only downstream significance testing (e.g., samr) is needed — MA-plot QC is a preprocessing/normalization-stage decision tool
- The goal is automated batch-effect removal with limma rather than visual normalization-method comparison and QC

## Inputs

- Tab-separated microarray intensity file (e.g., example_unnorm.txt) with 5 descriptive text columns, gene IDs in column 1, gene symbols in column 2, and array intensities in columns 6-11
- Preprocess Data parameters: ntext=5, data.col=6:11, symbol.index=2, id.index=1
- Microarray data such as Illumina Bead array intensity measurements (example dataset accession E-MEXP-E-MTAB-2400 in Array Express)

## Outputs

- MA plots (M vs A) for raw, quantile-normalized, and LOESS-normalized data, each subplot comparing adjacent samples with the final sample compared to the first
- Dendrogram of all chosen samples computed with Euclidean distances and Ward's method
- Saved plot file ma_plots.png
- A recorded normalization-method decision (e.g., quantile normalization preferred) and a batch-effect assessment

## How to apply

Load the tab-separated input file via PIMENTo's Preprocess Data command with parameters ntext=5, data.col=6:11, symbol.index=2, id.index=1 (gene symbols in column 2, gene IDs in column 1, intensities in columns 6-11). Apply both shipped normalizations: quantile normalization via the 'limma' package and LOESS normalization via the 'affy' package. Generate a series of MA plots — graphs of log ratios (M) vs mean average scale (A) — for the raw, quantile-normalized, and LOESS-normalized data, where each subplot compares a sample with the next one in the series and the final sample is compared with the first. Also generate a dendrogram of all chosen samples using Euclidean distances and Ward's clustering to reveal clustering by date, operator, or reagent lot (batch effects). Judge which normalized data appear linear on the MA scale; for the paper's example dataset quantile normalization was judged better, and no batch effect was present as clusters aligned with sample response classes. Save the resulting figures as ma_plots.png.

## Related tools

- **PIMENTo** (R package whose Preprocess Data command normalizes the data and generates the MA plots and dendrogram) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Programming language in which PIMENTo is built and run)
- **RStudio** (Open-source IDE recommended for running PIMENTo commands with autocomplete capability)
- **limma** (Provides the quantile normalization (and batch-effect removal if necessary) whose output is compared on the MA scale)
- **affy** (Provides the LOESS normalization whose output is compared on the MA scale)

## Examples

```
install.packages("devtools"); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- After the preferred normalization (quantile for the example dataset), the data appear linear on the MA scale in the MA plots
- The dendrogram shows clusters aligning with sample response classes rather than with date, operator, or reagent lot, indicating no batch effect
- The MA plot series contains one subplot per sample pairing adjacent samples, with the final sample compared to the first
- Plots are successfully written to ma_plots.png
- Both quantile (limma) and LOESS (affy) normalized versions were produced so a side-by-side MA-scale linearity comparison is possible

## Limitations

- Normalization-method choice and batch-effect judgment are made by visual inspection of MA plots and the dendrogram, not by an automated statistic
- No version numbers are stated for the software dependencies (limma, affy, samr, R, Bioconductor), so reproducibility across versions is not guaranteed
- The promised R Shiny user interface is not yet available; interaction is command-line only
- The analysis dataset (wild-type vs villin-gp130Act small intestinal crypt expression profiles) has no accession or hosting location given beyond the Array Express deposit E-MEXP-E-MTAB-2400

## Evidence

- [methods] one can see that quantile normalization is the better of the two normalization methods for this data set, as the data appears linear on the MA scale.: [evidence span withheld — anchor status: not_found]
- [methods] PIMENTo generates a dendrogram plot of all chosen samples.: [evidence span withheld — anchor status: not_found]
