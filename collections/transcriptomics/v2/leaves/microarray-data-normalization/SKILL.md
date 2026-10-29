---
name: microarray-data-normalization
description: Use when raw microarray intensity data (e.g., a tab-separated file like
  example_unnorm.txt with gene IDs, symbols, and per-sample intensity columns) must
  be normalized before differential expression or background subtraction.
when_to_use_negative:
- The input is RNA-seq count data rather than microarray intensities — PIMENTo's Preprocess
  Data command expects microarray intensity columns
- The data have already been normalized; re-running quantile or LOESS normalization
  would distort downstream QC interpretations
- The goal is significance testing only — that is carried out separately with the
  'samr' package, not during normalization
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: PIMENTo
  role: R package providing the Preprocess Data command that runs normalization, MA
    plots, and dendrograms
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: limma
  role: Bioconductor package used by PIMENTo for quantile normalization (and batch
    effect removal if necessary)
- name: affy
  role: Bioconductor package used by PIMENTo for LOESS normalization
- name: R / RStudio
  role: Runtime environment; running the commands in RStudio gives autocomplete for
    PIMENTo commands
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:f38eebb8c05974315dcc3c6518944ad71cf7aec5b6dc0d134e1be484c67ae239
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - R / RStudio
  - RStudio
  - affy
  - limma
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# microarray-data-normalization

## Summary

Normalizes microarray intensity data using PIMENTo, which implements quantile normalization (via the 'limma' package) and LOESS normalization (via the 'affy' package), then judges the better method by inspecting MA plots and dendrograms for batch effects.

## When to use

Use when raw microarray intensity data (e.g., a tab-separated file like example_unnorm.txt with gene IDs, symbols, and per-sample intensity columns) must be normalized before differential expression or background subtraction.

## When NOT to use

- The input is RNA-seq count data rather than microarray intensities — PIMENTo's Preprocess Data command expects microarray intensity columns
- The data have already been normalized; re-running quantile or LOESS normalization would distort downstream QC interpretations
- The goal is significance testing only — that is carried out separately with the 'samr' package, not during normalization

## Inputs

- Tab-separated microarray intensity file (e.g., example_unnorm.txt) with five descriptive text columns, gene IDs in column 1, gene symbols in column 2, and intensity data in columns 6-11
- Parameter settings for Preprocess Data: ntext=5, data.col=6:11, symbol.index=2, id.index=1

## Outputs

- Quantile-normalized intensity matrix (limma)
- LOESS-normalized intensity matrix (affy)
- MA plots comparing raw, quantile-normalized, and LOESS-normalized data (saved as ma_plots.png)
- Dendrogram of all chosen samples built from Euclidean distances with Ward's clustering

## How to apply

Run PIMENTo's Preprocess Data command on the tab-separated input file using parameters ntext=5, data.col=6:11, symbol.index=2, id.index=1, so gene symbols (column 2) and gene IDs (column 1) are mapped onto the intensity columns (6-11). Apply both normalization methods PIMENTo ships: quantile normalization via 'limma' and LOESS normalization via 'affy'. Generate MA plots (log ratios M vs mean average A) for raw, quantile-normalized, and LOESS-normalized data — each subplot compares a sample with the next in the series and the final sample with the first — and generate a dendrogram using Euclidean distances between samples with Ward's clustering. Judge correctness by checking which normalized data appear linear on the MA scale (for the example dataset, quantile normalization was better) and by verifying the dendrogram clusters align with sample response classes rather than date, operator, or reagent lot, indicating no batch effect. Save the diagnostic plots as ma_plots.png.

## Related tools

- **PIMENTo** (R package providing the Preprocess Data command that runs normalization, MA plots, and dendrograms) — https://www.github.com/MUSC-CGM/PIMENTo
- **limma** (Bioconductor package used by PIMENTo for quantile normalization (and batch effect removal if necessary))
- **affy** (Bioconductor package used by PIMENTo for LOESS normalization)
- **R / RStudio** (Runtime environment; running the commands in RStudio gives autocomplete for PIMENTo commands)

## Examples

```
install.packages("devtools"); source("https://bioconductor.org/biocLite.R"); biocLite(c("affy", "limma", "DESeq2")); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- MA plots of quantile-normalized data appear linear on the MA scale, whereas raw or LOESS-normalized data may not (for the example dataset quantile was judged better)
- Dendrogram clusters align with sample response classes rather than with date, operator, or reagent lot, confirming no batch effect (as in Fig. 3)
- All pairwise MA subplots are produced, with each sample compared to the next and the final sample compared to the first
- Plots are saved to ma_plots.png
- Preprocess Data runs without errors on the tab-separated file with ntext=5, data.col=6:11, symbol.index=2, id.index=1

## Limitations

- No version numbers are stated for any software dependency (limma, affy, samr, R, Bioconductor)
- The R Shiny user interface is not yet available; only a command-line R interface exists until a future release
- Normalization alone does not remove systematic batch effects — limma is additionally used for removing batch effects if the dendrogram indicates clustering by processing batch rather than response class
- Installing PIMENTo requires biocLite installation of affy, limma, and DESeq2 plus devtools before the GitHub package can be loaded

## Evidence

- [methods] one can see that quantile normalization is the better of the two normalization methods for this data set, as the data appears linear on the MA scale.: [evidence span withheld — anchor status: not_found]
