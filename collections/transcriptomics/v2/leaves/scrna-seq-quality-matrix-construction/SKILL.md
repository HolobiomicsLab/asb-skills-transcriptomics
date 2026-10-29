---
name: scrna-seq-quality-matrix-construction
description: Use when building the compact percentile-coverage matrix from SkewC gene
  body coverage output (coverage.r) as input for SkewC plotting, trim clustering,
  and typical-vs-skewed cell classification.
when_to_use_negative:
- Input is raw per-cell gene body coverage profile data with all 101 positional/percentile
  columns and you need the full-resolution coverage matrix, not the 11-column percentile-mean
  subset
- The task is the gene body coverage computation itself — that is performed by bin/geneBodyCoverage.pl
  via 1_geneBodyCoverage.sh on split BAM files, not by this matrix-construction step
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SkewC
  role: Provides the R Markdowns (1-SkewC_Create_Coverage_Matrixes.Rmd), the sample
    coverage.r data, and the pipeline context for the coverage matrix construction
    step
  repo: https://github.com/LSBDT/SkewC.git
- name: R
  role: Language in which the coverage matrix construction is implemented (loading
    coverage.r, subsetting to pmean10..pmean100 + Annotation)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:8ade37d40cc8f9173b02efbd7657e9a8f03d5aa4df71803ca9940edff942212d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# scrna-seq-quality-matrix-construction

## Summary

Reconstructs SkewC's coverage matrix construction step (SkewC_Create_Coverage_Matrixes.Rmd), which loads the gene body coverage sample data (coverage.r) into Coverage_DF and subsets it to the ten percentile-mean columns plus the cell-ID 'Annotation' column to form Coverage_means_DF for downstream plotting and trim clustering.

## When to use

Use when building the compact percentile-coverage matrix from SkewC gene body coverage output (coverage.r) as input for SkewC plotting, trim clustering, and typical-vs-skewed cell classification.

## When NOT to use

- Input is raw per-cell gene body coverage profile data with all 101 positional/percentile columns and you need the full-resolution coverage matrix, not the 11-column percentile-mean subset
- The task is the gene body coverage computation itself — that is performed by bin/geneBodyCoverage.pl via 1_geneBodyCoverage.sh on split BAM files, not by this matrix-construction step

## Inputs

- coverage.r — SkewC gene body coverage sample data (R data file, 101 columns, one row per cell keyed by cell ID/barcode)

## Outputs

- Coverage_means_DF — R data frame with 11 columns (pmean10, pmean20, pmean30, pmean40, pmean50, pmean60, pmean70, pmean80, pmean90, pmean100 + Annotation)
- coverage_means_df.tsv — tabular export of Coverage_means_DF for downstream SkewC R Markdowns

## How to apply

Load the coverage.r file (shipped in SkewC's TestData) into R; it contains the gene body coverage sample data as a data frame with 101 columns, one row per cell keyed by its cell ID/barcode. Construct Coverage_means_DF by subsetting Coverage_DF to retain only the ten percentile-mean coverage columns pmean10, pmean20, pmean30, pmean40, pmean50, pmean60, pmean70, pmean80, pmean90, pmean100 plus the 'Annotation' column holding the cell IDs. Verify that Coverage_DF has 101 columns and that Coverage_means_DF has exactly 11 columns with the expected names and one row per cell. Write Coverage_means_DF to coverage_means_df.tsv so it can be consumed by the downstream SkewC R Markdowns (gene body coverage plotting and SkewC_TrimClustering.Rmd), which the README says must be run in the listed order.

## Related tools

- **SkewC** (Provides the R Markdowns (1-SkewC_Create_Coverage_Matrixes.Rmd), the sample coverage.r data, and the pipeline context for the coverage matrix construction step) — https://github.com/LSBDT/SkewC.git
- **R** (Language in which the coverage matrix construction is implemented (loading coverage.r, subsetting to pmean10..pmean100 + Annotation))

## Examples

```
load("coverage.r"); Coverage_means_DF <- Coverage_DF[, c("Annotation", paste0("pmean", 10*1:10))]; write.table(Coverage_means_DF, "coverage_means_df.tsv", sep="\t", quote=FALSE)
```

## Evaluation signals

- Coverage_DF has exactly 101 columns after loading coverage.r
- Coverage_means_DF has exactly 11 columns with the expected names: pmean10, pmean20, pmean30, pmean40, pmean50, pmean60, pmean70, pmean80, pmean90, pmean100, Annotation
- Row count of Coverage_means_DF equals the number of cells (rows) in coverage.r
- coverage_means_df.tsv exists on disk and contains the 11 expected columns
- The Annotation column values match the cell IDs/barcodes present in the source coverage.r data

## Limitations

- Only the ten percentile-mean columns plus cell ID are retained; the full 101-column positional coverage resolution is discarded in Coverage_means_DF
- Downstream SkewC R Markdowns depend on this matrix, so it must be generated and exported before running the plotting and trim-clustering steps

## Evidence

- [intro] Workflow step listing: "This folder contains several R Markdowns and should be run in the same order as listed below:"
- [readme] TestData description: "coverage.r - sample data for SkewC Rmarkdown"
- [methods] Coverage computation context: "geneBodyCoverage.pl is a perl script to imitate process of RSeQC 'geneBody_coverage.py'"
