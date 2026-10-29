---
name: set-difference-significant-interaction-comparison
description: Use when evaluating whether a joint two-condition DCI method (e.g. scHiCDiff
  NB/ZINB) recovers differential bin pairs more reliably than independently thresholding
  significance calls in each condition and subtracting the resulting interaction sets.
when_to_use_negative:
- Input is bulk Hi-C data rather than single-cell-level interaction counts, where
  the excessive-zero modeling motivating the ZINB comparison does not apply
- The two conditions have no per-condition significance calls or adjusted p-values
  available, so the set-difference baseline cannot be derived
- The goal is DCI detection itself rather than benchmarking a detection method against
  a baseline
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: scHiCDiff
  role: Provides the four DCI detection functions (scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB,
    scHiCDiff.ZINB) whose call sets are compared against the set-difference baseline,
    and scHiCDiff.common.DCI to assemble the consensus set
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Environment in which scHiCDiff is implemented and the benchmark comparison
    is scripted
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:2a3ba42850524248ddfad03ffaeb62b3a5ce5fe05a41628d35b3706fdff40e49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# set-difference-significant-interaction-comparison

## Summary

Benchmarks scHiCDiff's differential chromatin interaction (DCI) calls against the set-difference baseline, where significant interactions are called independently per condition via Benjamini & Hochberg adjusted p-values and the two sets are subtracted. This comparison tests whether joint modeling of both conditions (especially the ZINB model) outperforms the naive per-condition approach on real single-cell Hi-C data.

## When to use

Use when evaluating whether a joint two-condition DCI method (e.g. scHiCDiff NB/ZINB) recovers differential bin pairs more reliably than independently thresholding significance calls in each condition and subtracting the resulting interaction sets.

## When NOT to use

- Input is bulk Hi-C data rather than single-cell-level interaction counts, where the excessive-zero modeling motivating the ZINB comparison does not apply
- The two conditions have no per-condition significance calls or adjusted p-values available, so the set-difference baseline cannot be derived
- The goal is DCI detection itself rather than benchmarking a detection method against a baseline

## Inputs

- scHiCDiff detection result data frames (result.ks, result.cvm, result.nb, result.zinb) with rows as bin pairs and columns of test statistics, pvalue, and pvalue.adj.FDR
- Normalized single-cell Hi-C count matrix (count.table) with bin pairs as rows and samples/cells as columns, plus a factor group vector labeling the two conditions

## Outputs

- Set-difference baseline: difference of independently thresholded significant-interaction sets from the two conditions
- Consensus scHiCDiff DCI set from scHiCDiff.common.DCI
- Benchmark comparison table written to scdhicdiff_vs_setdifference_benchmark.csv

## How to apply

Load the four scHiCDiff detection result data frames (result.ks, result.cvm, result.nb, result.zinb), each listing bin pairs with pvalue and pvalue.adj.FDR columns. Derive the set-difference baseline by applying the Benjamini & Hochberg adjustment to per-condition significance calls independently in the two conditions, then taking the difference of the two significant-interaction sets. Assemble the scHiCDiff consensus call set with scHiCDiff.common.DCI(result.ks, result.cvm, result.nb, result.zinb). Compare the consensus DCI set (and the ZINB call set in particular) against the set-difference baseline in terms of recovered differential bin pairs, using the statistics columns (norm_foldChange, chi2LR1, pvalue.adj.FDR) to characterize agreement. Summarize the comparison in a table written to scdhicdiff_vs_setdifference_benchmark.csv. The rationale: per-condition thresholding ignores zero inflation and joint modeling, whereas the zero-inflated negative binomial model is specifically designed for single-cell interaction comparison by accounting for excessive zeros.

## Related tools

- **scHiCDiff** (Provides the four DCI detection functions (scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB, scHiCDiff.ZINB) whose call sets are compared against the set-difference baseline, and scHiCDiff.common.DCI to assemble the consensus set) — https://github.com/wmalab/scHiCDiff
- **R** (Environment in which scHiCDiff is implemented and the benchmark comparison is scripted) — https://github.com/wmalab/scHiCDiff

## Examples

```
count.table <- readRDS("path/sampledata/sim 1 .rds"); result.zinb <- scHiCDiff.ZINB(count.table,group); common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)
```

## Evaluation signals

- The evaluation qualitatively reproduces the paper's reported result that scHiCDiff — especially the zero-inflated negative binomial model — yields more reliable and consistent DCI detections than the set-difference approach on real data (mouse oocyte vs zygote; astrocytes vs endothelial cells)
- The ZINB call set (pvalue.adj.FDR from scHiCDiff.ZINB) recovers more differential bin pairs than the per-condition set-difference baseline
- The consensus set from scHiCDiff.common.DCI(result.ks, result.cvm, result.nb, result.zinb) is non-empty and comparable against the baseline using norm_foldChange, chi2LR1, and pvalue.adj.FDR columns
- The benchmark table scdhicdiff_vs_setdifference_benchmark.csv is produced with the summarized comparison statistics

## Limitations

- The set-difference baseline treats each condition independently and cannot model zero inflation or joint covariance, so it is a weak reference rather than an equal competitor
- The comparison on real data is qualitative; the article reports outperformance of scHiCDiff over set-difference without a stated numeric threshold for the win margin
- No version numbers are stated for scHiCDiff or R, so reproducibility of exact call sets may vary across versions

## Evidence

- [abstract] findings on scHiCDiff outperforming set-difference baseline: "Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions"
- [intro] ZINB rationale vs naive approaches: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [methods] consensus DCI assembly: "common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)"
- [readme] output schema of parametric detection functions used in the comparison: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
