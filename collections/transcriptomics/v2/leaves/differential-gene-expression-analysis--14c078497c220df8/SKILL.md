---
name: differential-gene-expression-analysis
description: Use when comparing gene-level expression between exactly two user-defined
  cohorts from a parsed PDAC expression dataset and you need per-gene logFC and padj
  values plus a volcano plot.
when_to_use_negative:
- Multivariate (more than two cohort) differential expression is requested — the app's
  comparison control only supports a single A vs B pick; multivariate comparisons
  are deferred to a future release
- Input is single-cell data — the article notes the computational burden and memory
  of scRNA datasets are beyond the scope of a responsive GUI, and the scOh atlas has
  no web interface
- Datasets need to be merged across platforms before DE — the platform notes that
  robustly merging datasets across platforms (bulk RNA-seq and microarrays) was removed
  because aggregation introduced more problems than it solved
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: pdacR
  role: R package / RShiny app whose Differential Expression (DE) tab implements the
    two-cohort dispatch, dataset loading, sample-track selection and volcano rendering
  repo: https://github.com/rmoffitt/pdacR.git
- name: DESeq2
  role: Statistical backend invoked when the experiment type is RNAseq, computing
    A-vs-B log fold change and Benjamini-Hochberg adjusted p-values per gene
- name: limma
  role: Statistical backend invoked when the experiment type is Array, from which
    the logFC and padj columns are extracted
- name: R
  role: Implementation language; wrapper functions live in R/ of the pdacR package
  repo: https://github.com/rmoffitt/pdacR.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:a65b0351505e204e92d20a0c24d296fdc355ed73edcf77f82960426ca2f987d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - DESeq2
  - R
  - limma
  - pdacR
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# differential-gene-expression-analysis

## Summary

Two-cohort, platform-conditional differential expression dispatch from the pdacR GUI: RNAseq input is routed to DESeq2 and microarray input to limma, yielding per-gene log2 fold change and Benjamini-Hochberg adjusted p-values that feed volcano-plot comparisons.

## When to use

Use when comparing gene-level expression between exactly two user-defined cohorts from a parsed PDAC expression dataset and you need per-gene logFC and padj values plus a volcano plot.

## When NOT to use

- Multivariate (more than two cohort) differential expression is requested — the app's comparison control only supports a single A vs B pick; multivariate comparisons are deferred to a future release
- Input is single-cell data — the article notes the computational burden and memory of scRNA datasets are beyond the scope of a responsive GUI, and the scOh atlas has no web interface
- Datasets need to be merged across platforms before DE — the platform notes that robustly merging datasets across platforms (bulk RNA-seq and microarrays) was removed because aggregation introduced more problems than it solved

## Inputs

- Parsed pdacR dataset: $ex expression matrix (genes as rows, samples as columns), $sampInfo, $featInfo with SYMBOL identifiers, $metadata including the log.transformed flag
- Sample track with two cohort labels defining the A vs B comparison
- One user-selected gene set (curated list or 'User selected genes' text-box entries, comma/space separated)

## Outputs

- Per-gene results table with logFC and padj columns, gene-set membership flagged
- Volcano plot (x = log fold change, y = -log10(padj)) with significant gene-set members labeled

## How to apply

Load a parsed dataset from pdacR's data/ directory in the GUI list format ($ex expression matrix with genes as rows and samples as columns, $sampInfo, $featInfo with SYMBOL identifiers, $metadata including the log.transformed flag). Subset samples to the two cohorts defined by the selected sample track (A vs B, chosen via the comparison radio buttons; multivariate comparisons are not yet supported). Route by experiment type: RNAseq goes to DESeq2, microarray/Array goes to limma — the provided datasets already select the appropriate method of DE analysis based on the mode of experimentation. Extract the A-vs-B log2 fold change (mean of cohort A / mean of cohort B) and Benjamini-Hochberg adjusted p-values, assembling a per-gene results table of logFC and padj and flagging members of the user-selected gene set (exactly 1 gene set and 1 sample track are required). Compute -log10(padj) and draw a volcano plot (x = log fold change, y = -log10(padj)), labeling gene-set members that are significantly differentially expressed regardless of fold change.

## Related tools

- **pdacR** (R package / RShiny app whose Differential Expression (DE) tab implements the two-cohort dispatch, dataset loading, sample-track selection and volcano rendering) — https://github.com/rmoffitt/pdacR.git
- **DESeq2** (Statistical backend invoked when the experiment type is RNAseq, computing A-vs-B log fold change and Benjamini-Hochberg adjusted p-values per gene)
- **limma** (Statistical backend invoked when the experiment type is Array, from which the logFC and padj columns are extracted)
- **R** (Implementation language; wrapper functions live in R/ of the pdacR package) — https://github.com/rmoffitt/pdacR.git

## Examples

```
pdacR::pdacShiny()  # then: select dataset, filter samples, pick 1 gene set + 1 Sample Track, choose A-vs-B comparison in the DE tab
```

## Evaluation signals

- Output table contains logFC and padj columns for every gene in $ex
- All padj values lie in [0,1]
- logFC equals mean of cohort A divided by mean of cohort B per gene (log2 scale)
- Volcano plot renders with axes x = log fold change and y = -log10(padj), and gene-set members that are significantly differentially expressed are labeled regardless of fold change
- Backend matches experiment type: DESeq2 for RNAseq datasets, limma for Array datasets, as pre-selected by the provided datasets

## Limitations

- Only bivariate comparisons are supported; multivariate comparisons will be provided in a future release
- DE requires exactly 1 gene set and 1 sample track to be picked
- The limma path is used for microarray data, so padj semantics depend on the backend; logFC is defined as mean of cohort A / mean of cohort B
- Non-selected samples must be removed via 'Filter samples by' first — e.g. TCGA-PAAD has 182 samples but only 150 are histologically confirmed PDAC, and failing to filter to the 'whitelist' misleads analyses

## Evidence

- [readme] Two-cohort dispatch via DESeq2/limma: "To perform DE, you must pick 1 gene set and 1 "Sample Track". In the top right corner, radio buttons will appear so you may pick your comparison (multivariate comparisons will be provided in a future"
- [readme] Platform-conditional method selection: "Our provided data sets already select the appropriate method of DE analysis based on the mode of experimentation."
- [abstract] DE is a first-class workflow in the platform: "an interface for user-friendly analysis in clustering, differential expression, survival, and dimensionality reduction"
- [methods] Only 150 of 182 TCGA-PAAD samples are PDAC: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [readme] Filtering removes mislabeled samples from all analyses: "Checking a box in this list will remove all samples with that label from all analyses. This is important to do to ensure accruate analyses."
