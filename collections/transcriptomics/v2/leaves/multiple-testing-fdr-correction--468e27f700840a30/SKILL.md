---
name: multiple-testing-fdr-correction
description: Use when calling significant differential chromatin interactions from
  genome-wide scHi-C bin-pair test results, or when thresholding per-condition significance
  calls to build a set-difference baseline, so that thousands of simultaneous hypothesis
  tests are controlled with BH-adjusted p-values.
when_to_use_negative:
- When only a single pre-specified interaction (one hypothesis) is being tested —
  BH multiplicity correction is unnecessary and over-conservative
- When the input data frame already contains an adjusted p-value column (pvalue.adj.FDR
  / pvalue.adj); re-correcting adjusted p-values is incorrect
- When data are bulk Hi-C replicate counts rather than single-cell-level scHi-C count
  tables — scHiCDiff's NB/ZINB models and their BH-adjusted outputs are designed for
  single-cell comparison
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: scHiCDiff
  role: R package whose detection functions (scHiCDiff.KS/CVM/NB/ZINB) perform the
    hypothesis tests and return Benjamini & Hochberg adjusted p-values (pvalue.adj
    / pvalue.adj.FDR) per bin pair
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Runtime environment in which scHiCDiff's detection and BH-adjusted DCI calling
    are executed
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:201b9f24a66a0619d00c59a73d93d2ed6f61cabed230234fd4292f5aefbe2681
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - STEM
  - scHiCDiff
  - scSTEM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# multiple-testing-fdr-correction

## Summary

Benjamini & Hochberg FDR adjustment of per-bin-pair DCI p-values in scHiCDiff, applied across thousands of single-cell Hi-C bin pairs and returned as the pvalue.adj.FDR (parametric) or pvalue.adj (non-parametric) columns used for DCI calling and for thresholding the set-difference baseline.

## When to use

Use when calling significant differential chromatin interactions from genome-wide scHi-C bin-pair test results, or when thresholding per-condition significance calls to build a set-difference baseline, so that thousands of simultaneous hypothesis tests are controlled with BH-adjusted p-values.

## When NOT to use

- When only a single pre-specified interaction (one hypothesis) is being tested — BH multiplicity correction is unnecessary and over-conservative
- When the input data frame already contains an adjusted p-value column (pvalue.adj.FDR / pvalue.adj); re-correcting adjusted p-values is incorrect
- When data are bulk Hi-C replicate counts rather than single-cell-level scHi-C count tables — scHiCDiff's NB/ZINB models and their BH-adjusted outputs are designed for single-cell comparison

## Inputs

- Data frame of bin-pair test results with a raw 'pvalue' column (output of scHiCDiff.NB / scHiCDiff.ZINB / scHiCDiff.KS / scHiCDiff.CVM)
- Non-negative matrix of normalized scHi-C read counts (rows = bin pairs, columns = samples/cells) with a two-condition factor vector 'group'

## Outputs

- Data frame column pvalue.adj.FDR (Benjamini & Hochberg adjusted p-value) for parametric model outputs (NB/ZINB)
- Data frame column pvalue.adj (BH-adjusted p-value) for non-parametric test outputs (KS/CVM)
- Significant DCI call sets obtained by thresholding the adjusted p-value column, per condition or consensus via scHiCDiff.common.DCI

## How to apply

Run each scHiCDiff detection function (scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB, scHiCDiff.ZINB) on the normalized count.table (rows = bin pairs, columns = samples/cells) with the two-condition factor group; each function applies Benjamini & Hochberg correction internally and returns an adjusted p-value column alongside the raw pvalue. For parametric (NB/ZINB) results use the pvalue.adj.FDR column; for non-parametric (KS/CVM) results use the pvalue.adj column. Call DCIs by thresholding the adjusted p-value column (e.g. pvalue.adj.FDR) rather than the raw pvalue, since thousands of bin pairs are tested simultaneously. When constructing the set-difference baseline, threshold the BH-adjusted p-values independently per condition and take the difference of the two significant-interaction sets. Adjusted p-values must be monotonically >= their corresponding raw p-values.

## Related tools

- **scHiCDiff** (R package whose detection functions (scHiCDiff.KS/CVM/NB/ZINB) perform the hypothesis tests and return Benjamini & Hochberg adjusted p-values (pvalue.adj / pvalue.adj.FDR) per bin pair) — https://github.com/wmalab/scHiCDiff
- **R** (Runtime environment in which scHiCDiff's detection and BH-adjusted DCI calling are executed) — https://github.com/wmalab/scHiCDiff

## Examples

```
result.zinb <- scHiCDiff.ZINB(count.table, group)  # returns pvalue and Benjamini & Hochberg adjusted pvalue.adj.FDR per bin pair
```

## Evaluation signals

- Adjusted p-value columns are present: pvalue.adj.FDR in NB/ZINB result data frames and pvalue.adj in KS/CVM result data frames
- Every pvalue.adj.FDR / pvalue.adj value is >= its corresponding raw pvalue, and adjusted values lie in [0,1]
- DCI call sets derived by thresholding pvalue.adj.FDR reproduce the ZINB call sets used in the real-data comparisons (oocyte vs zygote; astrocytes vs endothelial cells)
- Set-difference baseline built by BH-thresholding per condition independently differs from the scHiCDiff consensus (scHiCDiff.common.DCI) call set, with ZINB calls agreeing more with recovered differential bin pairs (characterized via norm_foldChange, chi2LR1, pvalue.adj.FDR)
- Adjusted p-values are computed with Benjamini & Hochberg's method as stated in the scHiCDiff output specification

## Limitations

- The column name for BH-adjusted p-values differs by test type: pvalue.adj for non-parametric (KS/CVM) outputs vs pvalue.adj.FDR for parametric (NB/ZINB) outputs — downstream scripts must handle both
- No version numbers for scHiCDiff or R are stated, and no explicit FDR cutoff threshold is fixed in the documentation; the cutoff choice (e.g. FDR <= 0.05) is left to the analyst
- Abnormal program information is only recorded in the 'Remark' column of parametric-model outputs; failure to check it can leave silently problematic bin-pair fits in the adjusted p-value table

## Evidence

- [readme] Adjusted P value of H0's pvalue using Benjamini & Hochberg's method.: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [readme] Non-parametric outputs carry pvalue.adj: "pvalue.adj           Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [abstract] ZINB effectively detects reliable and consistent single-cell DCIs: "Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions"
- [methods] Detection functions return DCI analysis results as a data frame: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics."
- [methods] Consensus DCIs across four methods: "#common DCIs identified by four methods
common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)"
