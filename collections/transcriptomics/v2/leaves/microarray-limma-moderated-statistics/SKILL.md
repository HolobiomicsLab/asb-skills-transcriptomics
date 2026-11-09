---
name: microarray-limma-moderated-statistics
description: Use when the selected pdacR dataset's experiment type is Array (microarray)
  and a two-cohort A-vs-B differential expression comparison is requested via a sample
  track and comparison control.
when_to_use_negative:
- The dataset's experiment type is RNAseq — the dispatch routes to DESeq2 for count-based
  A-vs-B testing instead of limma
- More than two cohorts are needed — pdacR currently supports only pairwise A-vs-B
  comparisons ('multivariate comparisons will be provided in a future release')
- Input is single-cell data — pdacR's datasets are bulk analyses and the scOh single-cell
  atlas has no web interface or GUI DE workflow
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: limma
  role: Statistical backend run on microarray (Array) datasets to produce per-gene
    logFC and Benjamini-Hochberg adjusted p-values
- name: pdacR
  role: R package / Shiny GUI that stores parsed datasets, dispatches Array input
    to limma, assembles the logFC/padj table, and renders the volcano plot
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Language in which pdacR and the limma wrapper functions are implemented
  repo: github.com/rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:008f182f06961eca1545a580f76b0f8166fdc40b44ed2acd8a1d5bcbb3f78fae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - limma
  - pdacR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# microarray-limma-moderated-statistics

## Summary

The microarray branch of pdacR's differential-expression dispatch: when a curated PDAC dataset is flagged as Array-platform, limma is run on the two user-defined cohorts to extract per-gene logFC and Benjamini-Hochberg adjusted p-values (padj), which feed the head-to-head volcano plots in the GUI.

## When to use

Use when the selected pdacR dataset's experiment type is Array (microarray) and a two-cohort A-vs-B differential expression comparison is requested via a sample track and comparison control.

## When NOT to use

- The dataset's experiment type is RNAseq — the dispatch routes to DESeq2 for count-based A-vs-B testing instead of limma
- More than two cohorts are needed — pdacR currently supports only pairwise A-vs-B comparisons ('multivariate comparisons will be provided in a future release')
- Input is single-cell data — pdacR's datasets are bulk analyses and the scOh single-cell atlas has no web interface or GUI DE workflow

## Inputs

- Parsed pdacR dataset list: $ex expression matrix (genes x samples), $sampInfo, $featInfo with SYMBOL identifiers, $metadata with log.transformed flag
- Experiment-type flag indicating microarray (Array) rather than RNAseq
- Two cohort labels selected via a Sample Track and the A-vs-B comparison radio control
- One selected gene set to highlight in the volcano plot

## Outputs

- Per-gene results table with logFC and padj columns for every gene
- Volcano plot with x = log fold change, y = -log10(padj), and gene-set members labeled when significantly differentially expressed

## How to apply

Load a parsed dataset from pdacR's data/ directory in the GUI list format ($ex expression matrix with genes as rows and samples as columns, $sampInfo, $featInfo with SYMBOL identifiers, $metadata including the log.transformed flag). Subset samples to the two cohorts defined by the selected sample track (A vs B via the comparison radio buttons). Because the dataset's experiment type is Array, route to limma rather than DESeq2 — pdacR's provided datasets already pre-select the appropriate DE backend by mode of experimentation. Extract the logFC and padj columns (mean of cohort A / mean of cohort B; Benjamini-Hochberg adjusted p-values) into a per-gene results table, flagging members of the user-selected gene set. Compute -log10(padj) and render a volcano plot (x = logFC, y = -log10(padj)), labeling gene-set members that are significant regardless of fold change.

## Related tools

- **limma** (Statistical backend run on microarray (Array) datasets to produce per-gene logFC and Benjamini-Hochberg adjusted p-values)
- **pdacR** (R package / Shiny GUI that stores parsed datasets, dispatches Array input to limma, assembles the logFC/padj table, and renders the volcano plot) — github.com/rmoffitt/pdacR
- **R** (Language in which pdacR and the limma wrapper functions are implemented) — github.com/rmoffitt/pdacR

## Examples

```
devtools::install_github("rmoffitt/pdacR"); pdacR::pdacShiny()
```

## Evaluation signals

- Output table contains logFC and padj columns for every gene in $ex
- All padj values lie in [0, 1]
- Volcano plot renders with x = log fold change and y = -log10(padj)
- Gene-set members that are significantly differentially expressed are labeled on the volcano plot, matching the module's documented output figure
- The correct backend was used: limma was invoked only because the experiment type is Array (RNAseq subsets would have routed to DESeq2)

## Limitations

- Only two-cohort comparisons are supported; multivariate comparisons are a stated future release in pdacR's DE tab
- Mixing platforms (bulk RNA-seq and microarrays) is a known confound — the article notes this 'adds another layer of complexity to aggregation and batch correction', which is why datasets are analyzed per-platform rather than merged
- The DE backend is auto-selected by the provided datasets' mode of experimentation; users overriding it on incompatible data risk invalid statistics

## Evidence

- [abstract] an interface for user-friendly analysis in clustering, differential expression, survival, and dimensionality reduction: "an interface for user-friendly analysis in clustering, differential expression, survival, and dimensionality reduction"
- [discussion] our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction: "our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction"
- [readme] Our provided data sets already select the appropriate method of DE analysis based on the mode of experimentation.: "Our provided data sets already select the appropriate method of DE analysis based on the mode of experimentation."
