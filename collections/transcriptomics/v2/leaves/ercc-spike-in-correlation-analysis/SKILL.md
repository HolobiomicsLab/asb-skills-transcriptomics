---
name: ercc-spike-in-correlation-analysis
description: Use when evaluating whether an imputation method (e.g., scImpute on raw
  read counts with drop_thre = 0.5) recovers true expression levels masked by dropout
  events, using ERCC spike-ins with known concentrations as ground truth.
when_to_use_negative:
- The dataset lacks ERCC spike-ins with known concentrations, so no ground-truth correlation
  can be computed
- Only a filtered gene subset is available — scImpute recommends applying it to the
  whole-genome count matrix to ensure robust dropout identification
- The analysis goal is clustering, differential expression, or time-series analysis
  downstream of imputation rather than quantifying imputation accuracy
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: Imputes dropout values in the raw count matrix via the scimpute function (drop_thre
    = 0.5, Kcluster, labeled = FALSE, ncores), producing scimpute_count.csv whose
    ERCC values are then correlated against known spike-in concentrations
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:148bea97343f795f604c103fb90186786b0442aa46a61adc506bb089ce7fb0aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ercc-spike-in-correlation-analysis

## Summary

Quantifies dropout-recovery quality of scImpute imputation by computing per-cell Pearson correlations between observed ERCC spike-in read counts (57 transcripts in the GSE60361 mouse somatosensory cortex dataset, 3005 cells) and their known concentrations, before vs. after imputation.

## When to use

Use when evaluating whether an imputation method (e.g., scImpute on raw read counts with drop_thre = 0.5) recovers true expression levels masked by dropout events, using ERCC spike-ins with known concentrations as ground truth.

## When NOT to use

- The dataset lacks ERCC spike-ins with known concentrations, so no ground-truth correlation can be computed
- Only a filtered gene subset is available — scImpute recommends applying it to the whole-genome count matrix to ensure robust dropout identification
- The analysis goal is clustering, differential expression, or time-series analysis downstream of imputation rather than quantifying imputation accuracy

## Inputs

- GSE60361 raw count matrix (CSV, genes as rows, 3005 mouse somatosensory cortex cells as columns) including 57 ERCC spike-in transcripts
- ERCC spike-in transcript concentration table (known true concentrations)

## Outputs

- scimpute_count.csv — imputed count matrix with the same gene-by-cell dimensions as the raw input
- Per-cell Pearson correlations between ERCC observed counts and known concentrations (raw and imputed)
- Median and minimum pre- vs. post-imputation correlation summary (expected: median 0.92→0.95, minimum 0.81→0.89)

## How to apply

Load the GSE60361 raw count matrix (genes as rows, 3005 cells as columns; 57 ERCC transcripts) plus the ERCC spike-in concentration table into R. Run scImpute's scimpute function with type = 'count', infile/outfile = 'csv', drop_thre = 0.5, Kcluster set to the known number of cell subpopulations, labeled = FALSE, and parallel ncores; scImpute writes the imputed matrix to scimpute_count.csv in out_dir. Compute per-cell Pearson correlations between the 57 ERCC observed counts and their known concentrations for both the raw and imputed matrices. Summarize the median and minimum of the pre- vs. post-imputation correlation distributions; success is median rising from 0.92 to 0.95 and minimum rising from 0.81 to 0.89, with scimpute_count.csv preserving the gene-by-cell dimensions of the input. scImpute should be run on the whole-genome count matrix so dropout identification is robust.

## Related tools

- **scImpute** (Imputes dropout values in the raw count matrix via the scimpute function (drop_thre = 0.5, Kcluster, labeled = FALSE, ncores), producing scimpute_count.csv whose ERCC values are then correlated against known spike-in concentrations) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = "raw_count.csv", infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- Median per-cell ERCC count-vs-concentration Pearson correlation increases from 0.92 (raw) to 0.95 (imputed)
- Minimum per-cell ERCC correlation increases from 0.81 (raw) to 0.89 (imputed)
- scimpute_count.csv exists in out_dir and has the same gene-by-cell dimensions as the raw count input
- The scimpute call returns the column indices of outlier cells, which scImpute excludes from imputation
- Per-cell correlations are computed over exactly the 57 ERCC transcripts present in the raw matrix

## Limitations

- Imputation efficiency degrades when dropout rates in the raw data are severely high (e.g., droplet-based technologies), noted as a future direction by the authors
- Kcluster (number of cell subpopulations) must be chosen by the user, e.g., from clustering of the raw data; no definitive selection procedure is given
- Computational efficiency depends on an optional cell-filtering step based on biological knowledge that is not specified
- A filtering step on genes is acceptable, but most genes should be present to ensure robust identification of dropouts

## Evidence

- [methods] Only the values that have dropout probability larger than drop_thre are imputed by scImpute: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [abstract] scImpute automatically identifies likely dropouts, and only perform imputation on these values: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [readme] README quick-start scimpute invocation with drop_thre and Kcluster parameters: "drop_thre = 0.5,          # threshold set on dropout probability
         Kcluster = 2,             # 2 cell subpopulations"
- [readme] README note on applying scImpute to the whole-genome count matrix: [evidence span withheld — anchor status: external_doc]
- [methods] Parallel computation recommendation: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
