---
name: volcano-plot-generation
description: Use when you need per-gene log2 fold change and adjusted p-values for
  a two-cohort comparison (A vs B) from a pdacR-style parsed dataset, and a volcano
  plot visualizing differential expression with gene-set members labeled.
when_to_use_negative:
- You need multivariate (3+ group) comparisons — the current pdacR DE control supports
  only pairwise A-vs-B radio-button comparisons ('multivariate comparisons will be
  provided in a future release').
- You intend to merge or aggregate expression across platforms before testing — the
  article notes the inability to robustly merge datasets across platforms (bulk RNA-seq
  vs microarrays), so cross-platform pooling prior to DE is out of scope.
- Input is single-cell data — the paper restricts the responsive GUI to bulk transcriptomic
  datasets because the computational burden of scRNA is beyond a responsive GUI.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: Hosts the curated datasets and the RShiny Differential Expression tab that
    performs the dispatch and volcano plotting
  repo: https://github.com/rmoffitt/pdacR.git
- name: DESeq2
  role: 'Statistical backend for RNAseq cohorts: computes A-vs-B log fold change and
    Benjamini-Hochberg adjusted p-values per gene'
- name: limma
  role: 'Statistical backend for microarray cohorts: supplies logFC and padj columns
    per gene'
- name: shiny
  role: Web framework for the RShiny app (pdacShiny) exposing the DE comparison and
    volcano plot controls
  repo: https://github.com/rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:8a90005a09f458e16a70c77bceb852937cae8865b11c9c2dd8d4ab733821e435
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - DESeq2
  - R
  - limma
  - pdacR
  - shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# volcano-plot-generation

## Summary

Generates per-gene volcano plots from a two-cohort differential expression comparison, routing RNAseq data through DESeq2 and microarray data through limma within pdacR, and highlighting user-selected gene sets. It underpins the head-to-head PDAC cohort comparisons in the pdacR RShiny interface.

## When to use

Use when you need per-gene log2 fold change and adjusted p-values for a two-cohort comparison (A vs B) from a pdacR-style parsed dataset, and a volcano plot visualizing differential expression with gene-set members labeled.

## When NOT to use

- You need multivariate (3+ group) comparisons — the current pdacR DE control supports only pairwise A-vs-B radio-button comparisons ('multivariate comparisons will be provided in a future release').
- You intend to merge or aggregate expression across platforms before testing — the article notes the inability to robustly merge datasets across platforms (bulk RNA-seq vs microarrays), so cross-platform pooling prior to DE is out of scope.
- Input is single-cell data — the paper restricts the responsive GUI to bulk transcriptomic datasets because the computational burden of scRNA is beyond a responsive GUI.

## Inputs

- pdacR parsed dataset list ($ex expression matrix genes x samples, $sampInfo, $featInfo with SYMBOL identifiers, $metadata with log.transformed flag)
- Sample track selection defining two cohorts (A vs B comparison)
- One selected gene set (curated list or user-defined comma/space-separated genes)

## Outputs

- Per-gene results table with logFC and Benjamini-Hochberg padj columns
- Volcano plot (x = log fold change, y = -log10(padj)) with significant gene-set members labeled

## How to apply

Load a parsed pdacR dataset from data/ in the GUI list format ($ex expression matrix with genes as rows and samples as columns, $sampInfo, $featInfo with SYMBOL identifiers, $metadata including the log.transformed flag). Subset samples to the two cohorts defined by the selected sample track, then dispatch by platform: run DESeq2 on RNAseq subsets or limma on Array subsets, extracting logFC and padj columns in each case. Compute log2 fold change as mean of cohort A over mean of cohort B with Benjamini-Hochberg adjusted p-values, and flag genes belonging to the user-selected gene set (chosen via the 'gene sets to use' and 'User selected genes' controls). Plot x = log fold change and y = -log10(padj), labeling gene-set members that are significantly differentially expressed regardless of fold change.

## Related tools

- **pdacR** (Hosts the curated datasets and the RShiny Differential Expression tab that performs the dispatch and volcano plotting) — https://github.com/rmoffitt/pdacR.git
- **DESeq2** (Statistical backend for RNAseq cohorts: computes A-vs-B log fold change and Benjamini-Hochberg adjusted p-values per gene)
- **limma** (Statistical backend for microarray cohorts: supplies logFC and padj columns per gene)
- **shiny** (Web framework for the RShiny app (pdacShiny) exposing the DE comparison and volcano plot controls) — https://github.com/rmoffitt/pdacR

## Examples

```
pdacR::pdacShiny()  # then select dataset, 1 gene set, 1 Sample Track, and an A-vs-B comparison in the Differential Expression (DE) tab
```

## Evaluation signals

- Output table contains logFC and padj columns for every gene in $ex
- All padj values lie in [0,1]
- Backend matches platform: DESeq2 for experiment type RNAseq, limma for experiment type Array
- Volcano plot axes are x = log fold change and y = -log10(padj), with gene-set members labeled as in the module's documented output figure
- Gene-set members are labeled even when not extreme in fold change, consistent with 'labeling gene-set members that are significantly differentially expressed regardless of fold change'

## Limitations

- Only two-cohort (A vs B) comparisons are supported; multivariate DE is deferred to a future release
- Datasets come from mixed platforms (bulk RNA sequencing and microarrays), so no cross-platform merging or aggregation precedes the test
- Bulk data cannot fully separate cellular populations, so DE signals include tumor-microenvironment contributions
- Beware of filtering pitfalls — e.g. TCGA-PAAD has 182 samples but only 150 are histologically confirmed PDAC; filter via 'Decision' -> 'whitelist' before running DE

## Evidence

- [results] route RNAseq input to DESeq2 and microarray input to limma, returning per-gene log2 fold change (mean of cohort A / mean of cohort B) and adjusted p-values: [evidence span withheld — anchor status: not_found]
- [readme] Our provided data sets already select the appropriate method of DE analysis based on the mode of experimentation: "Our provided data sets already select the appropriate method of DE analysis based on the mode of experimentation. You may select your own in"
- [abstract] an interface for user-friendly analysis in clustering, differential expression, survival, and dimensionality reduction: "an interface for user-friendly analysis in clustering, differential expression, survival, and dimensionality reduction"
- [readme] These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots.: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [methods] TCGA-PAAD has 182 samples, but only 150 are PDAC.: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
