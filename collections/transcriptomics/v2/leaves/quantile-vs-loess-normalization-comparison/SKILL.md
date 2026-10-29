---
name: quantile-vs-loess-normalization-comparison
description: Use when raw microarray intensity data must be normalized and there is
  no a priori reason to prefer quantile or LOESS; compare both, inspect MA plots for
  linearity, and check the dendrogram for batch effects.
when_to_use_negative:
- Input is already normalized or is a processed feature/count table rather than raw
  per-sample microarray intensity columns
- Data are RNA-seq read counts rather than microarray intensities — PIMENTo's quantile/LOESS
  comparison targets microarray data such as Illumina Bead arrays and Affymetrix GeneChips
- Only one normalization method is mandated by the platform's standard preprocessing
  protocol, making a side-by-side comparison unnecessary
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: R package that runs Preprocess Data, both normalizations, MA plot and dendrogram
    generation
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: limma
  role: Provides quantile normalization (and batch-effect removal if needed)
- name: affy
  role: Provides LOESS normalization
- name: R / RStudio
  role: Runtime environment; RStudio's autocomplete eases running PIMENTo commands
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:2d7344bcacd58340221f885bdd71d234c5cfe2a835c053b7b0013abe712a4a8a
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# quantile-vs-loess-normalization-comparison

## Summary

A PIMENTo-based workflow that applies both shipped normalization methods (quantile via limma, LOESS via affy) to a raw microarray intensity table, then judges which is better by inspecting MA plots for linearity on the MA scale and a Ward-clustered dendrogram for batch effects.

## When to use

Use when raw microarray intensity data must be normalized and there is no a priori reason to prefer quantile or LOESS; compare both, inspect MA plots for linearity, and check the dendrogram for batch effects.

## When NOT to use

- Input is already normalized or is a processed feature/count table rather than raw per-sample microarray intensity columns
- Data are RNA-seq read counts rather than microarray intensities — PIMENTo's quantile/LOESS comparison targets microarray data such as Illumina Bead arrays and Affymetrix GeneChips
- Only one normalization method is mandated by the platform's standard preprocessing protocol, making a side-by-side comparison unnecessary

## Inputs

- Tab-separated raw microarray intensity file (e.g. example_unnorm.txt) with 5 descriptive text columns, gene IDs in column 1, gene symbols in column 2, and intensity data in columns 6–11
- Preprocess Data parameters: ntext=5, data.col=6:11, symbol.index=2, id.index=1

## Outputs

- MA plots (M vs A) for raw, quantile-normalized, and LOESS-normalized data (e.g. ma_plots.png)
- Dendrogram of all chosen samples from Euclidean distances with Ward's method clustering
- A decision record stating which normalization is better for the dataset and whether batch effects are present

## How to apply

Load the tab-separated intensity file (e.g. example_unnorm.txt) with PIMENTo's Preprocess Data command using ntext=5, data.col=6:11, symbol.index=2, id.index=1, since the first five columns are descriptive text, gene IDs sit in column 1 and gene symbols in column 2, and intensities occupy columns 6–11. Run both normalizations PIMENTo ships: quantile normalization through the 'limma' package and LOESS normalization through the 'affy' package. Generate MA plots (log ratios M vs mean average A) for raw, quantile-normalized, and LOESS-normalized data, where each subplot compares a sample with the next in the series and the final sample is compared back with the first. Generate a dendrogram of all chosen samples using Euclidean distances and Ward's method clustering. Decision point: choose the normalization whose MA plots appear linear on the MA scale (quantile was better for the example dataset), and read the dendrogram for clusters aligning with date, operator, or reagent lot that would signal batch effects (none were present in the example; clusters aligned with sample response classes). Save the plots, e.g. as ma_plots.png.

## Related tools

- **PIMENTo** (R package that runs Preprocess Data, both normalizations, MA plot and dendrogram generation) — https://www.github.com/MUSC-CGM/PIMENTo
- **limma** (Provides quantile normalization (and batch-effect removal if needed))
- **affy** (Provides LOESS normalization)
- **R / RStudio** (Runtime environment; RStudio's autocomplete eases running PIMENTo commands)

## Examples

```
install.packages("devtools"); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- MA plots for the chosen normalization show data that appears linear on the MA scale, whereas the alternative method shows visible non-linearity
- Dendrogram clusters align with known sample response classes rather than with date, operator, or reagent lot, indicating no batch effect
- All subplots are present: raw, quantile-normalized, and LOESS-normalized comparisons, with each sample compared to the next and the final sample compared to the first
- Preprocess Data correctly parsed the input: gene IDs from column 1 and gene symbols from column 2, intensities from columns 6–11, no text columns leaking into the intensity matrix
- Plots are saved to the expected file (e.g. ma_plots.png) for later inspection

## Limitations

- The article states no version numbers for any software dependency (limma, affy, samr, R, Bioconductor), so exact reproducibility of the normalization behavior is not guaranteed
- Judgment of which normalization 'appears linear on the MA scale' is a visual inspection, not an automated metric
- A user interface (R Shiny) was not yet available; only a command-line/R workflow exists in this release
- The example dataset analyzed is deposited in Array Express as accession E-MEXP-E-MTAB-2400, but the wild-type vs villin-gp130Act crypt expression profiles have no accession or hosting location given

## Evidence

- [methods] Quantile normalization is the better method for this dataset: [evidence span withheld — anchor status: not_found]
