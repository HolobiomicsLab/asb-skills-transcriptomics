---
name: rnaseq-deseq2-count-modeling
description: Use when computing gene-level differential expression between exactly
  two user-defined cohorts from a parsed pdacR dataset (GUI list format with $ex,
  $sampInfo, $featInfo with SYMBOL, $metadata), where the platform type determines
  the statistical backend.
when_to_use_negative:
- More than two cohorts or a multivariate comparison is needed — the GUI comparison
  control only supports two cohorts ('multivariate comparisons will be provided in
  a future release')
- Single-cell data (e.g., the rmoffitt/scOh atlas) — the article notes the computational
  burden and memory required are beyond the scope of a responsive GUI and the atlas
  lacks a web interface
- Cross-dataset or cross-platform DE comparison — the article states aggregation was
  removed because it 'introduced more problems than it solved' and datasets cannot
  be robustly merged across platforms
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: DESeq2
  role: Statistical backend used when the experiment type is RNAseq; computes the
    A-vs-B log fold change and Benjamini-Hochberg adjusted p-values per gene
- name: limma
  role: Statistical backend used when the experiment type is Array; supplies the logFC
    and padj columns extracted into the results table
- name: pdacR
  role: R package and RShiny GUI housing the parsed datasets in data/, the differential
    expression tab, and the volcano-plot rendering
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Implementation language; pdacR wrapper functions in R/ are called via ?function
    help and run through pdacShiny()
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:5afa5ba655effb22de2043d910b7909f220431e2e62480ca6c3ca13563fb083e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - R
  - limma
  - pdacR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# rnaseq-deseq2-count-modeling

## Summary

pdacR's bounded two-cohort differential expression mechanism that routes RNAseq data to DESeq2 and microarray data to limma, producing per-gene log2 fold change (mean of cohort A / mean of cohort B) and Benjamini-Hochberg adjusted p-values for volcano-plot comparisons in the GUI.

## When to use

Use when computing gene-level differential expression between exactly two user-defined cohorts from a parsed pdacR dataset (GUI list format with $ex, $sampInfo, $featInfo with SYMBOL, $metadata), where the platform type determines the statistical backend.

## When NOT to use

- More than two cohorts or a multivariate comparison is needed — the GUI comparison control only supports two cohorts ('multivariate comparisons will be provided in a future release')
- Single-cell data (e.g., the rmoffitt/scOh atlas) — the article notes the computational burden and memory required are beyond the scope of a responsive GUI and the atlas lacks a web interface
- Cross-dataset or cross-platform DE comparison — the article states aggregation was removed because it 'introduced more problems than it solved' and datasets cannot be robustly merged across platforms

## Inputs

- Parsed pdacR dataset as a GUI list: $ex expression matrix (genes as rows, samples as columns), $sampInfo, $featInfo with SYMBOL identifiers, $metadata including the log.transformed flag
- Selected sample track defining the two cohorts (cohort A vs cohort B)
- One user-selected gene set (curated list or 'User selected genes' text-box entry, comma/space separated)

## Outputs

- Per-gene differential expression table with logFC (mean of cohort A / mean of cohort B) and padj (Benjamini-Hochberg adjusted) columns for every gene
- Volcano plot with x = log fold change, y = -log10(padj), labeling gene-set members regardless of fold change

## How to apply

Load a parsed dataset from pdacR's data/ directory in the required GUI list format ($ex expression matrix with genes as rows and samples as columns, $sampInfo, $featInfo with SYMBOL identifiers, $metadata including the log.transformed flag), then subset samples to the two cohorts defined by the selected sample track (A vs B via the comparison control). Check $metadata to decide the backend: if the experiment type is RNAseq, run DESeq2 on the subset to compute A-vs-B log fold change and Benjamini-Hochberg adjusted p-values per gene; if the experiment type is Array, run limma and extract the logFC and padj columns. Assemble a per-gene results table of log fold change and padj, flagging members of the user-selected gene set for highlighting. Compute -log10(padj) and draw a volcano plot (x = log fold change, y = -log10(padj)), labeling gene-set members that are significantly differentially expressed regardless of fold change. Only pairwise A-vs-B comparisons are supported; multivariate comparisons are not provided in the current release.

## Related tools

- **DESeq2** (Statistical backend used when the experiment type is RNAseq; computes the A-vs-B log fold change and Benjamini-Hochberg adjusted p-values per gene)
- **limma** (Statistical backend used when the experiment type is Array; supplies the logFC and padj columns extracted into the results table)
- **pdacR** (R package and RShiny GUI housing the parsed datasets in data/, the differential expression tab, and the volcano-plot rendering) — github.com/rmoffitt/pdacR
- **R** (Implementation language; pdacR wrapper functions in R/ are called via ?function help and run through pdacShiny())

## Examples

```
devtools::install_github("rmoffitt/pdacR"); pdacR::pdacShiny()
```

## Evaluation signals

- Output table contains logFC and padj columns for every gene in the input matrix
- All padj values lie in [0,1]
- Volcano plot renders with x = log fold change and y = -log10(padj), matching the module's documented output figure
- Gene-set members are labeled on the volcano plot even when significantly expressed regardless of fold change
- Correct backend routing: RNAseq experiments invoke DESeq2, Array experiments invoke limma

## Limitations

- Only two-cohort pairwise comparisons are supported; multivariate comparisons are deferred to a future release
- The provided datasets auto-select the DE method based on experimentation mode; users who override the selection must match it to the platform or results will be invalid
- Datasets span bulk RNA sequencing and microarray platforms, which cannot be robustly merged, so DE results are dataset-specific and not comparable across platforms
- Sample filtering must be applied before DE (e.g., TCGA-PAAD has 182 samples but only 150 are PDAC; filter via 'Decision' keeping only 'whitelist') or non-adenocarcinoma cases will contaminate the cohorts

## Evidence

- [readme] Our provided data sets already select the appropriate method of DE analysis based on the mode of experimentation.: "Our provided data sets already select the appropriate method of DE analysis based on the mode of experimentation."
- [readme] Checking a box in this list will remove all samples with that label from all analyses.: "Checking a box in this list will remove all samples with that label from all analyses."
- [abstract] an interface for user-friendly analysis in clustering, differential expression, survival, and dimensionality reduction: "an interface for user-friendly analysis in clustering, differential expression, survival, and dimensionality reduction"
- [methods] To filter to the correct 150, select "Decision", and then remove all variables except "whitelist".: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [discussion] we found that aggregation introduced more problems than it solved: "we found that aggregation introduced more problems than it solved"
