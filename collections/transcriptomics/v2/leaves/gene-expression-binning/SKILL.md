---
name: gene-expression-binning
description: Use when you have null (e.g., gRNA-free or split-group) P values from
  single-cell DE or co-expression testing and need to check P-value uniformity across
  expression levels via expression-stratified KS tests.
when_to_use_negative:
- The method under evaluation is a generalized linear model or log(CPM+1) normalization
  known to introduce expression-dependent normalization biases, and the goal is to
  evaluate its null calibration rather than a covariate-removing method like Normalisr
- Input P values are not null (e.g., real gRNA perturbation effects or true case–control
  DE); uniformity is expected only for null P values (guide-unassigned cells or synthetic
  co-expression-free data)
- The data are not UMI-based (the article only focused on UMI datasets; full-length
  platforms were not validated and may need revalidation of covariates)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Provides the normalized expression matrix (lcpm, normcov, compute_var, normvar)
    and the null DE (normalisr.de) or co-expression (normalisr.coex) P values that
    are binned and KS-tested; also supplies qc_reads/qc_outlier filtering
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:a8f382b068b58645eeeda474f317f0cb0309f210a275531ba8e047216998fc9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gene-expression-binning

## Summary

Stratifies genes into 10 ordered expression-level bins and applies two-sided Kolmogorov–Smirnov tests of null P values against the uniform distribution, to verify that a normalization/association method (e.g., Normalisr) yields unbiased P values independent of gene expression level.

## When to use

Use when you have null (e.g., gRNA-free or split-group) P values from single-cell DE or co-expression testing and need to check P-value uniformity across expression levels via expression-stratified KS tests.

## When NOT to use

- The method under evaluation is a generalized linear model or log(CPM+1) normalization known to introduce expression-dependent normalization biases, and the goal is to evaluate its null calibration rather than a covariate-removing method like Normalisr
- Input P values are not null (e.g., real gRNA perturbation effects or true case–control DE); uniformity is expected only for null P values (guide-unassigned cells or synthetic co-expression-free data)
- The data are not UMI-based (the article only focused on UMI datasets; full-length platforms were not validated and may need revalidation of covariates)

## Inputs

- Null P values per gene from single-cell differential expression (e.g., normalisr.de on random cell-group splits)
- Null P values per gene pair from single-cell co-expression testing (e.g., normalisr.coex conditional Pearson tests)
- Normalized UMI count/expression matrix (lcpm + normcov + compute_var + normvar) used to compute per-gene mean expression levels for binning

## Outputs

- 10 expression bins (or 55 unique bin pairs) of genes ordered by mean expression level
- Per-bin (or per-bin-pair) two-sided KS statistics and P values against the uniform distribution
- Aggregated summary tables such as ks_uniformity_results.csv or bin_pair_ks_results.csv
- Optional P-value histogram figures per bin pair (e.g., fig2de_null_pvalue_histograms.png)

## How to apply

After applying the method's QC and normalization (for Normalisr: qc_reads, qc_outlier with 2% prior outlier bound and Bonferroni P <= 1e-10, then lcpm, normcov, compute_var, normvar), obtain null P values — e.g., normalisr.de on 100 random two-group splits of 4622 guide-unassigned K562 Perturb-seq cells, or normalisr.coex conditional Pearson tests on a synthetic null dataset. Sort genes by mean expression level and divide them into 10 ordered bins (for co-expression, form 55 unique bin pairs). For each bin or bin pair, test the null P-value distribution against uniform [0,1] with a two-sample/two-sided Kolmogorov–Smirnov test, optionally building P-value histograms (as in Fig. 2d,e). Acceptance requires KS tests showing no significant departure from uniformity within every bin (pair), aggregated over bins and splits into a summary table (e.g., ks_uniformity_results.csv or bin_pair_ks_results.csv).

## Related tools

- **Normalisr** (Provides the normalized expression matrix (lcpm, normcov, compute_var, normvar) and the null DE (normalisr.de) or co-expression (normalisr.coex) P values that are binned and KS-tested; also supplies qc_reads/qc_outlier filtering) — github.com/lingfeiwang/normalisr

## Examples

```
from scipy import stats; ks_results = [stats.kstest(bin_pvalues, 'uniform') for bin_pvalues in ten_expression_bins]
```

## Evaluation signals

- Two-sided KS tests within every expression bin (or all 55 bin pairs) show no significant departure from uniformity of null P values
- Uniformity holds in all 10 expression bins and across all 100 random two-group splits of gRNA-free cells for the DE null test
- Output summary tables (ks_uniformity_results.csv / bin_pair_ks_results.csv) contain KS statistic and P value per bin or bin pair
- Histograms of null P values per bin pair are approximately flat across [0,1] (reproducing Fig. 2d) for Normalisr, unlike log(CPM+1) with or without covariates
- QC upstream retained high-quality cells: outlier removal with prior outlier-proportion bound 2% and Bonferroni P <= 1e-10

## Limitations

- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Only UMI-based datasets were analyzed; full-length platforms may produce a different read-count distribution and require revalidation of the nonlinear cellular summary covariates
- Binning into 10 expression levels yields 55 unique bin pairs for co-expression, so per-bin-pair KS tests may lack power at fine stratification

## Evidence

- [abstract] unbiased p-value estimation across expression levels: "Normalisr achieves high sensitivity, specificity, speed, and generalizability across multiple scRNA-seq protocols and experimental conditions with unbiased p-value estimation"
- [methods] normalization pipeline functions: "Normalization
==============
.. autosummary::
	:toctree:
	nosignatures:

	lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [discussion] outlier removal thresholds: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [readme] README evidence: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
