---
name: gene-expression-matrix-export
description: Use when a cleaned, batch-corrected gene-by-sample matrix of the MDI-induced
  3T3-L1 microarray data must be written to disk for downstream differential expression
  or gene set enrichment analysis.
when_to_use_negative:
- Input is raw probe-level intensities that have not been collapsed to gene symbols
  via collapseRows (upstream step required first).
- Data are RNA-seq counts rather than microarray intensities — use the curatedAdipoRNA
  workflow instead.
- Datasets were excluded for missing probe intensities or missing probe annotations
  (one and two studies, respectively) and should not be merged into the export.
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: log2-transformed gene intensities are normalized across arrays and studies
    with limma before export
- name: sva
  role: parametric empirical Bayes (ComBat-style) removal of data-series/platform
    batch effects on the normalized matrix prior to export
- name: curatedAdipoArray
  role: source of the curated SummarizedExperiment data and homogenized sample metadata
    accessed via ExperimentHub
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: SummarizedExperiment
  role: container format for the clean and processed curated data from which expression
    values and metadata are extracted for export
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_004
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:f115ce8c5e6337cfc381f163d4eed5904b59adba403eaaaf6ccb664cfb9db846
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - GEOquery
  - R/Bioconductor
  - SummarizedExperiment
  - collapseRows
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# gene-expression-matrix-export

## Summary

Exports the curated 3T3-L1 gene-level expression matrix — after low-intensity/missing-gene filtering, log2 transformation, limma normalization across arrays, and sva batch removal — into reusable tabular artifacts (normalized_gene_expression.tsv, batch_corrected_matrix.rds) that feed downstream differential expression and gene set enrichment.

## When to use

Use when a cleaned, batch-corrected gene-by-sample matrix of the MDI-induced 3T3-L1 microarray data must be written to disk for downstream differential expression or gene set enrichment analysis.

## When NOT to use

- Input is raw probe-level intensities that have not been collapsed to gene symbols via collapseRows (upstream step required first).
- Data are RNA-seq counts rather than microarray intensities — use the curatedAdipoRNA workflow instead.
- Datasets were excluded for missing probe intensities or missing probe annotations (one and two studies, respectively) and should not be merged into the export.

## Inputs

- gene-level collapsed intensity matrix from collapseRows MaxMean (gene_level_expression.csv)
- curatedAdipoArray SummarizedExperiment sample metadata via ExperimentHub
- batch covariate of study-of-origin factors (data series and platform)

## Outputs

- normalized_gene_expression.tsv (tab-separated gene-by-sample matrix, no missing/low-intensity genes)
- batch_corrected_matrix.rds (batch-corrected expression matrix, same dimensions and gene/sample order as input)

## How to apply

Start from the gene-level collapsed intensity matrix produced by the collapseRows (MaxMean) step and the curated sample metadata from the curatedAdipoArray SummarizedExperiment object accessed via ExperimentHub. Remove genes with missing values or intensities below 10, then log2-transform and normalize the intensities across arrays and studies using limma so distributions are comparable across the curated datasets. Next, assemble a batch covariate from the known study-of-origin factors (data series and platform) and apply the parametric empirical Bayes batch-removal procedure in sva (ComBat-style adjustment), preserving biological signal. Write the normalized matrix to normalized_gene_expression.tsv (tab-separated, gene symbols as row identifiers) and the batch-corrected matrix to batch_corrected_matrix.rds, checking dimensions, gene/sample ordering, and batch homogenization before release.

## Related tools

- **limma** (log2-transformed gene intensities are normalized across arrays and studies with limma before export)
- **sva** (parametric empirical Bayes (ComBat-style) removal of data-series/platform batch effects on the normalized matrix prior to export)
- **curatedAdipoArray** (source of the curated SummarizedExperiment data and homogenized sample metadata accessed via ExperimentHub) — https://github.com/MahShaaban/curatedAdipoArray
- **SummarizedExperiment** (container format for the clean and processed curated data from which expression values and metadata are extracted for export)

## Examples

```
write.table(norm_expr, "normalized_gene_expression.tsv", sep="\t", quote=FALSE)
```

## Evaluation signals

- normalized_gene_expression.tsv exists with expected tab-separated format and contains no missing or low-intensity (<10) genes
- batch_corrected_matrix.rds has the same dimensions and gene/sample order as the input normalized matrix
- intensity distributions are similar across arrays/studies after limma normalization
- batch distributions across data series and platforms are homogenized after sva adjustment
- published biological observations (differentiation time course and perturbation responses) remain reproducible after correction

## Limitations

- Datasets with missing probe intensities or missing probe annotations were excluded from the curated resource, so those series cannot be exported.
- Batch correction is limited to the known batch defined by data series and platform; unknown batch effects are not addressed by this parametric empirical Bayes adjustment.

## Evidence

- [intro] Filtering, transformation and normalization: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [intro] Batch removal: "the known batch effects (data series and platform) were removed using `sva`."
- [intro] Downstream use case: "The data contained in the package can be used in any number of downstream analysis such as differential expression and gene set enrichment."
- [readme] Data access: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
