---
name: multi-method-consensus-dci-identification
description: Use when scHi-C differential interaction results from multiple scHiCDiff
  detectors (KS, CVM, NB, ZINB) are available and a robust, cross-method-supported
  set of DCIs is needed rather than a single-method call.
when_to_use_negative:
- You only need the best single-method call set — ZINB alone gave the most accurate
  simulation results, so intersecting all four may be unnecessarily stringent
- Count matrix is raw (not normalized) or column counts do not match the two-group
  factor vector lengths
- Data are bulk Hi-C rather than single-cell Hi-C; ZINB is specifically designed for
  single-cell interaction comparison with excessive zeros
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: scHiCDiff
  role: Provides scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB, scHiCDiff.ZINB detection
    functions and the scHiCDiff.common.DCI intersection function
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Implementation environment; scHiCDiff is installed as a source package and
    run from the R library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:7f6b6abad3cc5a392d3f3a04abd68b75f52f32f4919833e1dbfa41713096de7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# multi-method-consensus-dci-identification

## Summary

Intersect the differential chromatin interaction (DCI) calls from four scHiCDiff detectors — non-parametric KS and CVM tests and parametric NB/ZINB likelihood-ratio tests — to obtain the consensus set of bin pairs identified by all methods, as demonstrated on simulated chr1 1CDX1 scHi-C data.

## When to use

Use when scHi-C differential interaction results from multiple scHiCDiff detectors (KS, CVM, NB, ZINB) are available and a robust, cross-method-supported set of DCIs is needed rather than a single-method call.

## When NOT to use

- You only need the best single-method call set — ZINB alone gave the most accurate simulation results, so intersecting all four may be unnecessarily stringent
- Count matrix is raw (not normalized) or column counts do not match the two-group factor vector lengths
- Data are bulk Hi-C rather than single-cell Hi-C; ZINB is specifically designed for single-cell interaction comparison with excessive zeros

## Inputs

- Normalized scHi-C count matrix (matrix) with bin pairs as rows and 100 cell columns (50 per condition), e.g. sampledata/sim 1.rds
- Factor vector group, e.g. factor(c(rep(1,50), rep(2,50))) matching count.table columns
- Four DCI result data frames: result.ks, result.cvm (non-parametric) and result.nb, result.zinb (parametric)

## Outputs

- common.DCIs data frame of bin pairs (bin_1, bin_2) called as DCIs by all four methods
- Per-method result data frames with bin_1, bin_2, test.statistic/chi2LR1, pvalue, pvalue.adj.FDR/pvalue.adj columns

## How to apply

Run each of the four scHiCDiff detection functions separately on the same normalized count.table (rows = bin pairs, columns = 50 condition-1 + 50 condition-2 cells) and the same factor group vector. The non-parametric calls return bin_1, bin_2, test.statistic, pvalue and BH-adjusted pvalue.adj; the parametric NB/ZINB calls return MLE parameters (mu, theta, pi), chi2LR1 statistics, pvalue and pvalue.adj.FDR. Pass all four result objects into scHiCDiff.common.DCI(result.ks, result.cvm, result.nb, result.zinb) to compute the overlap. Rationale: per the paper, the parametric models — especially ZINB, which models excessive single-cell zeros — outperform KS/CVM in simulations, so the intersection retains bin pairs supported by both distribution-free and model-based evidence.

## Related tools

- **scHiCDiff** (Provides scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB, scHiCDiff.ZINB detection functions and the scHiCDiff.common.DCI intersection function) — https://github.com/wmalab/scHiCDiff
- **R** (Implementation environment; scHiCDiff is installed as a source package and run from the R library)

## Examples

```
result.zinb <- scHiCDiff.ZINB(count.table,group); common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)
```

## Evaluation signals

- All four per-method data frames contain bin_1, bin_2, pvalue and FDR-adjusted p-value columns with adjusted values in [0,1]
- Parametric results expose chi2LR1 statistics and MLE parameters (mu, theta, pi) without 'Remark' abnormal-program notes
- The common-DCI set is non-empty and equals the overlap of the four per-method DCI calls on the simulated data
- Reproduces the paper's benchmark behavior: NB/ZINB outperform KS/CVM, with ZINB most accurate across fold change, resolution and sample-size settings

## Limitations

- Consensus intersection is as sensitive as the weakest method; non-parametric KS/CVM lag the parametric models in all simulation settings
- Package is a developmental version (scHiCDiff_1.0.tar.gz) installed from source; no version numbers stated for scHiCDiff or R
- Sample data requires bin pairs with at least one non-zero count in one cell type; fully zero bin pairs are not analyzed

## Evidence

- [methods] common DCIs identified by four methods: "common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)"
- [intro] four DCI approaches: "we introduced 4 ways to capture the DCIs: two non-parametric tests (Kolmogorov–Smirnov test/ Cramér-von Mises test) and parametric likelihood ratio test"
- [readme] ZINB rationale: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros"
- [readme] sample data description: "The simulated data getting from chr1 of  1CDX1 with cell num=50, fold change=5 and resolution=200kb (Nagano et.al.)"
- [abstract] benchmark finding: "Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs"
