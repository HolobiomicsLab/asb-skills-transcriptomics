---
name: differential-expression-fold-change-calculation
description: Use when you have a SAM significance output (samr-based Significance
  Analysis of Microarrays) plus a quantile-normalized expression matrix and need heatmaps
  of top-ranked gene lists that cluster samples by response class.
when_to_use_negative:
- Input expression data has not been normalized — run PIMENTo's Preprocess Data command
  with quantile normalization first
- Genes have not yet been tested for significance — SAM testing with the samr package
  must precede heatmap generation on ranked gene lists
- Data is RNA-seq count data rather than microarray expression data; this PIMENTo
  workflow targets DNA microarray experiments such as Illumina Bead arrays
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: Provides the Create Heatmap command that generates sample-clustered heatmaps
    of top SAM-ranked genes and per-gene fold-change tables
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Language in which PIMENTo is built and in which the Create Heatmap commands
    are run
- name: RStudio
  role: IDE used to run PIMENTo R commands with autocomplete capability
- name: Bioconductor
  role: Source of open-source packages (e.g., samr, limma) leveraged by PIMENTo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:fdf1e4c776a41bd5bd21e33d088fe3fc89707371460c72a74df9715c05c803b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioconductor
  - PIMENTo
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# differential-expression-fold-change-calculation

## Summary

Generate heatmaps of top SAM-ranked significant genes (top 50/100/150) from microarray data using the PIMENTo Create Heatmap command, clustering samples (e.g., WT vs. villin-gp130Act response classes) and exporting per-gene fold-change tables to validate class separation.

## When to use

Use when you have a SAM significance output (samr-based Significance Analysis of Microarrays) plus a quantile-normalized expression matrix and need heatmaps of top-ranked gene lists that cluster samples by response class.

## When NOT to use

- Input expression data has not been normalized — run PIMENTo's Preprocess Data command with quantile normalization first
- Genes have not yet been tested for significance — SAM testing with the samr package must precede heatmap generation on ranked gene lists
- Data is RNA-seq count data rather than microarray expression data; this PIMENTo workflow targets DNA microarray experiments such as Illumina Bead arrays

## Inputs

- SAM significance output (samr-based Significance Analysis of Microarrays results with significance scores)
- Quantile-normalized expression matrix forwarded from the PIMENTo Preprocess Data step
- Gene-list input files: 'heatmap_input/top 50.txt', 'heatmap_input/top 100.txt', 'heatmap_input/top 150.txt'

## Outputs

- Top-50-gene heatmap exported in TIFF, PDF, Postscript, and FIG formats into heatmap_output
- Per-gene fold-change tables as CSV files in heatmap_output
- Column dendrogram clustering samples by response class (WT vs. villin-gp130Act)

## How to apply

Load the SAM significance output produced by the upstream samr-based Significance Analysis of Microarrays step and rank genes by significance score. Write the top 50, top 100, and top 150 ranked gene lists to 'heatmap_input/top 50.txt', 'heatmap_input/top 100.txt', and 'heatmap_input/top 150.txt'. Run the PIMENTo Create Heatmap command in R via RStudio on the SAM output together with the quantile-normalized expression matrix forwarded from the Preprocess Data step (quantile normalization was determined to be the better method for this data set). PIMENTo generates a dendrogram plot of all chosen samples, with clustering by response class. Export the top-50-gene heatmap in TIFF, PDF, Postscript, and FIG formats into heatmap_output, and write the per-gene fold-change tables as CSV files. Validation requires confirming heatmap_output contains the top-50 heatmap in all four graphics formats plus the fold-change CSVs, and that the column dendrogram separates WT from villin-gp130Act samples.

## Related tools

- **PIMENTo** (Provides the Create Heatmap command that generates sample-clustered heatmaps of top SAM-ranked genes and per-gene fold-change tables) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Language in which PIMENTo is built and in which the Create Heatmap commands are run)
- **RStudio** (IDE used to run PIMENTo R commands with autocomplete capability)
- **Bioconductor** (Source of open-source packages (e.g., samr, limma) leveraged by PIMENTo)

## Examples

```
install.packages("devtools"); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- heatmap_output contains the top-50 heatmap in all four graphics formats (TIFF, PDF, Postscript, FIG)
- heatmap_output contains the per-gene fold-change CSV files
- Samples in the top-50-gene heatmap separate according to response class (WT vs. villin-gp130Act), as indicated by the column dendrogram
- Gene-list input files top 50.txt, top 100.txt, and top 150.txt exist in heatmap_input and contain genes ranked by SAM significance score

## Limitations

- No version numbers are stated for any software dependency (limma, affy, samr, R, Bioconductor)
- The samr package citation appears as an incomplete placeholder '[ref]' in the article
- No R Shiny user interface exists yet; commands must be run in R/RStudio — a future release will utilize R Shiny
- The analysis dataset (wild-type and villin-gp130Act small intestinal crypt expression profiles) has no accession or hosting location given beyond the Array Express accession E-MEXP-E-MTAB-2400 for the chapter's microarray data

## Evidence

- [methods] PIMENTo generates a dendrogram plot of all chosen samples.: [evidence span withheld — anchor status: not_found]
- [discussion] SAM testing via samr package: [evidence span withheld — anchor status: not_found]
- [readme] README package description: "PIMENTo is an R package to normalize, analyze, and visualize microarray data"
- [methods] Quantile normalization best for this dataset: [evidence span withheld — anchor status: not_found]
