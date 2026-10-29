---
name: zinb-differential-chromatin-interaction-testing
description: Use when you have a normalized scHi-C count matrix (rows = bin pairs,
  columns = samples/cells) plus a two-condition factor vector and need single-cell
  DCI calls that handle excessive zero counts.
when_to_use_negative:
- Data are bulk Hi-C interaction counts without single-cell-level excessive zeros
  — the ordinary negative binomial model (scHiCDiff.NB) is described as the common
  assumption for bulk Hi-C parametric approaches
- No distributional assumption is desired — the article offers non-parametric alternatives
  (scHiCDiff.KS, scHiCDiff.CVM) that detect DCIs without any assumption on data distribution
- Counts are raw/unnormalized — the README requires 'A non-negative matrix of scHi-C
  normalized read counts', so scHiCDiff.norm normalization must be applied first
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: scHiCDiff
  role: Provides the scHiCDiff.ZINB function that fits the zero-inflated negative
    binomial likelihood-ratio test per bin pair, plus scHiCDiff.norm for the required
    upstream normalization and scHiCDiff.common.DCI for cross-method concordance checks
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Implementation environment; scHiCDiff is installed from source (scHiCDiff_1.0.tar.gz)
    and loaded via install.packages(..., repos = NULL, type = "source") and library(scHiCDiff)
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:5c868dd7e2e4ffae9b76d6d6266fde8441357252b517a2048ae9fdd965f957e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# zinb-differential-chromatin-interaction-testing

## Summary

Detect differential chromatin interactions (DCIs) between two scHi-C conditions by fitting a zero-inflated negative binomial (ZINB) likelihood-ratio test per bin pair with scHiCDiff.ZINB in R; the ZINB model is specifically designed for single-cell interaction comparison because it accounts for excessive zeros.

## When to use

Use when you have a normalized scHi-C count matrix (rows = bin pairs, columns = samples/cells) plus a two-condition factor vector and need single-cell DCI calls that handle excessive zero counts.

## When NOT to use

- Data are bulk Hi-C interaction counts without single-cell-level excessive zeros — the ordinary negative binomial model (scHiCDiff.NB) is described as the common assumption for bulk Hi-C parametric approaches
- No distributional assumption is desired — the article offers non-parametric alternatives (scHiCDiff.KS, scHiCDiff.CVM) that detect DCIs without any assumption on data distribution
- Counts are raw/unnormalized — the README requires 'A non-negative matrix of scHi-C normalized read counts', so scHiCDiff.norm normalization must be applied first

## Inputs

- count.table: non-negative matrix of scHi-C normalized read counts (rows = bin pairs, columns = samples/cells), loadable via readRDS from an .rds file
- group: factor vector labeling the two conditions to be compared, corresponding to count.table columns

## Outputs

- Data frame of DCI results with columns bin_1, bin_2, mu_1, mu_2, theta_1, theta_2, pi_1, pi_2, norm_total_mean_1, norm_total_mean_2, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR, and Remark
- Set of significant DCIs selected from pvalue.adj.FDR with genomic coordinates from bin_1/bin_2

## How to apply

Load the normalized count matrix via readRDS (e.g. sampledata/sim 1 .rds from chr1 of 1CDX1, cell num=50, resolution=200kb) and coerce to a matrix with as.matrix; build the group factor (e.g. factor(c(rep(1,50), rep(2,50)))) matching column order. Call scHiCDiff.ZINB(count.table, group), which fits ZINB regression per bin pair and returns mu_1/mu_2, theta_1/theta_2, pi_1/pi_2 (zero-percentage estimates), chi2LR1 (chi-square statistic for H0), a raw pvalue, and pvalue.adj.FDR computed by Benjamini & Hochberg. Call significant DCIs by thresholding pvalue.adj.FDR and use the bin_1/bin_2 columns to obtain genomic coordinates of the interacting regions. Rationale: NB alone is the common bulk Hi-C assumption, but the ZINB variant adds the zero-inflation term (pi) that single-cell Hi-C interaction counts require. As an internal sanity check, the article also provides scHiCDiff.common.DCI to intersect ZINB results with KS, CVM, and NB calls to identify DCIs shared across the four methods.

## Related tools

- **scHiCDiff** (Provides the scHiCDiff.ZINB function that fits the zero-inflated negative binomial likelihood-ratio test per bin pair, plus scHiCDiff.norm for the required upstream normalization and scHiCDiff.common.DCI for cross-method concordance checks) — https://github.com/wmalab/scHiCDiff
- **R** (Implementation environment; scHiCDiff is installed from source (scHiCDiff_1.0.tar.gz) and loaded via install.packages(..., repos = NULL, type = "source") and library(scHiCDiff)) — https://github.com/wmalab/scHiCDiff

## Examples

```
count.table <- as.matrix(readRDS("path/sampledata/sim 1 .rds")); group <- factor(c(rep(1,50), rep(2,50))); result.zinb <- scHiCDiff.ZINB(count.table, group)
```

## Evaluation signals

- Result data frame contains the documented ZINB parameter columns (mu_1, mu_2, theta_1, theta_2, pi_1, pi_2, chi2LR1, pvalue, pvalue.adj.FDR) for every bin pair
- pvalue.adj.FDR values fall in [0, 1] and are >= the raw pvalue (Benjamini & Hochberg adjustment invariant)
- The Remark column should be free of abnormal program information for bins whose results are used downstream
- Validation per task_006: a majority of ZINB-detected DCIs are supported by bulk Hi-C method DCIs and located within differential TAD regions (Supplementary Note 6)
- ZINB calls overlap the DCIs from scHiCDiff.KS, scHiCDiff.CVM, and scHiCDiff.NB when intersected via scHiCDiff.common.DCI

## Limitations

- ZINB fits assume two groups only (group is 'A vector of factor which mentions the two condition to be compared'); multi-condition designs are out of scope
- Bin pairs with all-zero counts across cells cannot be tested; the sample data list only bin pairs with at least one non-zero count in one of the cell types
- No version numbers stated for scHiCDiff or R, so reproducibility pinning is left to the analyst
- Normalization is a prerequisite: count.table must already be normalized read counts (local features effective length, GC content, and mappability handled by scHiCDiff.norm)

## Evidence

- [intro] zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [readme] The inputs for all detection functions are illustrated below: count.table ... A non-negative matrix of scHi-C normalized read counts. The rows of the matrix are bin pair and columns are samples/cells.: "count.table      A non-negative  matrix of scHi-C normalized read counts.The rows of the 
                 matrix are bin pair and columns are samples/cells."
- [readme] pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method.: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [methods] result.zinb <- scHiCDiff.ZINB(count.table,group): "result.zinb <- scHiCDiff.ZINB(count.table,group)"
- [abstract] Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions: "Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions"
