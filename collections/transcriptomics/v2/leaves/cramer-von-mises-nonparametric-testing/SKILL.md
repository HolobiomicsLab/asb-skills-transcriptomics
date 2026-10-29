---
name: cramer-von-mises-nonparametric-testing
description: Use when you have a normalized scHi-C count matrix (rows = bin pairs,
  columns = cells) split into two condition groups, and want distribution-free DCI
  detection per bin pair.
when_to_use_negative:
- Input is raw (unnormalized) scHi-C matrices — normalize first (scHiCDiff.norm with
  effective length, GC content, mappability features) so the function receives normalized
  read counts
- You need model-based effect sizes (mu/theta/pi MLEs, fold change) or a zero-inflation-aware
  comparison — use scHiCDiff.NB or scHiCDiff.ZINB instead, since CVM returns only
  a test statistic and p-values
- Comparing more than two conditions in one call — the detection functions take a
  factor of exactly two conditions
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scHiCDiff
  role: Provides the scHiCDiff.CVM detection function that performs the Cramér-von
    Mises two-sample test on scHi-C count tables, plus scHiCDiff.common.DCI for cross-method
    comparison
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Runtime environment in which scHiCDiff is installed from source and the CVM
    detection is executed
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:35e63f37003caf75c6c062a29c074097bcfaa5b278ff34ff13aa44c2c43294c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cramer-von-mises-nonparametric-testing

## Summary

Runs the Cramér-von Mises (CVM) non-parametric two-sample test implemented as scHiCDiff.CVM in the scHiCDiff R package to flag differential chromatin interactions (DCIs) between two single-cell Hi-C conditions, without assuming any parametric count distribution.

## When to use

Use when you have a normalized scHi-C count matrix (rows = bin pairs, columns = cells) split into two condition groups, and want distribution-free DCI detection per bin pair.

## When NOT to use

- Input is raw (unnormalized) scHi-C matrices — normalize first (scHiCDiff.norm with effective length, GC content, mappability features) so the function receives normalized read counts
- You need model-based effect sizes (mu/theta/pi MLEs, fold change) or a zero-inflation-aware comparison — use scHiCDiff.NB or scHiCDiff.ZINB instead, since CVM returns only a test statistic and p-values
- Comparing more than two conditions in one call — the detection functions take a factor of exactly two conditions

## Inputs

- count.table: non-negative matrix of scHi-C normalized read counts (rows = bin pairs, columns = samples/cells), e.g. read from sampledata/sim 1.rds
- group: factor vector of the two conditions to be compared, corresponding to the count.table columns

## Outputs

- Data frame of CVM DCI results with one row per bin pair and columns: bin_1, bin_2, test.statistic, pvalue, pvalue.adj (BH-adjusted)

## How to apply

Load the scHi-C normalized read counts as a non-negative count.table matrix whose rows are bin pairs and whose columns are samples/cells, and build a factor 'group' vector labeling the two conditions to be compared, matching the column order. Call scHiCDiff.CVM(count.table, group); the function computes a two-sample Cramér-von Mises test statistic for each bin pair's contact-count distributions across the two groups. The function returns a data frame with rows = bin pairs and columns bin_1, bin_2, test.statistic, pvalue, and pvalue.adj (p-values BH-adjusted by Benjamini & Hochberg). Rationale: the CVM test is one of two non-parametric options in scHiCDiff chosen because it detects DCIs without any assumption on data distribution — a useful contrast to the NB/ZINB likelihood-ratio tests, which model NB dispersion and zero inflation. For confirmation, compare its common DCI set against the KS, NB and ZINB results via scHiCDiff.common.DCI, since the ZINB model was reported to give the most stable detections on real data.

## Related tools

- **scHiCDiff** (Provides the scHiCDiff.CVM detection function that performs the Cramér-von Mises two-sample test on scHi-C count tables, plus scHiCDiff.common.DCI for cross-method comparison) — https://github.com/wmalab/scHiCDiff
- **R** (Runtime environment in which scHiCDiff is installed from source and the CVM detection is executed)

## Examples

```
result.cvm <- scHiCDiff.CVM(count.table, group)
```

## Evaluation signals

- Returned data frame exists with exactly the CVM schema: rows = bin pairs; columns = bin_1, bin_2, test.statistic, pvalue, pvalue.adj
- All pvalue and pvalue.adj entries are in [0, 1] with no NA/NaN in non-failed rows
- test.statistic is finite and non-negative (CVM statistic) for bin pairs with at least one non-zero count
- Rows are valid bin pairs (bin_1, bin_2 present, e.g. 0–10 Mb intra-chromosomal distance range if that QC filter was applied)
- Common DCIs identified with result.ks/result.nb/result.zinb via scHiCDiff.common.DCI overlap with the CVM calls; ZINB detections should agree per the paper's consistency findings

## Limitations

- CVM is distribution-free but does not model excessive zeros in single-cell Hi-C; the paper reports the zero-inflated negative binomial (ZINB) option gave consistent, stable detections that outperformed KS/CVM/NB on real data
- Abnormal program information may appear in a Remark column (parametric outputs); verify per-bin-pair results rather than assuming clean fits
- No version numbers for scHiCDiff or R are stated; install the developmental version from the GitHub tarball and expect potential reproducibility drift

## Evidence

- [readme] the other type is the detection function (scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB and scHiCDiff.ZINB): "the other type is the detection function (scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB and scHiCDiff.ZINB)"
- [methods] result.cvm <- scHiCDiff.CVM(count.table,group): "result.cvm <- scHiCDiff.CVM(count.table,group)"
- [intro] Non-parametric tests are advantageous by allowing us detecting DCIs without any assumption on data distribution: "Non-parametric tests are advantageous by allowing us detecting DCIs without any assumption on data distribution"
- [readme] pvalue.adj           Adjusted P value of H0's pvalue using Benjamini & Hochberg's method.: "pvalue.adj           Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [abstract] provides both nonparametric tests and parametirc models to detect differential chromatin interactions (DCIs) from single-cell Hi-C data: "provides both nonparametric tests and parametirc models to detect differential chromatin interactions (DCIs) from single-cell Hi-C data"
