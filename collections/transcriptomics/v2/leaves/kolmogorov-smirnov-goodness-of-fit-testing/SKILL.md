---
name: kolmogorov-smirnov-goodness-of-fit-testing
description: Use when validating that null P-value distributions from a statistical
  association method are uniform (unbiased), e.g. stratifying gene pairs into expression-level
  bins and testing each bin's P-value histogram against Uniform[0,1].
when_to_use_negative:
- The association method is not designed for P-value-based FDR estimation (e.g. ScVI
  was excluded from the paper's comparison for this reason), so uniformity testing
  of P values is not meaningful
- Testing arbitrary nonlinear associations rather than the null-calibration of linear
  conditional Pearson co-expression P values — Normalisr is not designed for hypothesis
  testing of arbitrary nonlinear associations
- Data come from full-length (non-UMI) platforms, where the nonlinear cellular summary
  covariates and P-value calibration were not validated
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3516
tools:
- name: Normalisr
  role: Normalization (lcpm, normcov, scaling_factor, compute_var, normvar) and conditional
    Pearson co-expression testing producing the null P values whose uniformity is
    KS-tested
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:4841b503d5adb50b258990c61d31456b323587fe07e7d4864f308ef72bab8a14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3516
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kolmogorov-smirnov-goodness-of-fit-testing

## Summary

Two-sample Kolmogorov-Smirnov tests are used in the Normalisr workflow to verify that null co-expression P values on a synthetic co-expression-free scRNA-seq dataset are uniformly distributed within each pair of gene expression bins, validating unbiased P-value estimation across expression levels.

## When to use

Use when validating that null P-value distributions from a statistical association method are uniform (unbiased), e.g. stratifying gene pairs into expression-level bins and testing each bin's P-value histogram against Uniform[0,1].

## When NOT to use

- The association method is not designed for P-value-based FDR estimation (e.g. ScVI was excluded from the paper's comparison for this reason), so uniformity testing of P values is not meaningful
- Testing arbitrary nonlinear associations rather than the null-calibration of linear conditional Pearson co-expression P values — Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Data come from full-length (non-UMI) platforms, where the nonlinear cellular summary covariates and P-value calibration were not validated

## Inputs

- Simulated null scRNA-seq UMI count matrix (co-expression-free synthetic dataset)
- Null co-expression P values for all gene pairs from normalisr.coex conditional Pearson test
- Gene mean expression levels used to define the 10 expression bins

## Outputs

- Histograms of null co-expression P values for all 55 gene expression bin pairs (fig2de_null_pvalue_histograms.png)
- Per-bin-pair KS statistics and P values (bin_pair_ks_results.csv)

## How to apply

First obtain null co-expression P values by running the conditional Pearson co-expression test (normalisr.coex) on a simulated null scRNA-seq UMI count matrix after Normalisr normalization (normalisr.norm: lcpm, normcov, scaling_factor, compute_var, normvar) that removes nonlinear library-size confounders at mean and variance levels. Split genes into 10 bins ordered by mean expression, yielding 55 unique bin pairs. For each bin pair, build a histogram of the null P values (reproducing Fig. 2d) and apply a two-sample Kolmogorov-Smirnov test against the uniform [0,1] distribution (reproducing Fig. 2e). Acceptance requires no significant KS departure from uniformity in every bin pair, confirming the normalization removes technical confounding irrespective of gene expression level; Normalisr outperformed log(CPM+1) with or without covariates, and no other normalization or imputation method fully accounted for the confounding.

## Related tools

- **Normalisr** (Normalization (lcpm, normcov, scaling_factor, compute_var, normvar) and conditional Pearson co-expression testing producing the null P values whose uniformity is KS-tested) — https://github.com/lingfeiwang/normalisr

## Examples

```
stats.kstest(bin_pair_pvalues, 'uniform')
```

## Evaluation signals

- Every one of the 55 bin-pair KS tests shows no significant departure from uniform [0,1], i.e. approximately uniform null P values within every bin pair
- Histograms of P values in fig2de_null_pvalue_histograms.png are flat across [0,1] for all bin pairs, including low-expression bins
- KS results in bin_pair_ks_results.csv show comparable or smaller statistics than log(CPM+1) with or without covariates (Fig. 2d, e comparison)
- The input matrix is a synthetic null dataset in which no true co-expression exists, so any non-uniformity indicates residual technical confounding

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms were not validated and may produce a different read count distribution requiring revalidation of the nonlinear covariates
- Validation depends on the synthetic null dataset being genuinely co-expression-free; batch effects in multi-dataset integrative studies are not addressed
- Normalisr's unbiased P-value estimation relies on enough (> 100) cells for robustness against read count distribution distortions

## Evidence

- [results] Normalisr recovered uniformly distributed null co-expression P values irrespective of gene expression level, outperforming log(CPM+1) with or without covariates (Figs. 2d, e): [evidence span withheld — anchor status: not_found]
- [methods] apply normalisr.norm: lcpm, normcov, scaling_factor, compute_var, normvar to remove nonlinear library-size confounders at mean and variance levels: [evidence span withheld — anchor status: not_found]
- [abstract] Normalisr ... achieves high sensitivity, specificity, speed, and generalizability across multiple scRNA-seq protocols and experimental conditions with unbiased p-value estimation: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [readme] robustness against read count distribution distortions with enough (> 100) cells: "(iv) robustness against read count distribution distortions with enough (> 100) cells, and (iv) computational efficiency."
- [discussion] Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations: "Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations54"
