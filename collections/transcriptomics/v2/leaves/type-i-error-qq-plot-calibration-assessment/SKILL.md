---
name: type-i-error-qq-plot-calibration-assessment
description: Use when benchmarking a statistical method on null/permuted data and
  you need to verify that its p-values are Uniform(0,1)-calibrated rather than inflated
  (false positives) or deflated (lost power).
when_to_use_negative:
- The data are real (non-null) biological samples with genuine co-expression — type-I
  error at a nominal threshold cannot be judged without a known null
- You need to assess estimation bias of co-expression values rather than p-value calibration
  — use the mean-expression binning bias curves of Fig. 1a instead
- The method outputs only correlation/co-expression estimates without p-values or
  a test of independence — there is nothing to Q-Q against the uniform null
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CS-CORE
  role: Method under test providing theoretically justified p-values for gene-pair
    independence on raw UMI counts
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: Normalisr
  role: Comparison method whose p-value calibration is assessed on the same null datasets
  repo: https://github.com/lingfeiwang/normalisr
- name: locCSN
  role: Comparison method included in the type-I error benchmark on permuted null
    data
  repo: https://github.com/xuranw/locCSN
- name: Noise Regularization
  role: Comparison method (code/code.R) whose p-values are aggregated and Q-Q plotted
    on null data
  repo: https://github.com/RuoyuZhang/NoiseRegularization
- name: Seurat
  role: Loads and subsets the PBMC COVID Seurat object used to generate permuted null
    datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:f2c7190f3041623c86101c398229204d4458a2c64b53e8ba83a5afc9d51c124e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CSCORE
  - Noise Regularization
  - Normalisr
  - Seurat
  - locCSN
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Type-I error Q-Q plot calibration assessment

## Summary

Assesses whether a co-expression inference method's p-values are well-calibrated on permuted null scRNA-seq data by aggregating p-values, applying Benjamini-Hochberg correction, and drawing Q-Q plots against Uniform(0,1) to quantify type-I error at the 0.05 threshold.

## When to use

Use when benchmarking a statistical method on null/permuted data and you need to verify that its p-values are Uniform(0,1)-calibrated rather than inflated (false positives) or deflated (lost power).

## When NOT to use

- The data are real (non-null) biological samples with genuine co-expression — type-I error at a nominal threshold cannot be judged without a known null
- You need to assess estimation bias of co-expression values rather than p-value calibration — use the mean-expression binning bias curves of Fig. 1a instead
- The method outputs only correlation/co-expression estimates without p-values or a test of independence — there is nothing to Q-Q against the uniform null

## Inputs

- Permuted null scRNA-seq UMI count matrices (e.g. derived from the PBMC COVID Seurat object blish_covid.seu.rds) with varying sequencing depths and no true co-expression
- Per-method p-value vectors/matrices over gene pairs from each co-expression method run on the null data

## Outputs

- P-value Q-Q plots against Uniform(0,1) per method (Fig. 1b-style)
- Empirical type-I error estimates at the 0.05 (nominal) threshold per method
- BH-adjusted p-values (e.g. p_matrix_BH) with a calibration pass/fail verdict per method

## How to apply

Generate permuted null count datasets under the CS-CORE generative model z_i ~ F_p(mu, Sigma), x_ij | z_ij ~ Poisson(s_i * z_ij), so sequencing depth s_i varies across cells but no gene pairs are truly co-expressed. Run the method under test (e.g. CSCORE() on raw UMI counts) plus comparison methods (baredSC, locCSN, Noise Regularization, Normalisr, Pearson, propr, Spearman, SpQN, rho-analytic PR, rho-sctransform) on the same null data. Collect each method's p-values over gene pairs, apply Benjamini-Hochberg correction (p.adjust with method='BH'), and draw Q-Q plots of observed p-values against the Uniform(0,1) null as in Fig. 1b of the CS-CORE paper. A method passes calibration only if its Q-Q points fall on the identity diagonal, i.e. empirical type-I error at the 0.05 threshold is at or near nominal; upward departure from the diagonal indicates inflated false positives, as the paper found for all existing methods including rho-sctransform and rho-analytic PR.

## Related tools

- **CS-CORE** (Method under test providing theoretically justified p-values for gene-pair independence on raw UMI counts) — https://github.com/ChangSuBiostats/CS-CORE
- **Normalisr** (Comparison method whose p-value calibration is assessed on the same null datasets) — https://github.com/lingfeiwang/normalisr
- **locCSN** (Comparison method included in the type-I error benchmark on permuted null data) — https://github.com/xuranw/locCSN
- **Noise Regularization** (Comparison method (code/code.R) whose p-values are aggregated and Q-Q plotted on null data) — https://github.com/RuoyuZhang/NoiseRegularization
- **Seurat** (Loads and subsets the PBMC COVID Seurat object used to generate permuted null datasets)

## Examples

```
p_matrix_BH <- p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH"); qqplot(runif(length(p_matrix_BH)), p_matrix_BH); abline(0, 1)
```

## Evaluation signals

- Q-Q points for a calibrated method lie on the identity diagonal against Uniform(0,1); CS-CORE should pass while most comparison methods deviate upward
- Empirical type-I error at the 0.05 threshold is at/near nominal (≈0.05) for a passing method
- BH-adjusted p-values are computed consistently across methods, e.g. via p.adjust on the upper.tri of the p-value matrix
- Null datasets must be generated under the CS-CORE generative model (Poisson sampling x_ij | z_ij ~ Poisson(s_i * z_ij)) so no gene pair is truly co-expressed; any Q-Q departure for CS-CORE indicates a null-construction error
- The same permuted null datasets are used for all ten comparison methods so calibration differences are attributable to methods, not data

## Limitations

- Calibration on the CS-CORE Poisson generative model does not guarantee calibration under other count distributions (e.g. negative binomial), which the article defers to future work
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may fail across individuals with different genetic, demographic and clinical characteristics
- Existing methods' inflated type-I errors are documented on this specific PBMC COVID-derived null; results may not transfer to other datasets or protocols

## Evidence

- [abstract] most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
- [intro] all existing methods suffer from inflated type-I errors: "we found that all the existing methods, including ρ-sctransform and ρ-analytic PR, suffer from inflated type-I errors, varying degrees of estimation biases, reduced power in detecting co-expressions"
- [methods] BH adjustment code: "p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH")"
- [readme] Normalisr claims unbiased p-value estimation: [evidence span withheld — anchor status: external_doc]
