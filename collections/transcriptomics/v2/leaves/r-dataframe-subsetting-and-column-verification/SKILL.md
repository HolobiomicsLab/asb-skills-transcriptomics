---
name: r-dataframe-subsetting-and-column-verification
description: Use when preparing gene body coverage data for SkewC's plotting/trim-clustering
  R Markdowns and you need a compact per-cell percentile-coverage matrix derived from
  the full 101-column coverage.r table.
when_to_use_negative:
- Input data is not per-cell gene body coverage percentile data (e.g. raw BAM files
  or gene expression counts) — run 0_split10XbyBarcode.sh and 1_geneBodyCoverage.sh
  first
- Downstream R Markdowns expect a different schema than the 11-column pmean10..pmean100
  + Annotation layout — do not subset to fewer/more percentile columns
- Cell barcodes are unmatched between the coverage table and other SkewC inputs, which
  produces empty outputs downstream (a known barcode-mismatch failure mode)
tools:
- name: R
  role: Language in which Coverage_DF is loaded from coverage.r, subset to the 11-column
    Coverage_means_DF, and exported to TSV for the SkewC R Markdowns
  repo: https://github.com/LSBDT/SkewC.git
- name: SkewC
  role: Provides the coverage.r sample data in TestData and the R Markdown pipeline
    (1-SkewC_Create_Coverage_Matrixes.Rmd onward) that consumes the subsetted coverage
    matrix
  repo: https://github.com/LSBDT/SkewC.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:867d477eea747fae252267cef358102659b34b9d257424aa441ba9761cc3d533
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# r-dataframe-subsetting-and-column-verification

## Summary

Constructs the Coverage_means_DF data frame used by SkewC's downstream R Markdowns by subsetting the 101-column gene body coverage table (coverage.r) down to the ten percentile-mean columns (pmean10..pmean100) plus the 'Annotation' cell-ID column, then verifying the resulting 11-column schema.

## When to use

Use when preparing gene body coverage data for SkewC's plotting/trim-clustering R Markdowns and you need a compact per-cell percentile-coverage matrix derived from the full 101-column coverage.r table.

## When NOT to use

- Input data is not per-cell gene body coverage percentile data (e.g. raw BAM files or gene expression counts) — run 0_split10XbyBarcode.sh and 1_geneBodyCoverage.sh first
- Downstream R Markdowns expect a different schema than the 11-column pmean10..pmean100 + Annotation layout — do not subset to fewer/more percentile columns
- Cell barcodes are unmatched between the coverage table and other SkewC inputs, which produces empty outputs downstream (a known barcode-mismatch failure mode)

## Inputs

- coverage.r — R data file of gene body coverage with 101 columns (100 percentile positions + cell ID/barcode 'Annotation'), one row per cell, shipped in SkewC's TestData

## Outputs

- Coverage_means_DF — an R data frame with exactly 11 columns (pmean10, pmean20, ..., pmean100, Annotation)
- coverage_means_df.tsv — tab-separated file of Coverage_means_DF for downstream SkewC R Markdowns

## How to apply

Load coverage.r (the sample gene body coverage data shipped in SkewC's TestData directory) into R as Coverage_DF, a data frame with 101 columns and one row per cell keyed by its cell ID/barcode. Build Coverage_means_DF by column-subsetting Coverage_DF to keep only the ten percentile-mean coverage columns pmean10, pmean20, pmean30, pmean40, pmean50, pmean60, pmean70, pmean80, pmean90, pmean100 plus the 'Annotation' column holding the cell IDs. Verify Coverage_DF has 101 columns and Coverage_means_DF has exactly 11 columns with the expected names and one row per cell. Write Coverage_means_DF to coverage_means_df.tsv so it can be consumed by the downstream plotting and clustering R Markdowns, which the README says must be run in listed order (1-SkewC_Create_Coverage_Matrixes.Rmd, 2-SkewC_Plot_Gene_Body_Coverage.Rmd, 3-SkewC_TrimClustering.Rmd, 4-SkewC_Plot_Typical_Skewed_Coverage.Rmd).

## Related tools

- **R** (Language in which Coverage_DF is loaded from coverage.r, subset to the 11-column Coverage_means_DF, and exported to TSV for the SkewC R Markdowns) — https://github.com/LSBDT/SkewC.git
- **SkewC** (Provides the coverage.r sample data in TestData and the R Markdown pipeline (1-SkewC_Create_Coverage_Matrixes.Rmd onward) that consumes the subsetted coverage matrix) — https://github.com/LSBDT/SkewC.git

## Examples

```
load('coverage.r'); Coverage_means_DF <- Coverage_DF[, c('Annotation', paste0('pmean', seq(10, 100, 10)))]; write.table(Coverage_means_DF, 'coverage_means_df.tsv', sep='\t', quote=FALSE)
```

## Evaluation signals

- Coverage_DF loaded from coverage.r has exactly 101 columns
- Coverage_means_DF has exactly 11 columns named pmean10..pmean100 plus 'Annotation'
- Row count of Coverage_means_DF equals the number of cells (rows) in coverage.r
- coverage_means_df.tsv exists on disk and parses with the 11 expected column headers
- Each cell ID/barcode in the 'Annotation' column appears exactly once

## Limitations

- The subsetting assumes the 101-column schema of coverage.r; gene body coverage files from other pipelines may not carry the pmean10..pmean100 column naming
- Barcode mismatches between inputs (e.g. barcode.tsv and barcoded BAM) are a known SkewC failure mode that yields empty outputs downstream
- R Markdowns must be executed in the order listed in the SkewC repository or downstream steps may fail

## Evidence

- [readme] coverage.r sample data location: "└── coverage.r - sample data for SkewC Rmarkdown"
- [readme] R Markdown run order: "This folder contains several R Markdowns and should be run in the same order as listed below"
- [abstract] Trim clustering consumes the coverage matrix: "The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function."
- [intro] SkewC quality measure context: "The method relies on the measure of skewness of the gene coverage of each single cell as a quality measure."
