---
name: single-cell-hic-count-matrix-analysis
description: Use when you have a normalized scHi-C count matrix (rows = bin pairs,
  columns = cells) plus a two-level condition factor and need per-bin-pair differential
  interaction statistics and BH-adjusted p-values.
when_to_use_negative:
- Input is a raw, unnormalized N*N scHi-C contact matrix — run the scHiCDiff.norm
  normalization step (Gaussian convolution imputation + scHiCNorm NB-hurdle) first.
- 'Comparing bulk Hi-C experiments rather than single-cell Hi-C: the ZINB model is
  specifically designed for single-cell zero inflation, though NB remains the common
  bulk-Hi-C assumption.'
- 'More than two conditions per comparison: the detection functions take a factor
  of exactly two conditions.'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: scHiCDiff
  role: R package providing scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB, scHiCDiff.ZINB
    detection functions and scHiCDiff.common.DCI to extract common DCIs from a normalized
    count table
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Runtime environment in which scHiCDiff is implemented and its detection/normalization
    functions are invoked
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:54a7cd9266742e404cd0addb3ab3f847d5921513a4acdd9ad2cba00cfdf1fab9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-hic-count-matrix-analysis

## Summary

Detect differential chromatin interactions (DCIs) between two cell conditions from a normalized single-cell Hi-C count matrix using scHiCDiff's four tests (KS, CVM, NB, ZINB likelihood-ratio tests) and extract their common DCI set.

## When to use

Use when you have a normalized scHi-C count matrix (rows = bin pairs, columns = cells) plus a two-level condition factor and need per-bin-pair differential interaction statistics and BH-adjusted p-values.

## When NOT to use

- Input is a raw, unnormalized N*N scHi-C contact matrix — run the scHiCDiff.norm normalization step (Gaussian convolution imputation + scHiCNorm NB-hurdle) first.
- Comparing bulk Hi-C experiments rather than single-cell Hi-C: the ZINB model is specifically designed for single-cell zero inflation, though NB remains the common bulk-Hi-C assumption.
- More than two conditions per comparison: the detection functions take a factor of exactly two conditions.

## Inputs

- Normalized scHi-C count table (non-negative matrix; rows = bin pairs, columns = samples/cells), e.g. readRDS('path/sampledata/sim 1.rds')
- Factor vector 'group' labeling the two conditions to be compared, corresponding to count-table columns

## Outputs

- Per-method data frames: for KS/CVM — bin_1, bin_2, test.statistic, pvalue, pvalue.adj; for NB/ZINB — bin_1, bin_2, mu_1/mu_2, theta_1/theta_2, (pi_1/pi_2), norm_total_mean_1/2, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR, Remark
- Common DCI set across the four methods from scHiCDiff.common.DCI

## How to apply

Load the normalized count table with readRDS and coerce to matrix; rows must be bin pairs with the first two columns giving the interacting regions, and remaining columns the per-cell normalized read counts for the two conditions. Build a factor vector 'group' whose levels correspond to the condition of each column (e.g. factor(c(rep(1,50), rep(2,50)))). Run the two non-parametric tests scHiCDiff.KS and scHiCDiff.CVM (no distributional assumption, returning test.statistic, pvalue, pvalue.adj) and the two parametric likelihood-ratio tests scHiCDiff.NB and scHiCDiff.ZINB (returning mu/theta MLEs, pi zero-percent estimates for ZINB, chi2LR1, pvalue, pvalue.adj.FDR via Benjamini-Hochberg). Prefer the ZINB model for single-cell data because it models the excessive zeros characteristic of scHi-C. Combine the four result data frames with scHiCDiff.common.DCI to obtain DCIs identified by all methods; prior to this, data must have been normalized (Gaussian convolution imputation plus scHiCNorm NB-hurdle bias removal with effective length, GC content, and fragment-end mappability features).

## Related tools

- **scHiCDiff** (R package providing scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB, scHiCDiff.ZINB detection functions and scHiCDiff.common.DCI to extract common DCIs from a normalized count table) — https://github.com/wmalab/scHiCDiff
- **R** (Runtime environment in which scHiCDiff is implemented and its detection/normalization functions are invoked) — https://github.com/wmalab/scHiCDiff

## Examples

```
group <- factor(c(rep(1,50), rep(2,50))); result.zinb <- scHiCDiff.ZINB(count.table, group); common.DCIs <- scHiCDiff.common.DCI(result.ks, result.cvm, result.nb, result.zinb)
```

## Evaluation signals

- Output data frame rows are bin pairs and columns contain the documented statistics (bin_1, bin_2, test statistic, pvalue, pvalue.adj.FDR).
- All p-values fall in [0,1] and the result data frames are non-empty.
- Parametric outputs include mu/theta MLEs (and pi for ZINB) plus chi2LR1 chi-square statistics with BH-adjusted p-value columns present.
- ZINB-based detections are consistent and stable, agreeing with the KS, CVM, and NB detections (majority supported by bulk Hi-C DCIs and located within differential TAD regions on real data).
- scHiCDiff.common.DCI returns the intersection of DCIs identified by all four methods.

## Limitations

- Count table must be non-negative and pre-normalized; the detection functions expect normalized read counts, and rows should list bin pairs with at least one non-zero count in one cell type.
- No version numbers stated for scHiCDiff (developmental 1.0 tar.gz) or R; install from the developmental source package.
- Non-parametric tests make no distributional assumption but the parametric NB/ZINB fits can emit abnormal-program notes recorded in the 'Remark' output column.

## Evidence

- [intro] two non-parametric tests (Kolmogorov–Smirnov, Cramér-von Mises) and parametric likelihood ratio tests with two regression models (Negative Binomial, Zero-inflated Negative Binomial): "we introduced 4 ways to capture the DCIs: two non-parametric tests (Kolmogorov–Smirnov test/ Cramér-von Mises test) and parametric likelihood ratio test with two regression models"
- [readme] Detection function inputs and outputs: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics."
- [intro] ZINB rationale for excessive zeros: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [methods] Common DCI extraction across four methods: "common DCIs identified by four methods
common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)"
- [methods] Sample data format and group construction: "The first two columns represent the interacting region of each listed bin pair, then followed 50 columns denote the normalized read counts for condition 1 and the last 50 columns denote the"
