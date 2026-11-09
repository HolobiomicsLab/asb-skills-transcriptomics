---
name: longitudinal-scrnaseq-differential-expression
description: Use when you have scRNA-seq count matrices from two or more timepoints,
  a metadata table with a binary Group factor and a Time variable, and you need genes
  whose expression trends over time differ between strata (tDEGs).
when_to_use_negative:
- You expect nonlinear temporal changes in expression — the article states 'the current
  GLM framework can only detect linear effect, i.e., the linear change of gene expression
  by time or the interaction of time and condition'
- The data are cross-sectional (a single time point) with no Time-by-strata structure
  to test — use standard DE methods like DESeq2 or MAST instead
- The Group variable is multi-class rather than the binary strata factor the metadata
  check requires
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: R package providing the FindTemporalDEGs function that fits the per-gene negative
    binomial GLM with Time, Group, and interaction terms and performs Bonferroni-corrected
    tDEG calling
  repo: https://github.com/yizhuo-wang/CASi
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:d59a1a63510236c27170fd796c6165a9562fb97ef899a10d209efa800b89a817
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yizhuo-wang/CASi
  tools:
  - CASi
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# longitudinal-scrnaseq-differential-expression

## Summary

Identifies temporal differentially expressed genes (tDEGs) in multi-timepoint scRNA-seq data by fitting a per-gene negative binomial GLM (log link) with Time, Group, and Time-by-Group interaction covariates, followed by Bonferroni correction. This is CASi's FindTemporalDEGs mechanism for detecting genes with divergent time-trending behavior across strata.

## When to use

Use when you have scRNA-seq count matrices from two or more timepoints, a metadata table with a binary Group factor and a Time variable, and you need genes whose expression trends over time differ between strata (tDEGs).

## When NOT to use

- You expect nonlinear temporal changes in expression — the article states 'the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition'
- The data are cross-sectional (a single time point) with no Time-by-strata structure to test — use standard DE methods like DESeq2 or MAST instead
- The Group variable is multi-class rather than the binary strata factor the metadata check requires

## Inputs

- CASiList object holding gene expression count matrices for the initial timepoint (t0) and subsequent timepoints (t1), e.g. CASiList_tDEG_demo
- metadata table with a binary Group factor (e.g., response status) and a Time variable, with one row per cell across both timepoint subsets, e.g. metadata_tDEG_demo
- Gene expression matrix of the initial data point/t0 and following data points/t1

## Outputs

- tdeg_results.csv — per-gene table with columns: gene, coefficients, raw p-value, Bonferroni-adjusted p-value, tDEG call (genes with p < 0.05)
- Set of tDEG genes identified by the Time and Time-by-Group interaction coefficients

## How to apply

Load the packaged demo objects with data(CASiList_tDEG_demo) and data(metadata_tDEG_demo), verifying the metadata contains a binary Group factor and a Time variable whose row count equals the total number of cells in both time-point subsets. Filter genes by detection rate, keeping only genes expressed in at least pct.cutoff = 0.3 of cells. For each retained gene, fit a negative binomial GLM with a log link via CASi's FindTemporalDEGs, estimating coefficients and the dispersion parameter by maximum likelihood with Time, Group, and the Time-by-Group interaction as independent variables. Extract the p-value for the time (and interaction) coefficient of each gene, apply a Bonferroni correction across all tested genes, and call tDEGs with p.cutoff = 0.05.

## Related tools

- **CASi** (R package providing the FindTemporalDEGs function that fits the per-gene negative binomial GLM with Time, Group, and interaction terms and performs Bonferroni-corrected tDEG calling) — https://github.com/yizhuo-wang/CASi

## Examples

```
require(devtools); devtools::install_github("yizhuo-wang/CASi"); data(CASiList_tDEG_demo); data(metadata_tDEG_demo); FindTemporalDEGs(CASiList_tDEG_demo, metadata_tDEG_demo, pct.cutoff = 0.3, p.cutoff = 0.05)
```

## Evaluation signals

- tdeg_results.csv exists and contains one row per filtered gene with a p-value field
- Every gene flagged as a tDEG has a (Bonferroni-adjusted) p value smaller than 0.05 per the FindTemporalDEGs criterion
- The input metadata contains a binary Group factor and a Time variable whose row count equals the total number of cells in both time-point subsets
- Genes filtered out are exactly those expressed in fewer than pct.cutoff = 0.3 of cells
- Fitted models include a maximum-likelihood-estimated dispersion parameter alongside the Time, Group, and Time-by-Group coefficients

## Limitations

- The GLM framework only detects linear effects — the linear change of gene expression by time or the interaction of time and condition — and misses nonlinear temporal changes
- Bonferroni correction across all tested genes may be conservative for large gene sets
- Samples from the same subject are likely to be correlated across timepoints, and the correlation structure may not be fully modeled by the per-gene GLM
- CASi requires TensorFlow setup and the keras package to be available before use

## Evidence

- [readme] CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation"
- [methods] Below are the input parameters of the FindTemporalDEGs function: "Below are the input parameters of the FindTemporalDEGs function"
- [discussion] the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
- [abstract] identification of temporal differentially expressed genes (tDEGs): "identification of temporal differentially expressed genes (tDEGs)"
