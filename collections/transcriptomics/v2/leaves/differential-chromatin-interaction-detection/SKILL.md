---
name: differential-chromatin-interaction-detection
description: Use when you have a normalized scHi-C count matrix (rows = bin pairs,
  columns = cells) from two conditions and need per-bin-pair differential interaction
  statistics (pvalue, pvalue.adj.FDR) plus a consensus DCI set.
when_to_use_negative:
- Data are bulk Hi-C contact counts rather than single-cell-level count matrices —
  scHiCDiff's ZINB zero-inflation model is specifically designed for single-cell interaction
  comparison
- Counts are raw/unnormalized — normalize with scHiCDiff.norm (effective length, GC
  content, mappability) first, since detection functions expect normalized read counts
- Cells fail QC (< 5000 nondiagonal intra-chromosomal contacts) or bin pairs lie beyond
  the 0–10 Mb contact-distance range, per the paper's real-data filters
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: scHiCDiff
  role: Provides scHiCDiff.norm for normalization and scHiCDiff.KS, scHiCDiff.CVM,
    scHiCDiff.NB, scHiCDiff.ZINB for DCI detection plus scHiCDiff.common.DCI to intersect
    per-method calls
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Language in which scHiCDiff is implemented and run (install the source package
    scHiCDiff_1.0.tar.gz)
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:1232f5dddd2b434716109d3fc26a18171cfd59ee09206b8ba39d427b3942d20b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# differential-chromatin-interaction-detection

## Summary

Detect differential chromatin interactions (DCIs) between two conditions in single-cell Hi-C data using scHiCDiff's four detectors — nonparametric KS and Cramér-von Mises tests and parametric NB/ZINB likelihood-ratio tests — then take the common DCI set with scHiCDiff.common.DCI.

## When to use

Use when you have a normalized scHi-C count matrix (rows = bin pairs, columns = cells) from two conditions and need per-bin-pair differential interaction statistics (pvalue, pvalue.adj.FDR) plus a consensus DCI set.

## When NOT to use

- Data are bulk Hi-C contact counts rather than single-cell-level count matrices — scHiCDiff's ZINB zero-inflation model is specifically designed for single-cell interaction comparison
- Counts are raw/unnormalized — normalize with scHiCDiff.norm (effective length, GC content, mappability) first, since detection functions expect normalized read counts
- Cells fail QC (< 5000 nondiagonal intra-chromosomal contacts) or bin pairs lie beyond the 0–10 Mb contact-distance range, per the paper's real-data filters

## Inputs

- count.table: non-negative matrix of scHi-C normalized read counts (rows = bin pairs, columns = samples/cells), e.g. loaded from sampledata/sim 1.rds via readRDS
- group: factor vector labeling the two conditions, corresponding to count.table columns (e.g. rep(1,50), rep(2,50))
- bias.info.path: pathway of the three local features (effective length, GC content, mappability of fragment ends) for scHiCDiff.norm normalization

## Outputs

- Per-method data frames of bin pairs: bin_1, bin_2, test.statistic (KS/CVM) or mu_1/mu_2/theta_1/theta_2/pi_1/pi_2, norm_total_mean_1/2, norm_foldChange, chi2LR1 (NB/ZINB), pvalue and pvalue.adj / pvalue.adj.FDR, plus Remark for abnormal program information
- common.DCIs: data frame of bin pairs identified as DCIs by all four methods via scHiCDiff.common.DCI

## How to apply

Build count.table as a non-negative matrix of normalized scHi-C counts whose rows are bin pairs and whose columns are cells, plus a group factor (e.g. factor(c(rep(1,50), rep(2,50)))) matching the two conditions in column order. Run the nonparametric detectors scHiCDiff.KS and scHiCDiff.CVM, which return test.statistic and BH-adjusted pvalue.adj, and the parametric likelihood-ratio detectors scHiCDiff.NB and scHiCDiff.ZINB, which fit mu/theta (and pi, the zero-percentage estimate, for ZINB) per condition and return chi2LR1, pvalue and pvalue.adj.FDR. Apply a QC filter to real data first: retain only cells with more than 5000 total nondiagonal intra-chromosomal contacts and restrict bin pairs to the 0–10 Mb contact-distance range. ZINB is designed specifically for single-cell interaction comparison because it models the excessive zeros; in practice it gives the most accurate, consistent detections, so treat the ZINB call set as the primary result and intersect all four methods with scHiCDiff.common.DCI for the consensus set.

## Related tools

- **scHiCDiff** (Provides scHiCDiff.norm for normalization and scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB, scHiCDiff.ZINB for DCI detection plus scHiCDiff.common.DCI to intersect per-method calls) — https://github.com/wmalab/scHiCDiff
- **R** (Language in which scHiCDiff is implemented and run (install the source package scHiCDiff_1.0.tar.gz)) — https://github.com/wmalab/scHiCDiff

## Examples

```
result.zinb <- scHiCDiff.ZINB(count.table, group); common.DCIs <- scHiCDiff.common.DCI(result.ks, result.cvm, result.nb, result.zinb)
```

## Evaluation signals

- All result data frames contain the required columns: bin_1, bin_2, test.statistic or chi2LR1, pvalue and pvalue.adj / pvalue.adj.FDR, with adjusted values in [0,1]
- For parametric models, mu_1/mu_2 (means), theta_1/theta_2 (dispersions) and pi_1/pi_2 (zero percentages) are non-negative MLEs, and norm_foldChange equals norm_total_mean_1/norm_total_mean_2
- common.DCIs is non-empty and equals the overlap of the four per-method DCI call sets
- On simulated data (cell num=50, fold change=5, resolution=200kb), parametric NB/ZINB detectors outperform nonparametric KS/CVM, with ZINB producing the most accurate results
- The Remark column records any abnormal program information instead of silently failing a fit

## Limitations

- No version numbers stated for scHiCDiff or R; only a developmental tarball install is documented
- The sample data lists only bin pairs with at least one non-zero count in one of the cell types, so zero-count pairs across both conditions are excluded
- Parametric fits can emit abnormal program information recorded in the Remark column, so per-fit warnings must be inspected
- Nonparametric KS/CVM tests detect DCIs without distributional assumptions but underperform the parametric NB/ZINB models in simulations

## Evidence

- [intro] we introduced 4 ways to capture the DCIs: two non-parametric tests (Kolmogorov–Smirnov test/ Cramér-von Mises test) and parametric likelihood ratio test with two regression models: "we introduced 4 ways to capture the DCIs: two non-parametric tests (Kolmogorov–Smirnov test/ Cramér-von Mises test) and parametric likelihood ratio test with two regression models"
- [readme] The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics.: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics."
- [methods] common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb): "#common DCIs identified by four methods
common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)"
- [intro] zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [results] only cells with more than 5000 total nondiagonal intra-chromosomal contacts, and contacts within the 0–10 Mb range, were used: [evidence span withheld — anchor status: not_found]
