---
name: negative-binomial-likelihood-ratio-testing
description: Use when comparing two single-cell Hi-C conditions (e.g., mouse oocyte
  vs zygote, Astro vs Endo) at bin-pair resolution from a normalized count table,
  and per-bin-pair differential interaction calls with raw and BH-adjusted p-values
  are needed.
when_to_use_negative:
- The data are bulk Hi-C contact matrices without single-cell-level sparsity structure
  — the article positions the ZINB regression specially for the excessive zeros of
  single-cell interaction comparison
- Only non-negative KS/CVM results and pvalue.adj columns are available; scHiCDiff.NB/ZINB
  require a normalized count.table and produce the parametric (mu/theta/pi, chi2LR1)
  statistics
- The count table has not been normalized (e.g., with scHiCDiff.norm using effective
  length, GC content, mappability), since the detection functions expect normalized
  scHi-C read counts
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_0128
tools:
- name: scHiCDiff
  role: Provides scHiCDiff.NB and scHiCDiff.ZINB parametric likelihood-ratio test
    functions plus scHiCDiff.common.DCI for consensus DCI extraction
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Implementation environment in which scHiCDiff is installed from the source
    tarball and the detection functions are run
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:90c8211b08c24e52040873bb41c9fdd8e4d3858dcf7dd118ef55ed7011f4af5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# negative-binomial-likelihood-ratio-testing

## Summary

Parametric detection of differential chromatin interactions (DCIs) in single-cell Hi-C via scHiCDiff's Negative Binomial (scHiCDiff.NB) and Zero-inflated Negative Binomial (scHiCDiff.ZINB) likelihood-ratio tests, which fit (mu, theta, pi) per condition and test the null with a chi2LR1 statistic. The ZINB option is specifically designed for single-cell counts by accounting for excessive zeros.

## When to use

Use when comparing two single-cell Hi-C conditions (e.g., mouse oocyte vs zygote, Astro vs Endo) at bin-pair resolution from a normalized count table, and per-bin-pair differential interaction calls with raw and BH-adjusted p-values are needed.

## When NOT to use

- The data are bulk Hi-C contact matrices without single-cell-level sparsity structure — the article positions the ZINB regression specially for the excessive zeros of single-cell interaction comparison
- Only non-negative KS/CVM results and pvalue.adj columns are available; scHiCDiff.NB/ZINB require a normalized count.table and produce the parametric (mu/theta/pi, chi2LR1) statistics
- The count table has not been normalized (e.g., with scHiCDiff.norm using effective length, GC content, mappability), since the detection functions expect normalized scHi-C read counts

## Inputs

- Normalized scHi-C count.table matrix (rows = bin pairs, columns = cells/samples, non-negative counts)
- group factor vector of length ncol(count.table) labeling the two conditions to compare (e.g., rep(1,50), rep(2,50))

## Outputs

- Data frame of DCI results per bin pair with columns bin_1, bin_2, mu_1/mu_2, theta_1/theta_2 (and pi_1/pi_2 for ZINB), norm_total_mean_1/2, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR, Remark
- pvalue.adj.FDR (Benjamini & Hochberg adjusted p-values) for significance calling
- Optional consensus DCI set from scHiCDiff.common.DCI across NB, ZINB, KS, CVM results

## How to apply

Assemble a non-negative count.table matrix whose rows are bin pairs and whose columns are cells/samples, plus a group factor labeling the two conditions corresponding to the columns. Call scHiCDiff.NB(count.table,group) and/or scHiCDiff.ZINB(count.table,group); these fit NB/ZINB (mu, theta, and pi for ZINB) parameters per group via MLE and compute chi-square statistics (chi2LR1) for the null hypothesis that a bin pair is not a DCI. Use ZINB rather than NB when single-cell sparsity (excessive zeros) is expected, since the article found the ZINB option gave the most consistent and stable detections. Adjust p-values with Benjamini & Hochberg FDR (pvalue.adj.FDR) before interpreting significance, and check the Remark column for abnormal program information. Optionally intersect NB/ZINB results with the non-parametric KS/CVM results via scHiCDiff.common.DCI to identify consensus DCIs.

## Related tools

- **scHiCDiff** (Provides scHiCDiff.NB and scHiCDiff.ZINB parametric likelihood-ratio test functions plus scHiCDiff.common.DCI for consensus DCI extraction) — https://github.com/wmalab/scHiCDiff
- **R** (Implementation environment in which scHiCDiff is installed from the source tarball and the detection functions are run) — https://github.com/wmalab/scHiCDiff

## Examples

```
count.table <- as.matrix(readRDS("path/sampledata/sim 1 .rds")); group <- factor(c(rep(1,50), rep(2,50))); result.zinb <- scHiCDiff.ZINB(count.table,group)
```

## Evaluation signals

- Output data frame is non-empty, with rows as bin pairs and required columns present (bin_1, bin_2, chi2LR1, pvalue, pvalue.adj.FDR)
- All p-values and FDR-adjusted p-values lie in [0,1]; pvalue.adj.FDR >= pvalue
- norm_foldChange equals norm_total_mean_1/norm_total_mean_2 per bin pair
- Remark column records any abnormal program information (empty Remark indicates clean fits)
- ZINB-based DCI detections agree with the KS/CVM/NB call sets (consistency check via scHiCDiff.common.DCI), consistent with the reported result that ZINB provides the most consistent and stable detections

## Limitations

- No version numbers are stated for scHiCDiff or R; the developmental version is installed from a source tarball (scHiCDiff_1.0.tar.gz)
- Abnormal program information is only recorded in the Remark column and must be inspected manually
- Parametric assumptions (NB/ZINB) impose distributional assumptions, unlike the non-parametric KS/CVM tests, which detect DCIs without any assumption on data distribution

## Evidence

- [intro] ZINB regression is specifically designated for single-cell comparison: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [readme] README describes detection-function inputs: "count.table      A non-negative  matrix of scHi-C normalized read counts.The rows of the 
                 matrix are bin pair and columns are samples/cells."
- [readme] README describes chi2LR1 and FDR outputs: "chi2LR1              Chi-square statistic for hypothesis testing of H0."
- [readme] README example invocation: "result.nb <- scHiCDiff.NB(count.table,group)"
- [abstract] Abstract finding on ZINB effectiveness: "Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions"
