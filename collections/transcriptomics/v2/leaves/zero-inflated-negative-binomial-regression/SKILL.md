---
name: zero-inflated-negative-binomial-regression
description: Use when comparing single-cell Hi-C normalized read counts between two
  conditions (rows = bin pairs, columns = cells) and you need statistically principled
  DCI calls that handle the excessive zeros typical of scHi-C data.
when_to_use_negative:
- The input is raw (unnormalized) scHi-C data — apply the scHiCDiff.norm normalization
  using local features (effective length, GC content, mappability) first, since scHiCDiff.ZINB
  expects normalized read counts
- Only bulk Hi-C data without excessive zeros is available — the article states negative
  binomial (NB) is the common assumption for bulk Hi-C, while ZINB is 'specially designated
  for the interaction comparison at single-cell level'
- No distributional assumptions can be justified — the non-parametric KS or CVM tests
  (scHiCDiff.KS / scHiCDiff.CVM) are designed for detecting DCIs 'without any assumption
  on data distribution'
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0611
- http://edamontology.org/topic_3942
tools:
- name: scHiCDiff
  role: Provides the scHiCDiff.ZINB (and companion NB/KS/CVM/common.DCI) detection
    functions implementing the zero-inflated negative binomial regression test on
    scHi-C count tables
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Runtime environment in which scHiCDiff (developmental version, installed from
    scHiCDiff_1.0.tar.gz) is implemented and executed
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:4f623826d7b4b9d59961445f4d3480d32492fdcec7799412a4830eb77caf0286
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0611
  - http://edamontology.org/topic_3942
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# zero-inflated-negative-binomial-regression

## Summary

A parametric likelihood-ratio-test DCI detector from the scHiCDiff R package that fits zero-inflated negative binomial (mu, theta, pi) regression models per condition to single-cell Hi-C bin-pair count matrices and calls differential chromatin interactions. It is designed for single-cell interaction comparison because it explicitly accounts for excessive zeros in scHi-C counts.

## When to use

Use when comparing single-cell Hi-C normalized read counts between two conditions (rows = bin pairs, columns = cells) and you need statistically principled DCI calls that handle the excessive zeros typical of scHi-C data.

## When NOT to use

- The input is raw (unnormalized) scHi-C data — apply the scHiCDiff.norm normalization using local features (effective length, GC content, mappability) first, since scHiCDiff.ZINB expects normalized read counts
- Only bulk Hi-C data without excessive zeros is available — the article states negative binomial (NB) is the common assumption for bulk Hi-C, while ZINB is 'specially designated for the interaction comparison at single-cell level'
- No distributional assumptions can be justified — the non-parametric KS or CVM tests (scHiCDiff.KS / scHiCDiff.CVM) are designed for detecting DCIs 'without any assumption on data distribution'

## Inputs

- Normalized scHi-C count matrix (count.table): non-negative matrix with bin pairs as rows and cells/samples as columns; sample data file 'sampledata/sim 1.rds' from chr1 of 1CDX1 (cell num=50, fold change=5, resolution=200kb)
- Factor vector `group` with two levels labeling the two conditions to be compared, corresponding one-to-one with count.table columns (e.g. factor(c(rep(1,50), rep(2,50))))

## Outputs

- Data frame of DCI results with one row per bin pair and columns: bin_1, bin_2, mu_1, mu_2, theta_1, theta_2, pi_1, pi_2, norm_total_mean_1, norm_total_mean_2, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR, Remark
- Statistically significant bin pairs (FDR-adjusted p-values) for downstream intersection via scHiCDiff.common.DCI

## How to apply

Assemble a non-negative matrix of scHi-C normalized read counts (rows = bin pairs, columns = samples/cells) and a factor vector `group` labeling the two conditions, matching the count-table column order. Call scHiCDiff.ZINB(count.table, group), which computes MLEs of the NB/ZINB parameters mu (mean), theta (dispersion) and pi (zero percentage) for each group, returns norm_foldChange (norm_total_mean_1/norm_total_mean_2), the chi2LR1 chi-square statistic for H0, and raw plus Benjamini & Hochberg FDR-adjusted p-values (pvalue, pvalue.adj.FDR), with abnormal program information recorded in a Remark column. ZINB is preferred over NB for single-cell data because it models the zero-inflation, and in the paper's benchmarks ZINB gave the most accurate results across simulated settings (fold change, resolution, sample size) and the most consistent/stable detections on real datasets (oocyte vs zygote, Astro vs Endo). Pair it with the non-parametric scHiCDiff.KS/CVM tests and scHiCDiff.common.DCI to intersect common DCIs across the four methods.

## Related tools

- **scHiCDiff** (Provides the scHiCDiff.ZINB (and companion NB/KS/CVM/common.DCI) detection functions implementing the zero-inflated negative binomial regression test on scHi-C count tables) — https://github.com/wmalab/scHiCDiff
- **R** (Runtime environment in which scHiCDiff (developmental version, installed from scHiCDiff_1.0.tar.gz) is implemented and executed) — https://github.com/wmalab/scHiCDiff

## Examples

```
count.table <- as.matrix(readRDS("path/sampledata/sim 1.rds")); group <- factor(c(rep(1,50), rep(2,50))); result.zinb <- scHiCDiff.ZINB(count.table, group)
```

## Evaluation signals

- Result data frame exists and contains the required parametric-model columns: bin_1, bin_2, mu/theta/pi MLEs, chi2LR1, pvalue, and pvalue.adj.FDR, with p-values in [0,1]
- Remark column is inspected for abnormal program information and the bin-pair rows are non-empty
- ZINB-based significant DCI set overlaps (is consistent with) the DCI calls from scHiCDiff.KS, scHiCDiff.CVM and scHiCDiff.NB, and the scHiCDiff.common.DCI intersection is non-empty
- norm_foldChange equals norm_total_mean_1/norm_total_mean_2 for each bin pair, and H0 testing is based on chi2LR1 chi-square statistics
- On simulated benchmarks (sim 1.rds, chr1 of 1CDX1, cell num=50, fold change=5, 200kb resolution), ZINB ranks as the most accurate detector relative to KS, CVM and NB

## Limitations

- Parametric assumptions: ZINB assumes zero-inflated negative-binomial distributed counts; if the count distribution or zero-inflation assumption is violated, non-parametric KS/CVM alternatives are preferable
- No version numbers stated for scHiCDiff (beyond the developmental scHiCDiff_1.0.tar.gz) or R in the documentation
- The detection function only records abnormal program information in a Remark column rather than failing loudly, so the Remark field must be checked
- Parametric model fitting can be slower/costlier than non-parametric KS/CVM tests (reason for pairing with common-DCI intersection was not detailed in the provided context)

## Evidence

- [intro] ZINB specially designed for single-cell zero excess: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [methods] Detection function returns DCI data frame: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results"
- [readme] ZINB invocation example: "result.zinb <- scHiCDiff.ZINB(count.table,group)"
- [readme] FDR-adjusted p-value output column: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [abstract] ZINB outperforms on simulated and real data: "Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions"
