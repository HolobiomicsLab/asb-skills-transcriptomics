---
name: gene-body-coverage-data-loading
description: Use when starting the SkewC R Markdown analysis chain on gene body coverage
  output (e.g. coverage.r) and a compact per-cell percentile-coverage matrix (Coverage_means_DF)
  is needed for plotting or trim clustering.
when_to_use_negative:
- Input is already a reduced percentile-coverage matrix (e.g. an existing Coverage_means_DF
  with pmean10..pmean100 + Annotation) — do not re-subset.
- Raw per-cell BAM files have not yet been processed by 1_geneBodyCoverage.sh / geneBodyCoverage.pl,
  so no coverage.r data exists to load.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SkewC
  role: Provides the R Markdown (1-SkewC_Create_Coverage_Matrixes.Rmd) and TestData
    coverage.r sample data used to build Coverage_means_DF
  repo: https://github.com/LSBDT/SkewC.git
- name: R
  role: Language in which coverage.r is loaded and Coverage_means_DF is constructed
    by column subsetting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:e97a1f1df73805031714f1982e80768f3c351c3fe40c878d0120b25ebbfb9081
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-body-coverage-data-loading

## Summary

Loads SkewC's per-cell gene body coverage sample data (coverage.r from SkewC's TestData, 101 columns keyed by cell ID/barcode) into R and subsets it to the Coverage_means_DF data frame of ten percentile-mean columns (pmean10..pmean100) plus the 'Annotation' column, for use by downstream SkewC R Markdowns.

## When to use

Use when starting the SkewC R Markdown analysis chain on gene body coverage output (e.g. coverage.r) and a compact per-cell percentile-coverage matrix (Coverage_means_DF) is needed for plotting or trim clustering.

## When NOT to use

- Input is already a reduced percentile-coverage matrix (e.g. an existing Coverage_means_DF with pmean10..pmean100 + Annotation) — do not re-subset.
- Raw per-cell BAM files have not yet been processed by 1_geneBodyCoverage.sh / geneBodyCoverage.pl, so no coverage.r data exists to load.

## Inputs

- coverage.r (R data file with gene body coverage sample data from SkewC TestData; 101 columns, one row per cell keyed by cell ID/barcode)
- gene body coverage output produced by bin/geneBodyCoverage.pl from per-cell BAM files

## Outputs

- Coverage_means_DF data frame with 11 columns (pmean10, pmean20, pmean30, pmean40, pmean50, pmean60, pmean70, pmean80, pmean90, pmean100, Annotation)
- coverage_means_df.tsv for use by downstream SkewC plotting and clustering R Markdowns

## How to apply

Load the coverage.r file shipped with SkewC's TestData into R (via load()), producing Coverage_DF with 101 columns and one row per cell keyed by its cell ID/barcode. Construct Coverage_means_DF by subsetting Coverage_DF to retain only the ten percentile-mean coverage columns pmean10, pmean20, ..., pmean100 plus the 'Annotation' column holding the cell IDs. Verify column counts (101 for Coverage_DF, 11 for Coverage_means_DF), expected column names, and that row count equals the number of cells. Write the result to coverage_means_df.tsv so the downstream R Markdowns (SkewC_Plot_Gene_Body_Coverage, SkewC_TrimClustering, SkewC_Plot_Typical_Skewed_Coverage) can consume it in order.

## Related tools

- **SkewC** (Provides the R Markdown (1-SkewC_Create_Coverage_Matrixes.Rmd) and TestData coverage.r sample data used to build Coverage_means_DF) — https://github.com/LSBDT/SkewC.git
- **R** (Language in which coverage.r is loaded and Coverage_means_DF is constructed by column subsetting)

## Examples

```
R -e 'load("coverage.r"); Coverage_means_DF <- Coverage_DF[, c(paste0("pmean", 10*(1:10)), "Annotation")]; write.table(Coverage_means_DF, "coverage_means_df.tsv", sep="\t", quote=FALSE)'
```

## Evaluation signals

- Coverage_DF has exactly 101 columns after loading coverage.r.
- Coverage_means_DF has exactly 11 columns with the expected names: pmean10..pmean100 plus Annotation.
- Row count of Coverage_means_DF equals the number of cells (rows) in coverage.r.
- coverage_means_df.tsv exists and its contents match the 11-column schema and row count of the in-memory data frame.

## Limitations

- The R Markdowns in the SkewC RMarkdown/ folder must be run in the same order as listed, so this loading/subsetting step cannot be skipped or reordered without breaking downstream steps.
- Coverage column naming (pmean10..pmean100) is fixed; datasets producing different percentile granularity will not map to the expected 11-column schema.

## Evidence

- [results] In SkewC_Create_Coverage_Matrix.Rmd, the Coverage_means_DF data frame consists of 10 columns [pmean10...pmean100] plus the cell ID/barcode column "Annotation".: [evidence span withheld — anchor status: not_found]
- [readme] README structure of SkewC repository listing the sample coverage data: "coverage.r - sample data for SkewC Rmarkdown"
- [methods] geneBodyCoverage.pl generates the coverage data that this skill loads: "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
- [intro] Order constraint on running the SkewC R Markdowns: "This folder contains several R Markdowns and should be run in the same order as listed below:"
