---
name: null-p-value-uniformity-assessment
description: Use when benchmarking a normalization/association method (e.g., Normalisr)
  on a synthetic null UMI count matrix and you need to verify null co-expression P
  values are uniform across expression levels.
when_to_use_negative:
- The dataset contains true biological co-expression or experimental perturbations
  — the P-value-uniformity criterion only holds on a synthetic null dataset
- Normalisr's co-expression module is inappropriate because the goal is hypothesis
  testing of arbitrary nonlinear associations or causal network reconstruction (v-structures,
  regularized regression, Granger causality)
- The data are full-length (non-UMI) platform reads, which the study did not validate
  and where the nonlinear cellular summary covariates may need revalidation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: 'Provides the normalization module (normalisr.norm: lcpm, normcov, scaling_factor,
    compute_var, normvar) and the conditional Pearson co-expression test (normalisr.coex)
    whose null P values are tested for uniformity'
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:622ba08ce56525d41de13b6d52a537c7a5e150bebfc505fc52aff0c4644616e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# null-p-value-uniformity-assessment

## Summary

Assess whether a co-expression/association testing framework yields unbiased null P values on a synthetic null scRNA-seq dataset by testing P-value distributions against Uniform[0,1], stratified by gene expression bins with KS tests (as done for Normalisr vs. log(CPM+1) in Fig. 2d, e).

## When to use

Use when benchmarking a normalization/association method (e.g., Normalisr) on a synthetic null UMI count matrix and you need to verify null co-expression P values are uniform across expression levels.

## When NOT to use

- The dataset contains true biological co-expression or experimental perturbations — the P-value-uniformity criterion only holds on a synthetic null dataset
- Normalisr's co-expression module is inappropriate because the goal is hypothesis testing of arbitrary nonlinear associations or causal network reconstruction (v-structures, regularized regression, Granger causality)
- The data are full-length (non-UMI) platform reads, which the study did not validate and where the nonlinear cellular summary covariates may need revalidation

## Inputs

- Simulated (synthetic null) scRNA-seq UMI count matrix (tsv or Cell Ranger sparse mtx)
- Gene and cell name lists, one per line (Normalisr text file format)
- Optional covariates (e.g., batches, house-keeping programs, untested gRNAs) as fixed effects

## Outputs

- Null co-expression P values for all gene pairs (normalisr.coex output)
- 55 histograms of null P values stratified by gene-expression bin pair (fig2de_null_pvalue_histograms.png)
- Per-bin-pair two-sample KS statistics and P values against Uniform[0,1] (bin_pair_ks_results.csv)

## How to apply

Load the simulated null scRNA-seq UMI count matrix and apply the method's normalization module — for Normalisr this is normalisr.norm (lcpm, normcov, scaling_factor, compute_var, normvar), which systematically detects and removes nonlinear library-size confounders at the mean and variance levels. Then run the conditional Pearson co-expression test from normalisr.coex on the normalized expression matrix to obtain null P values for all gene pairs. Split genes into 10 bins ordered by mean expression, yielding 55 unique bin pairs. For each bin pair, build a histogram of null P values (reproducing Fig. 2d) and run a two-sample Kolmogorov–Smirnov test of the distribution against Uniform[0,1] (reproducing Fig. 2e). Compare against baselines such as log(CPM+1) with or without covariates; acceptance requires no significant KS departure from uniformity within every bin pair, confirming unbiased P-value estimation across expression levels.

## Related tools

- **Normalisr** (Provides the normalization module (normalisr.norm: lcpm, normcov, scaling_factor, compute_var, normvar) and the conditional Pearson co-expression test (normalisr.coex) whose null P values are tested for uniformity) — https://github.com/lingfeiwang/normalisr

## Examples

```
python3 -m normalisr coex -h
```

## Evaluation signals

- Two-sample KS tests of null P values against Uniform[0,1] show no significant departure within every one of the 55 gene-expression bin pairs
- P-value histograms per bin pair visually approximate flat (uniform) shapes, unlike log(CPM+1) with or without covariates
- Normalization pipeline ran in the prescribed order: lcpm → normcov → scaling_factor → compute_var → normvar before coex
- All gene pairs received valid P values in [0,1] with no missing or degenerate (all-zero/one) distributions
- Acceptance criterion: uniformity holds irrespective of gene expression level (all 10 mean-expression bins)

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms were not validated and the nonlinear cellular summary covariates may produce a different read-count distribution requiring revalidation
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Uniformity of null P values is a property of the synthetic null setting and does not by itself establish sensitivity on data with real co-expression

## Evidence

- [abstract] By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity"
- [intro] Normalization estimates pre-measurement mRNA frequencies from scRNA-seq read counts: "The normalization step estimates the pre-measurement mRNA frequencies from the scRNA-seq read counts"
- [readme] Normalisr uses submodules for different analysis steps: [evidence span withheld — anchor status: external_doc]
