---
name: differential-detection-analysis
description: Use when the goal is to detect subpopulation-specific changes in the
  fraction of cells expressing a gene (differential detection) across conditions from
  multi-sample scRNA-seq data, e.g. with muscat's pbDD.
when_to_use_negative:
- The analysis goal is differential expression of mean expression levels (DS) rather
  than the fraction of detecting cells; use standard pbDS on sum-aggregated pseudobulks
  instead
- Data lack the multi-sample structure (multiple samples per group) required for sample-level
  pseudobulk inference; cell-level methods were not designed for the multi-group multi-sample
  problem and the DD workflow assumes it
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: muscat
  role: Provides aggregateData (fun='num.detected') to build binarized-detection pseudobulks
    and pbDD (or pbDS with method='DD') to test differential detection, plus stagewise
    DS+DD combination
  repo: github.com/HelenaLC/muscat
- name: R
  role: Runtime environment for the muscat Bioconductor package and the DD workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:f490dcdd18b296b81b3ea6584fad282fe9d377f0c5da8a1b62fb5a52a7da66b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - muscat
  - scater
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# differential-detection-analysis

## Summary

Performs differential detection (DD) analysis in multi-sample, multi-group, multi-subpopulation scRNA-seq data by testing for shifts in the proportion of cells with detected (nonzero) gene expression across conditions, using muscat's pbDD on num.detected pseudobulks.

## When to use

Use when the goal is to detect subpopulation-specific changes in the fraction of cells expressing a gene (differential detection) across conditions from multi-sample scRNA-seq data, e.g. with muscat's pbDD.

## When NOT to use

- The analysis goal is differential expression of mean expression levels (DS) rather than the fraction of detecting cells; use standard pbDS on sum-aggregated pseudobulks instead
- Data lack the multi-sample structure (multiple samples per group) required for sample-level pseudobulk inference; cell-level methods were not designed for the multi-group multi-sample problem and the DD workflow assumes it

## Inputs

- SingleCellExperiment object with colData columns sample_id, group_id, and cluster_id (e.g. muscat's example_sce Kang IFN-β PBMC data)
- Raw counts assay (used with fun='num.detected' for binarized detection pseudobulks)

## Outputs

- DD results table per gene and cluster_id with p-values and adjusted p-values for shifts in detection proportions
- Per-gene/per-cluster results table (e.g. dd_ds_stagewise_results.csv) with p_val, p_adj.loc, DD statistics, and stagewise rejections when combined with DS analysis

## How to apply

From a SingleCellExperiment with sample_id, group_id, and cluster_id metadata columns, first aggregate counts into cluster-sample pseudobulks with muscat::aggregateData setting fun='num.detected', which sums binarized detection counts (cells with nonzero expression) rather than total counts, yielding detection-count pseudobulks per cell subpopulation. Then run muscat::pbDD(pb) (or equivalently pbDS(pb, method='DD')) on these pseudobulks to test for shifts in the proportion of detecting cells across conditions per cell type. DD results can be used standalone, or combined with differential state (DS) results via muscat's stagewise two-stage DD+DS testing, which reweights/combines DD and DS hypotheses to increase statistical power and provide better FDR control than either analysis alone. Compare the number of stagewise rejections at FDR 0.05 against DS-only calls (p_adj.loc < 0.05) to verify the expected power gain.

## Related tools

- **muscat** (Provides aggregateData (fun='num.detected') to build binarized-detection pseudobulks and pbDD (or pbDS with method='DD') to test differential detection, plus stagewise DS+DD combination) — github.com/HelenaLC/muscat
- **R** (Runtime environment for the muscat Bioconductor package and the DD workflow)

## Examples

```
pb <- aggregateData(sce, assay="counts", fun="num.detected", by=c("cluster_id","sample_id")); dd <- pbDD(pb)
```

## Evaluation signals

- Output table exists and contains gene and cluster_id fields plus DD and DS-only adjusted p-value (p_adj.loc) columns
- The stagewise DD+DS procedure yields at least as many rejections at FDR 0.05 as the DS-only calls (p_adj.loc < 0.05), consistent with the vignette's claim of effective DD/DS combination
- Pseudobulks were built with fun='num.detected' (binarized detection counts), not fun='sum' — verify the aggregation function used
- Adjusted p-values across genes/clusters show FDR control at the 0.05 level

## Limitations

- DD and DS analyze different aspects of the same data; the two-stage stagewise procedure is needed to control FDR when combining hypotheses, and using individual DD or DS results alone gives lower power
- Aggregation-based pseudobulk methods can show a strong bias in statistical power toward larger cell populations, with decreased power for rarer populations
- For hypotheses where a cell type contributes little, covariate-driven approaches (as in related bbhw binning) set the covariate to NA, leaving those hypotheses unweighted

## Evidence

- [methods] muscat can be used to perform differrential detection (DD) analyses in multi-sample, multi-group, multi-(cell-)subpopulation scRNA-seq data: "`muscat` can be used to perform differrential detection (DD) analyses in multi-sample, multi-group, multi-(cell-)subpopulation scRNA-seq data"
- [methods] we show how DD and differential state (DS) analysis results on the same data can be effectively combined: "we show how DD and differential state (DS) analysis results on the same data can be effectively combined"
- [readme] README: num.detected aggregation and pbDD invocation: "pb <- aggregateData(sce, 
    assay = "counts", 
    fun = "num.detected", # !!!
    by = c("cluster_id", "sample_id"))"
- [readme] README: test for differential detection: "# test for differential detection
dd <- pbDD(pb) # or..
dd <- pbDS(pb, method = "DD")"
- [readme] README: DD workflows for multi-sample single-cell RNA-seq data reference: "`muscat` also supports testing for differential detection as proposed in 

> Gilis J, Perin L, Malfait M, Crowell HL, Van den"
