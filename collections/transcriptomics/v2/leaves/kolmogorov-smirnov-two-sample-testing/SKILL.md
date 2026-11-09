---
name: kolmogorov-smirnov-two-sample-testing
description: Use when comparing scHi-C contact counts per bin pair between exactly
  two cell groups (e.g. oocyte vs zygote), as a non-negative count.table matrix with
  a group factor, and no distributional assumption on counts is desired.
when_to_use_negative:
- Input is raw (not normalized) scHi-C counts or data violating the QC criteria (cells
  with ≤5000 nondiagonal intra-chromosomal contacts, contacts beyond 0–10 Mb)
- Counts can reasonably be modeled parametrically and the single-cell excess-zeros
  structure should be exploited — use scHiCDiff.NB or scHiCDiff.ZINB instead, which
  return chi2LR1 and pvalue.adj.FDR
- Comparison involves more than two conditions — the group factor must label exactly
  two conditions to be compared
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: scHiCDiff
  role: R package providing the scHiCDiff.KS non-parametric detection function and
    the scHiCDiff.common.DCI consensus step
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: implementation environment; scHiCDiff is installed from source and loaded
    via library(scHiCDiff)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:42bf98f5a3a4955cf52675ee9e1efa7437c941d6f2faf7a165c0aa9580a29163
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kolmogorov-smirnov-two-sample-testing

## Summary

Non-parametric two-sample Kolmogorov–Smirnov testing for differential chromatin interactions (DCIs) between two single-cell Hi-C conditions, implemented as scHiCDiff.KS. It detects DCIs per bin pair without assuming any distribution on normalized interaction counts.

## When to use

Use when comparing scHi-C contact counts per bin pair between exactly two cell groups (e.g. oocyte vs zygote), as a non-negative count.table matrix with a group factor, and no distributional assumption on counts is desired.

## When NOT to use

- Input is raw (not normalized) scHi-C counts or data violating the QC criteria (cells with ≤5000 nondiagonal intra-chromosomal contacts, contacts beyond 0–10 Mb)
- Counts can reasonably be modeled parametrically and the single-cell excess-zeros structure should be exploited — use scHiCDiff.NB or scHiCDiff.ZINB instead, which return chi2LR1 and pvalue.adj.FDR
- Comparison involves more than two conditions — the group factor must label exactly two conditions to be compared

## Inputs

- count.table: non-negative matrix of scHi-C normalized read counts (rows = bin pairs, columns = samples/cells), QC-filtered to cells with >5000 nondiagonal intra-chromosomal contacts and 0–10 Mb contact range
- group: factor vector labeling the two conditions to be compared, corresponding to the columns of count.table

## Outputs

- result data frame with rows = bin pairs and columns: bin_1, bin_2, test.statistic (KS statistic), pvalue, pvalue.adj (BH FDR adjusted)

## How to apply

First QC the scHi-C data: keep only cells with more than 5000 total nondiagonal intra-chromosomal contacts and restrict bin pairs to the 0–10 Mb contact-distance range, then optionally normalize with scHiCDiff.norm using local features (effective length, GC content, mappability). Assemble count.table with bin pairs as rows and samples/cells as columns, plus a group factor vector labeling the two conditions. Call scHiCDiff.KS(count.table, group) to obtain a per-bin-pair Kolmogorov–Smirnov test.statistic, pvalue, and pvalue.adj adjusted with Benjamini & Hochberg FDR. The KS result can then be combined with the CVM and parametric NB/ZINB results via scHiCDiff.common.DCI to identify DCIs supported by all four methods. The non-parametric route is chosen precisely because it detects DCIs without any assumption on data distribution, unlike the NB/ZINB likelihood-ratio alternatives.

## Related tools

- **scHiCDiff** (R package providing the scHiCDiff.KS non-parametric detection function and the scHiCDiff.common.DCI consensus step) — https://github.com/wmalab/scHiCDiff
- **R** (implementation environment; scHiCDiff is installed from source and loaded via library(scHiCDiff))

## Examples

```
count.table <- readRDS(paste("path/sampledata/sim 1 .rds")); group <- factor(c(rep(1,50), rep(2,50))); result.ks <- scHiCDiff.KS(count.table, group)
```

## Evaluation signals

- Returned object is a data frame with rows of bin pairs and the exact non-parametric output schema: bin_1, bin_2, test.statistic, pvalue, pvalue.adj
- pvalue.adj values are BH-adjusted (Benjamini & Hochberg) versions of the raw pvalues for the same bin pairs
- Every count.table entry is non-negative and the group factor has exactly two levels matching the column count
- Running scHiCDiff.common.DCI across result.ks, result.cvm, result.nb, result.zinb returns a non-empty common-DCIs set consistent with the KS ranking
- Abnormal program information would be recorded in a Remark column for parametric outputs — its absence alongside valid KS statistics indicates clean execution

## Limitations

- Non-parametric tests detect DCIs without distributional assumptions, but they do not model the excessive zeros of single-cell data the way the zero-inflated negative binomial model does
- No version numbers stated for scHiCDiff or R in the provided context; only a developmental version (scHiCDiff_1.0.tar.gz) installation is documented
- No discussion section text available to confirm downstream interpretation caveats

## Evidence

- [intro] Non-parametric tests are advantageous by allowing us detecting DCIs without any assumption on data distribution: "Non-parametric tests are advantageous by allowing us detecting DCIs without any assumption on data distribution"
- [readme] count.table non-negative matrix of scHi-C normalized read counts; rows are bin pair and columns are samples/cells: "count.table      A non-negative  matrix of scHi-C normalized read counts.The rows of the 
                 matrix are bin pair and columns are samples/cells."
- [readme] The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results"
- [results] only cells with more than 5000 total nondiagonal intra-chromosomal contacts, and contacts within the 0–10 Mb range, were used: [evidence span withheld — anchor status: not_found]
