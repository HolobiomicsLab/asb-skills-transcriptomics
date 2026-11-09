---
name: nonparametric-goodness-of-fit-testing
description: Use when comparing single-cell Hi-C interaction counts between two conditions
  on a bin-pair-by-cell normalized count matrix and you want DCI calls that make no
  assumption on the data distribution.
when_to_use_negative:
- 'If you can justify a parametric count model and want maximum sensitivity: the article''s
  simulation benchmark shows the parametric NB and especially the zero-inflated negative
  binomial (ZINB) likelihood-ratio tests outperform KS and CVM in all simulation settings
  (cell num=50, fold change=5, 200kb resolution).'
- 'If the input is raw (not normalized) scHi-C counts: the detection functions expect
  a matrix of scHi-C normalized read counts, so scHiCDiff.norm should be applied first
  using local bias features (effective length, GC content, mappability).'
- If your data are bulk Hi-C counts without excessive zeros rather than single-cell-level
  comparisons; the ZINB alternative was specially designed for single-cell interaction
  comparison.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scHiCDiff
  role: R package providing the non-parametric detection functions scHiCDiff.KS and
    scHiCDiff.CVM plus scHiCDiff.common.DCI to intersect DCI calls across methods
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Runtime environment in which scHiCDiff is installed (developmental version
    via install.packages with type='source') and the KS/CVM detection functions are
    invoked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:ab2593d1624629a4322460dd9a56c0c673d7f073700b294a9764c778a50bad5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# nonparametric-goodness-of-fit-testing

## Summary

Apply distribution-free Kolmogorov–Smirnov and Cramér–von Mises tests (scHiCDiff.KS, scHiCDiff.CVM) to detect differential chromatin interactions (DCIs) between two conditions in single-cell Hi-C count matrices, without assuming a parametric count distribution.

## When to use

Use when comparing single-cell Hi-C interaction counts between two conditions on a bin-pair-by-cell normalized count matrix and you want DCI calls that make no assumption on the data distribution.

## When NOT to use

- If you can justify a parametric count model and want maximum sensitivity: the article's simulation benchmark shows the parametric NB and especially the zero-inflated negative binomial (ZINB) likelihood-ratio tests outperform KS and CVM in all simulation settings (cell num=50, fold change=5, 200kb resolution).
- If the input is raw (not normalized) scHi-C counts: the detection functions expect a matrix of scHi-C normalized read counts, so scHiCDiff.norm should be applied first using local bias features (effective length, GC content, mappability).
- If your data are bulk Hi-C counts without excessive zeros rather than single-cell-level comparisons; the ZINB alternative was specially designed for single-cell interaction comparison.

## Inputs

- count.table: non-negative matrix of scHi-C normalized read counts (rows = bin pairs, columns = samples/cells), e.g. sampledata/sim 1.rds
- group: factor vector labeling the two conditions, corresponding to the columns of count.table (e.g. 50 cells per condition)

## Outputs

- Data frame per method (KS, CVM) with columns bin_1, bin_2, test.statistic, pvalue, pvalue.adj (BH-adjusted)
- Common DCI set from scHiCDiff.common.DCI listing bin pairs significant across KS, CVM, NB and ZINB

## How to apply

Coerce the normalized scHi-C count matrix (rows = bin pairs, columns = cells, e.g. read from sampledata/sim 1.rds with readRDS then as.matrix) and define the two-condition comparison as a factor vector group <- factor(c(rep(1,50), rep(2,50))) matching the 50 condition-1 and 50 condition-2 columns. Run scHiCDiff.KS (Kolmogorov–Smirnov test) and scHiCDiff.CVM (Cramér–von Mises test) on the count table and group vector; each returns a data frame with bin_1, bin_2, the test statistic, raw pvalue and Benjamini–Hochberg adjusted p-value (pvalue.adj for KS/CVM). These non-parametric tests are chosen because they detect DCIs without any distributional assumption, in contrast to the parametric NB/ZINB likelihood-ratio options in the same package. Compare the KS and CVM DCI calls with the parametric scHiCDiff.NB and scHiCDiff.ZINB results and intersect them via scHiCDiff.common.DCI to obtain DCIs supported by all four methods.

## Related tools

- **scHiCDiff** (R package providing the non-parametric detection functions scHiCDiff.KS and scHiCDiff.CVM plus scHiCDiff.common.DCI to intersect DCI calls across methods) — https://github.com/wmalab/scHiCDiff
- **R** (Runtime environment in which scHiCDiff is installed (developmental version via install.packages with type='source') and the KS/CVM detection functions are invoked)

## Examples

```
count.table <- as.matrix(readRDS('path/sampledata/sim 1.rds')); group <- factor(c(rep(1,50), rep(2,50))); result.ks <- scHiCDiff.KS(count.table, group); result.cvm <- scHiCDiff.CVM(count.table, group)
```

## Evaluation signals

- Output data frames contain the required columns bin_1, bin_2, test.statistic, pvalue and pvalue.adj for the non-parametric results.
- All BH-adjusted p-values (pvalue.adj) fall within [0,1] and raw p-values are monotone-consistent with test statistics.
- The common-DCI set from scHiCDiff.common.DCI(result.ks, result.cvm, result.nb, result.zinb) is non-empty and equals the overlap of the four per-method DCI calls.
- In the simulated benchmark (chr1 of 1CDX1, cell num=50, fold change=5, 200kb resolution), KS/CVM DCIs should overlap substantially with parametric NB/ZINB calls, with ZINB giving the most accurate reference calls.
- Each listed bin pair has at least one non-zero count in one of the cell types, matching the sample-data construction.

## Limitations

- Simulation results show the non-parametric KS and CVM tests underperform the parametric NB and ZINB likelihood-ratio tests, with ZINB producing the most accurate results in all simulation settings.
- No version numbers are stated for scHiCDiff or R; the package is distributed as a developmental tarball (scHiCDiff_1.0.tar.gz).
- The non-parametric tests do not model the excessive zeros characteristic of single-cell Hi-C counts, which is why the ZINB regression was specially designated for single-cell-level interaction comparison.

## Evidence

- [intro] Non-parametric tests are advantageous by allowing us detecting DCIs without any assumption on data distribution: "Non-parametric tests are advantageous by allowing us detecting DCIs without any assumption on data distribution"
- [readme] README detection function example with KS and common.DCI: "result.ks <- scHiCDiff.KS(count.table,group)"
- [readme] common DCIs identified by four methods: "#common DCIs identified by four methods
common.DCIs <- scHiCDiff.common.DCI(result.ks,result.cvm,result.nb,result.zinb)"
- [readme] README output schema for non-parametric tests: "test.statistic       The statistic given by KS/CVM test.
pvalue                P value of hypothesis testing of H0"
- [results] Benchmark finding that parametric models outperform non-parametric ones: [evidence span withheld — anchor status: not_found]
